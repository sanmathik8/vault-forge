# 🛡️ SecureFlow - Reusable DevSecOps Security Pipeline

> **A reusable GitHub Actions DevSecOps pipeline that can secure any Dockerized application with minimal configuration.**

SecureFlow automates security testing throughout the CI pipeline by combining secret detection, Software Bill of Materials (SBOM) generation, container vulnerability scanning, static application security testing (SAST), dynamic application security testing (DAST), automated security gates, GitHub issue creation, and a consolidated security summary.

Instead of building security separately for every project, SecureFlow acts as a **drop-in GitHub Actions workflow** that can be reused across multiple applications.

---

# 🚀 Features

* 🔐 Secret Scanning (Gitleaks)
* 🐳 Docker Image Build
* 📦 Software Bill of Materials (SBOM) Generation
* 🛡️ Container Vulnerability Scanning (Trivy)
* 🔍 Static Application Security Testing (Semgrep)
* 🌐 Dynamic Application Security Testing (OWASP ZAP)
* 🚧 Automatic Security Gates
* 📄 Security Reports
* 📊 Security Score
* 📝 Automatic GitHub Issue Creation
* ♻️ Reusable Across Multiple Applications

---

# 📌 Pipeline Workflow

```text
Developer Push
      │
      ▼
Checkout Repository
      │
      ▼
Secrets Scan (Gitleaks)
      │
      ▼
Docker Image Build
      │
      ▼
Generate SBOM (Syft)
      │
      ▼
Container Scan (Trivy)
      │
      |
      ▼              
SAST (Semgrep)       
      │              
      ▼              
Run Application
      │
      ▼
DAST (OWASP ZAP)
      │
      ▼
Security Gates
      │
      ▼
Generate Reports
      │
      ▼
Create GitHub Issue (if critical findings exist)
      │
      ▼
Security Summary
```

---

# ⚙️ Configuration

SecureFlow is designed to work with **any Dockerized application**.

Only update the following variables:

```yaml
APP_PATH: ./app
IMAGE_NAME: my-application
APP_PORT: 8000
```

No workflow changes are required.

---

# 🛠 Security Stack

| Stage                        | Tool                   |
| ---------------------------- | ---------------------- |
| Secret Detection             | Gitleaks               |
| Container Build              | Docker                 |
| SBOM                         | Syft                   |
| Container Vulnerability Scan | Trivy                  |
| Static Code Analysis         | Semgrep                |
| Runtime Monitoring           | Falco                  |
| Dynamic Security Testing     | OWASP ZAP              |
| Reporting                    | GitHub Actions Summary |
| Issue Tracking               | GitHub Issues          |

---

# 📁 Project Structure

```text
.github/
│
├── workflows/
│     └── pipeline.yml
│
├── actions/
│     └── graceful-exit/
│
app/
│
Dockerfile
│
README.md
```

---

# 🔄 Security Pipeline Stages

## 1️⃣ Secret Detection

Detects exposed credentials, API keys, passwords, and tokens before the application is built.

---

## 2️⃣ Docker Build

Builds the application into a Docker image for security analysis.

---

## 3️⃣ SBOM Generation

Creates a Software Bill of Materials to provide complete visibility into application dependencies.

---

## 4️⃣ Container Vulnerability Scan

Scans the Docker image for:

* Critical CVEs
* High Severity CVEs

Automatically blocks the pipeline when critical vulnerabilities are detected.

---

## 5️⃣ Static Application Security Testing

Analyzes source code for:

* Injection flaws
* Hardcoded secrets
* Insecure coding patterns
* OWASP Top 10 issues

---

## 6️⃣ Dynamic Application Security Testing

Launches the application inside a temporary Docker container and performs active security testing using OWASP ZAP.

---

## 7️⃣ Security Gates

The pipeline automatically fails when:

* Secrets are detected
* Critical CVEs exist
* Critical SAST findings are discovered

---

## 8️⃣ Automated GitHub Issues

Critical findings automatically create GitHub Issues containing:

* Vulnerability counts
* Build information
* Workflow links
* Security reports

---

## 9️⃣ Security Summary

At the end of every workflow, SecureFlow generates:

* Stage Status
* Critical Vulnerabilities
* High Vulnerabilities
* SAST Findings
* DAST Status
* Security Score

---

# 📊 Generated Artifacts

* Docker Image
* SBOM Report
* Trivy Report (JSON & Markdown)
* Semgrep Report
* GitHub Job Summary
* Security Score

---

# 🚀 Running the Pipeline

```bash
git clone https://github.com/<your-username>/SecureFlow.git

cd SecureFlow

git push origin main
```

GitHub Actions automatically executes the complete security pipeline.

---

# 💡 Use Cases

* Secure CI/CD Pipelines
* DevSecOps Demonstrations
* Security Automation
* Docker Application Scanning
* Open Source Projects
* Enterprise CI Pipelines
* Cloud Native Applications
* Student DevSecOps Projects

---



