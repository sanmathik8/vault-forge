# VaultForge

A local DevSecOps pipeline that enforces security controls at every stage of the software delivery lifecycle — commit, build, image, deployment, and runtime — instead of relying on a single scan at the end.

The pipeline itself is the deliverable. [OWASP PyGoat](https://github.com/adeyosemanputra/pygoat), an intentionally vulnerable Django application, is used as the test subject to prove every stage actually detects real issues rather than reporting a false "clean" result.

---

## Why this exists

Most CI/CD pipelines answer one question: does the code build? That leaves five real attack surfaces unexamined:

| Stage | Attack surface | Question answered |
|---|---|---|
| Commit | Leaked credentials | Did a secret get committed? |
| Source code | Insecure code patterns | Does the code contain known vulnerability classes (injection, XSS, broken auth)? |
| Build artifact | Vulnerable dependencies | Does the container image ship known CVEs in its OS packages or libraries? |
| Deployment | Exploitable web surface | Does the *running* application expose attackable behavior? |
| Runtime | Post-compromise activity | If an attacker gets a foothold inside the container, does anything notice? |

VaultForge maps a tool to each stage, chosen because no single tool answers all five questions. The pipeline is structured for reuse via an `env:` block (`APP_PATH`, `IMAGE_NAME`, `APP_PORT`) — it has not yet been run against a target other than PyGoat, but no other files would need to change to do so.

## What it solves

- **Visibility** — every security layer produces a real, quantified result: actual counts (62 critical CVEs, 52 SAST findings), not a pass/fail checkbox, with full detail reports attached to every run
- **Enforcement** — critical-severity findings are configured to block the pipeline, not just get logged (see [Failure policy](#failure-policy) for how this is currently exercised)
- **Traceability** — a failed gate automatically opens a GitHub Issue with commit and run references
- **Reusability** — the same workflow can target a different application by changing a handful of environment variables

## Architecture

```
                              git push
                                 │
                                 ▼
                     ┌─────────────────────┐
                     │   Gitleaks           │   BLOCKING
                     │   secrets scan       │   (hard stop on match)
                     └──────────┬───────────┘
                                 │ pass
                 ┌───────────────┴────────────────┐
                 ▼                                ▼
       ┌──────────────────┐             ┌───────────────────────┐
       │  Semgrep          │  BLOCKING*  │  Docker build          │  BLOCKING
       │  SAST              │  on ERROR   │  + Falco (CI runner    │
       │                    │  severity   │  supply-chain monitor) │
       └──────────────────┘             └──────────┬────────────┘
                                                     │ image built
                                        ┌────────────┴─────────────┐
                                        ▼                          ▼
                              ┌──────────────────┐       ┌──────────────────┐
                              │  Syft             │       │  Trivy            │
                              │  SBOM generation   │      │  image CVE scan   │  BLOCKING*
                              └──────────────────┘       │                    │  on CRITICAL
                                                          └──────────┬───────┘
                                                                     │
                                                                     ▼
                                                          ┌──────────────────┐
                                                          │  OWASP ZAP        │  REPORTING
                                                          │  full active scan │  (always)
                                                          └──────────┬───────┘
                                                                     │
                                        ┌────────────────────────────┴───┐
                                        ▼                                ▼
                              ┌──────────────────┐          ┌──────────────────┐
                              │  Create GitHub    │          │  Security Summary │
                              │  Issue (if any     │          │  (score + table)   │
                              │  gate failed)      │          └──────────────────┘
                              └──────────────────┘

* BLOCKING gates can be bypassed via the ALLOW_CRITICAL_CVES repo variable —
  see Failure policy below for exactly how and why this is currently set.

Deployed separately (not part of CI) to a local kind cluster:

    Helm install ──▶ PyGoat pod running ──▶ Falco (runtime syscall monitoring)
```

## Pipeline flow

The pipeline runs as a job DAG in GitHub Actions, not a flat sequence:

- `secrets-scan` runs first and gates everything downstream via `needs:`
- `sast-scan` (Semgrep) and `build-and-sbom` (Docker + Falco + Syft) run **in parallel** once secrets pass — Semgrep only needs source code, so it doesn't wait on a built image
- `image-scan` (Trivy) and `dast-scan` (ZAP) both depend on `build-and-sbom`, since both need the actual built artifact, but run in parallel with each other
- `create-issue-on-critical` runs only if the Trivy or Semgrep gate failed
- `security-summary` runs last regardless of outcome, aggregating every prior job's output into one report

## Failure policy

- **Gitleaks (secrets) and the Docker build are unconditional hard gates** — no bypass exists for either. A leaked credential or an image that doesn't build always stops the pipeline.
- **Trivy is configured to block on any CRITICAL-severity CVE. Semgrep is configured to block on any ERROR-severity finding.** Both gates check a repository variable, `ALLOW_CRITICAL_CVES`, before enforcing.
- **This variable is currently set to `true` in this repository.** PyGoat runs on an end-of-life base image and contains intentional OWASP Top 10 vulnerabilities by design, so both gates would fail on every single run without a bypass — the results below were captured with the bypass active, so the pipeline could complete end-to-end and produce full reports. **With the variable unset, the same run fails at the `image-scan` job** on the 62 CRITICAL CVEs shown below; a screenshot of that failed run is in the repo's Actions history.
- **ZAP always reports and continues** — regardless of the bypass variable, since PyGoat's DAST findings are expected on every run and the value here is quantification, not gatekeeping.
- **A GitHub Issue is auto-created** whenever the Trivy or Semgrep gate fails (i.e., when the bypass variable is unset and a CRITICAL/ERROR finding exists), with finding counts, commit SHA, and a link to the run.

## Tools, and why each one is here

| Tool | Stage | Why this one |
|---|---|---|
| **Gitleaks** | Commit | Scans full git history, not just the latest diff — catches secrets committed and later "removed" but still present in history |
| **Semgrep** | Source code | Multi-language pattern matching with targeted rulesets (`p/owasp-top-ten`, `p/django`, `p/python`); chosen over Bandit for cross-language reuse beyond this one Python project |
| **Falco — CI mode** | Build (pipeline itself) | A separate Falco deployment, using `falcosecurity/falco-actions` with default rules, monitoring the GitHub Actions runner during the build for supply-chain anomalies |
| **Syft** | Build artifact | Generates a CycloneDX SBOM — a queryable inventory of every package in the image |
| **Trivy** | Build artifact | Scans OS packages and installed libraries against CVE databases — a different attack surface than Semgrep, which only sees application source |
| **OWASP ZAP** (full active scan) | Deployment | Crawls the running application and actively sends attack payloads (SQLi, XSS, command injection, path traversal) against every discovered input, then inspects responses for confirmed exploitation |
| **Falco — runtime mode** | Runtime | A second, independent Falco deployment with custom `values.yaml`, installed via Helm into the kind cluster, monitoring syscalls inside the running pod |
| **Helm** | Deployment packaging | Templated Kubernetes manifests with environment-specific values, replacing raw `kubectl apply` |

The two Falco deployments (CI and runtime) run independently, with separate configurations, and do not share rules or state.

**Deliberately excluded:**
- **Prometheus / Grafana / Loki** — PyGoat exposes no application metrics endpoint. Grafana/Loki was attempted for visualizing Falco's alerts but the `loki-stack` Helm chart's datasource auto-provisioning didn't reliably populate; rather than spend further time on a visualization layer, Falco's output is read directly via `kubectl logs`.
- **ArgoCD** — GitOps sync solves configuration drift against a persistent deployment target. This project deploys to a local, ephemeral kind cluster, so it doesn't solve a problem here.

## Why PyGoat

The pipeline is the artifact under test, not the application. PyGoat is used as the scan target because it has known, documented OWASP Top 10 vulnerabilities, which makes it possible to verify every scanner detects real issues rather than silently passing. A custom-written application risks looking clean simply because the author already avoids their own blind spots; PyGoat removes that bias.

## Results

Full pipeline run against PyGoat, with `ALLOW_CRITICAL_CVES=true`:

| Stage | Result |
|---|---|
| Secrets (Gitleaks) | PASS |
| SBOM (Syft) | PASS |
| Container CVEs (Trivy) | 62 critical, 903 high — gate bypassed (see Failure policy) |
| SAST (Semgrep) | 52 findings (14 critical) — gate bypassed (see Failure policy) |
| DAST (ZAP, active scan) | Findings reported |
| **Security Score** | **0 / 100** |

The score starts at 100 and deducts per finding (−10 per critical CVE, −3 per high CVE, −2 per Semgrep finding, −10 for a failed DAST job). Against PyGoat's finding volume this floors at 0 by mathematical certainty — the formula has not yet been exercised against a target with few enough findings to produce a discriminating, nonzero score.

Both Trivy and Semgrep produce a structured markdown report on every run: a summary table, every CRITICAL/ERROR finding in full, and HIGH/WARNING findings grouped by package or rule inside a collapsible section.

Cross-tool confirmation is visible in the data itself: Semgrep flags insecure deserialization in application code (`views.py`), and Trivy independently flags the vulnerable PyYAML dependency (three CRITICAL RCE-via-deserialization CVEs) that the code relies on — two tools catching the same risk class from two different angles.

**Runtime detection, captured live:**

```
Warning: Sensitive file opened for reading
process=cat command=cat /etc/shadow user=root user_uid=0
container_name=vaultforge-app k8s_pod_name=vaultforge-app-586949f867-dtgr5
```

Falco correctly attributed the process, user, and exact pod when a shell was used to read `/etc/shadow` inside the running container — confirming the runtime layer detects real, in-progress activity, not just "installed but idle."

## Local setup

**1. GitHub Actions configuration** (required before the pipeline can complete against PyGoat):

Go to the repo's **Settings → Secrets and variables → Actions → Variables** and add:
- Name: `ALLOW_CRITICAL_CVES`
- Value: `true`

Without this, the Trivy and Semgrep jobs will fail on their first run — this is expected, since PyGoat's CVE/finding counts are non-zero by design (see Failure policy above).

**2. Local cluster and deployment:**

```bash
git clone https://github.com/sanmathik8/vault-forge-.git
cd vault-forge-

# Create the kind cluster
kind create cluster --config kind/cluster-config.yaml

# Build and load the application image
docker build -t vault-forge-app:latest ./app
kind load docker-image vault-forge-app:latest --name vaultforge

# Deploy with Helm
helm install vaultforge ./helm/vaultforge-chart --kube-context kind-vaultforge

# Deploy Falco (runtime detection)
helm repo add falcosecurity https://falcosecurity.github.io/charts
kubectl create namespace falco --context kind-vaultforge
helm install falco falcosecurity/falco -f falco/values.yaml \
  --namespace falco --kube-context kind-vaultforge

# Access the application
kubectl port-forward svc/vaultforge-service 8000:8000 --context kind-vaultforge
# http://localhost:8000
```

**Requires:** Docker Desktop, WSL2 (if on Windows), `kind`, `helm`, and `kubectl` installed locally.

**To point the pipeline at a different application:** edit the `env:` block at the top of `.github/workflows/pipeline.yml` (`APP_PATH`, `IMAGE_NAME`, `APP_PORT`). This has not yet been tested end-to-end against a second application.

The GitHub Actions pipeline runs on every push to `main`. See the **Actions** tab for live output, the Security Summary, and downloadable Trivy/Semgrep reports.
