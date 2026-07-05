# Ghana Maternal Health Q&A Dataset (English)
## 20,000 Questions & Answers for Maternal Healthcare

---

## 📋 DATASET OVERVIEW

This dataset contains **20,000 unique question-answer pairs** for maternal health care in Ghana, presented in English and designed for AI training, health education, and clinical support systems.

### Key Statistics
- **Total Q&A Pairs**: 20,000
- **Organization**: 40 batches × 500 pairs each
- **Language**: English
- **Domain**: Maternal Health (Ghana)
- **ID Range**: Q00001 to Q20000
- **Total Size**: ~11MB (all JSON files)

---

## 📁 FILE STRUCTURE

```
maternal_health_dataset/
├── README.md                               # This file
├── batches/
│   ├── batch_01_maternal_health_qa.json   # Q00001-Q00500
│   ├── batch_02_maternal_health_qa.json   # Q00501-Q01000
│   ├── batch_03_maternal_health_qa.json   # Q01001-Q01500
│   ├── ...
│   └── batch_40_maternal_health_qa.json   # Q19501-Q20000
```

---

## 🎯 TOPIC COVERAGE

### Major Categories (17 topics):

1. **Hypertension & Pre-eclampsia** (~1,200 pairs)
   - Blood pressure monitoring
   - Pre-eclampsia danger signs
   - Emergency management

2. **Bleeding & Hemorrhage** (~1,200 pairs)
   - Antepartum hemorrhage
   - Placenta previa/abruption
   - Postpartum bleeding

3. **Routine Antenatal Care** (~2,300 pairs)
   - ANC visit schedules
   - Standard screenings
   - Health education

4. **Nutrition & Diet** (~1,500 pairs)
   - Pregnancy nutrition
   - Food safety
   - Supplement guidance

5. **Fetal Movement** (~800 pairs)
   - Kick counts
   - Reduced movement
   - Normal patterns

6. **Common Pregnancy Symptoms** (~2,000 pairs)
   - Morning sickness
   - Back pain
   - Swelling
   - Itching

7. **Labor & Delivery** (~1,500 pairs)
   - Labor signs
   - Pain management
   - Delivery options

8. **Postpartum Care** (~1,200 pairs)
   - Recovery
   - Danger signs
   - Return to activities

9. **Breastfeeding** (~1,000 pairs)
   - Latch techniques
   - Supply issues
   - Working mothers

10. **Family Planning** (~800 pairs)
    - Contraceptive methods
    - Timing after delivery
    - Side effects

11. **HIV & PMTCT** (~600 pairs)
    - Prevention strategies
    - ARV treatment
    - Testing protocols

12. **Malaria Prevention** (~400 pairs)
    - IPTp protocols
    - Net usage
    - Treatment safety

13. **Multiple Pregnancy** (~400 pairs)
    - Twin care
    - Delivery planning
    - Complications

14. **Gestational Diabetes** (~400 pairs)
    - Screening
    - Management
    - Long-term risks

15. **Teenage Pregnancy** (~400 pairs)
    - Special risks
    - Social support
    - Education continuity

16. **Traditional Practices** (~300 pairs)
    - Herb safety
    - Cultural beliefs
    - Safe practices

17. **Maternal Mental Health** (~300 pairs)
    - Depression
    - Anxiety
    - Support resources

---

## 📊 METADATA FIELDS

Each Q&A pair includes comprehensive metadata:

```json
{
  "id": "Q00001",
  "question": "I just found out I'm pregnant. When should I go for my first antenatal visit?",
  "answer": "Congratulations! You should go for your first antenatal visit as soon as possible...",
  "metadata": {
    "topic": "antenatal_care",
    "language": "english",
    "pregnancy_stage": "first_trimester",
    "urgency": "urgent",
    "setting": "Health Center",
    "keywords": ["antenatal", "pregnancy", "visit"],
    "generated_batch": 1
  }
}
```

### Metadata Categories:

**Urgency Levels:**
- `emergency` - Immediate life-threatening situations
- `urgent` - Needs prompt attention within hours
- `moderate` - Should be addressed within days
- `routine` - Regular care and information

**Pregnancy Stages:**
- `first_trimester` (0-13 weeks)
- `second_trimester` (14-27 weeks)
- `third_trimester` (28-40 weeks)
- `labor` (Active labor and delivery)
- `postpartum` (After delivery)
- `any_stage` (Applicable throughout)

**Care Settings:**
- CHPS (Community-based Health Planning and Services)
- Health Center
- District Hospital
- Regional Hospital

---

## 💡 USE CASES

### Primary Applications:

1. **AI Chatbot Training**
   - Train conversational AI for maternal health
   - Fine-tune language models for Ghanaian context
   - Build retrieval-augmented generation (RAG) systems

2. **Mobile Health Apps**
   - Question-answering systems
   - Symptom checkers
   - Health education platforms

3. **Clinical Decision Support**
   - Provider training tools
   - Triage assistance
   - Patient education resources

4. **Research & Analysis**
   - NLP model evaluation
   - Health communication research

---

## ⚠️ IMPORTANT DISCLAIMERS

### Medical Accuracy
- Based on WHO guidelines and Ghana Health Service protocols
- **Should NOT replace professional medical advice**
- Always direct users to seek professional care for serious concerns
- Intended for educational and support purposes

### Clinical Validation
- Generated using evidence-based templates
- Reflects current best practices (as of 2025)
- Should be reviewed by obstetric professionals before deployment
- Regular updates needed to reflect protocol changes

### Cultural Sensitivity
- Respects Ghanaian cultural contexts
- Balances traditional and modern practices
- Promotes health facility usage
- Addresses common local concerns

---

## 🔧 TECHNICAL SPECIFICATIONS

### JSON Structure
```json
{
  "batch_metadata": {
    "batch_number": 1,
    "id_range": "Q00001-Q00500",
    "total_pairs": 500,
    "generation_date": "2025-11-24T...",
    "language": "english",
    "domain": "maternal_health_ghana"
  },
  "qa_pairs": [
    {
      "id": "Q00001",
      "question": "...",
      "answer": "...",
      "metadata": {...}
    }
  ]
}
```

### Quality Assurance
- Hash-based duplicate detection
- Template variation system
- Realistic clinical values
- Keywords extracted from questions only

---

## 📈 DATASET STATISTICS

### Distribution Summary:
- **Emergency Cases**: 15% (3,000 pairs)
- **Urgent Cases**: 20% (4,000 pairs)
- **Moderate Concern**: 25% (5,000 pairs)
- **Routine Care**: 40% (8,000 pairs)

### Coverage by Trimester:
- **First Trimester**: 20% (4,000 pairs)
- **Second Trimester**: 25% (5,000 pairs)
- **Third Trimester**: 25% (5,000 pairs)
- **Labor & Delivery**: 10% (2,000 pairs)
- **Postpartum**: 15% (3,000 pairs)
- **Any Stage**: 5% (1,000 pairs)

### Care Settings:
- **CHPS**: 40% (8,000 pairs)
- **Health Center**: 25% (5,000 pairs)
- **District Hospital**: 20% (4,000 pairs)
- **Regional Hospital**: 15% (3,000 pairs)

---

## 🚀 GETTING STARTED

### Loading a Batch
```python
import json

# Load specific batch
with open('batches/batch_01_maternal_health_qa.json', 'r', encoding='utf-8') as f:
    batch = json.load(f)

# Access Q&A pairs
for pair in batch['qa_pairs']:
    print(f"Q: {pair['question']}")
    print(f"A: {pair['answer']}")
    print(f"Topic: {pair['metadata']['topic']}")
    print("-" * 50)
```

### Loading All Batches
```python
import json
from pathlib import Path

all_pairs = []
batch_dir = Path('batches')

for batch_file in sorted(batch_dir.glob('batch_*.json')):
    with open(batch_file, 'r', encoding='utf-8') as f:
        data = json.load(f)
        all_pairs.extend(data['qa_pairs'])

print(f"Loaded {len(all_pairs)} total Q&A pairs")
```

### Filtering by Topic
```python
# Get all emergency cases
emergency_pairs = [
    pair for pair in all_pairs 
    if pair['metadata']['urgency'] == 'emergency'
]

# Get specific topic
hypertension_pairs = [
    pair for pair in all_pairs 
    if 'hypertension' in pair['metadata']['topic']
]
```

---

## 🤝 CONTRIBUTING & FEEDBACK

### Validation Needed:
- Clinical accuracy review by OB/GYN professionals
- Cultural appropriateness assessment
- Usability testing with target users

### Potential Improvements:
- Additional language variants (Twi, Ga, Ewe, Dagbani)
- More edge cases and rare conditions
- Updated protocols and guidelines
- User feedback integration

---

## 📄 LICENSE & USAGE

**Educational & Non-Commercial Use:**
- Free to use for research, education, and healthcare improvement
- Attribution appreciated
- Not for commercial use without permission

**Clinical Deployment:**
- Must be validated by qualified healthcare professionals
- Should include disclaimers about professional medical advice
- Regular updates required to maintain accuracy

---

## 🌟 IMPACT POTENTIAL

This dataset represents:
- **Comprehensive** maternal health Q&A resource for Ghana
- **Structured** for AI/ML training and evaluation
- **Evidence-based** content from WHO and Ghana Health Service
- Potential to improve maternal health outcomes for millions of Ghanaian women

**Together, we can save lives through accessible health information! 🇬🇭**
# vault-forge-
