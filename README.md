# ALU Transcript Generator — Lab 2

**Course:** Introduction to Python Programming and Databases  
**Program:** Bachelor of Science in Software Engineering (BSE Year 1, Trimester 2)  
**Institution:** African Leadership University (ALU Rwanda)  
**Author:** Genereuse3  
**Repository:** [https://github.com/Genereuse3/lab2_Genereuse3.git](https://github.com/Genereuse3/lab2_Genereuse3.git)

---

## 1. Project Overview

The **ALU Transcript Generator** is an academic standing and grade evaluation system built using Python and Bash shell scripting. It automates two critical workflows in academic course management:

1. **Python Grade Evaluator (`grade-evaluator.py`)**:
   - Parses assignment grade records from a CSV file (`grades.csv`).
   - Validates that every individual score falls within the valid range ($0 \le \text{score} \le 100$).
   - Validates that category weight distributions match ALU's academic standards:
     - **Formative Assessments:** Exactly $60\%$
     - **Summative Assessments:** Exactly $40\%$
     - **Total Weight:** Exactly $100\%$
   - Computes weighted scores and total grade.
   - Computes final GPA on ALU's 5.0 scale using:
     $$\text{GPA} = \left(\frac{\text{Total Grade}}{100}\right) \times 5.0$$
   - Evaluates academic standing (**PASSED** vs **FAILED**): A student passes only if they score $\ge 50\%$ in **both** Formative and Summative categories.
   - Implements **Resubmission Logic**: Automatically identifies failed Formative assignments ($< 50\%$) that carry the highest weight. If multiple failed formative assignments share the same highest weight, all of them are presented as eligible.
   - Generates an official, beautifully formatted academic transcript in the terminal.

2. **Shell Script Organizer (`organizer.sh`)**:
   - Fully automates course data archiving and workspace preparation.
   - Verifies the existence of the `archive/` directory (creates it if missing).
   - Archives the processed `grades.csv` by renaming and moving it with a timestamp (e.g., `grades_20251105-170000.csv`).
   - Resets the workspace immediately by creating a fresh empty `grades.csv` for the next batch.
   - Maintains an append-only audit trail in `organizer.log` recording timestamps, original filenames, and destination archive paths.

---

## 2. Directory Structure

```text
lab2_Genereuse3/
├── grades.csv               # Active course grades CSV data source
├── grade-evaluator.py       # Core Python grade calculation and transcript engine
├── organizer.sh             # Bash shell script for archiving and workspace reset
├── organizer.log            # Persistent audit log created by organizer.sh
├── archive/                 # Directory holding timestamped archived grade CSV files
│   └── grades_*.csv
├── tests/                   # Automated test suite
│   └── test_grade_evaluator.py
└── Readme.md                # Project documentation and deployment guide
```

---

## 3. Data Source Specification (`grades.csv`)

The data file must be in comma-separated values (CSV) format with the following four columns:

| Column | Data Type | Description | Allowed Values |
|---|---|---|---|
| `assignment` | String | Name of the assignment or project | Non-empty text |
| `group` | String | Assessment category | `Formative` or `Summative` |
| `score` | Float | Student's earned percentage score | $0.0 \le \text{score} \le 100.0$ |
| `weight` | Float | Percentage weight of the assignment | Total Formative = 60, Summative = 40 |

### Standard Course Data (`grades.csv`):

```csv
assignment,group,score,weight
Quiz,Formative,85,20
Group Exercise,Formative,40,20
Functions and Debugging Lab,Formative,45,20
Midterm Project - Simple Calculator,Summative,70,20
Final Project - Text-Based Game,Summative,60,20
```

---

## 4. Evaluation and Calculation Rules

### A. Grade Validation
- Every assignment score must respect $0 \le \text{score} \le 100$.
- Non-numeric scores or out-of-range values trigger informative error messages and safe exit.

### B. Weight Validation
- Formative weights must sum to exactly **60.0%**.
- Summative weights must sum to exactly **40.0%**.
- Total weights must sum to exactly **100.0%**.
- Any mismatch halts execution with a detailed breakdown of expected vs. actual weights.

### C. GPA and Total Grade Calculation
- **Weighted Assignment Score**:
  $$\text{Weighted Score} = \frac{\text{Score} \times \text{Weight}}{100}$$
- **Total Grade**:
  $$\text{Total Grade} = \sum \text{Weighted Scores} \quad (\text{out of } 100)$$
- **ALU GPA**:
  $$\text{GPA} = \left(\frac{\text{Total Grade}}{100}\right) \times 5.0$$

### D. Final Decision (Pass / Fail)
- A student passes if and only if:
  $$\text{Formative \%} \ge 50\% \quad \text{AND} \quad \text{Summative \%} \ge 50\%$$
- If either category is $< 50\%$, the final standing is **FAILED**.
- If both categories are $\ge 50\%$, the final standing is **PASSED**.

### E. Resubmission Logic
- Only assignments in the **Formative** category with $\text{Score} < 50\%$ are considered.
- The system finds the maximum weight among these failed formative assignments.
- If multiple failed formative assignments share this maximum weight, **all** are displayed.
- If no formative assignments failed, the system displays that none are eligible.

---

## 5. Execution Guide

### Running the Python Grade Evaluator

#### Interactive Mode (prompts for filename):
```bash
python grade-evaluator.py
```
*Prompt:*
```text
Enter the name of the CSV file to process (e.g., grades.csv): grades.csv
```

#### Command-Line Argument Mode:
```bash
python grade-evaluator.py grades.csv
```

### Sample Output:
```text
--- Processing Grades ---
============================================================================
               AFRICAN LEADERSHIP UNIVERSITY
                 OFFICIAL ACADEMIC TRANSCRIPT
============================================================================
Assignment Name                        Category      Score   Weight  Weighted
----------------------------------------------------------------------------
Quiz                                   Formative     85.0%    20.0%     17.00
Group Exercise                         Formative     40.0%    20.0%      8.00
Functions and Debugging Lab            Formative     45.0%    20.0%      9.00
Midterm Project - Simple Calculator    Summative     70.0%    20.0%     14.00
Final Project - Text-Based Game        Summative     60.0%    20.0%     12.00
----------------------------------------------------------------------------
CATEGORY BREAKDOWN:
  * Formative Assessments: Earned 34.00 / 60.0 pts (56.67%) -> MEETS REQUIREMENT (>= 50%)
  * Summative Assessments: Earned 26.00 / 40.0 pts (65.00%) -> MEETS REQUIREMENT (>= 50%)
----------------------------------------------------------------------------
OVERALL PERFORMANCE:
  * Total Grade:  60.00 / 100.00
  * Final GPA:    3.00 / 5.00
  * Final Status: PASSED
----------------------------------------------------------------------------
RESUBMISSION ELIGIBILITY (Formative Only):
  The following formative assignment(s) failed (< 50%) with the highest weight (20.0%):
    - Group Exercise (Score: 40.0%, Weight: 20.0%)
    - Functions and Debugging Lab (Score: 45.0%, Weight: 20.0%)
============================================================================

Transcript generation complete.
```

---

## 6. Shell Script Organizer (`organizer.sh`)

The organizer script automates data lifecycle management in bash:

```bash
bash organizer.sh
```

### How It Works:
1. Verifies if the `archive/` directory exists; if not, creates it (`mkdir -p archive`).
2. Checks that `grades.csv` exists; if not, logs an error and exits cleanly.
3. Generates a timestamp formatted as `YYYYMMDD-HHMMSS`.
4. Moves `grades.csv` to `archive/grades_<timestamp>.csv`.
5. Creates a brand new, empty `grades.csv` (`touch grades.csv`) ready for new inputs.
6. Appends an audit entry to `organizer.log`:
   ```text
   [2026-11-09 11:15:30] Operation: ARCHIVE | Original: grades.csv | Archived: archive/grades_20261109-111530.csv
   ```

---

## 7. Running Automated Unit Tests

A comprehensive unit test suite is included in `tests/test_grade_evaluator.py`:

```bash
python -m unittest tests/test_grade_evaluator.py
```

Tests cover:
- Standard calculations (GPA, Total Grade, Pass status).
- Score validation (rejection of scores $< 0$ or $> 100$).
- Weight validation (rejection of weights not equaling 60/40/100).
- Failing status detection when either category drops below $50\%$.
- Resubmission tie handling and filtering for the highest-weight failed formative.

---

## 8. Deployment & GitHub Push Instructions

When you are ready to push this project to your GitHub repository:

### Step 1: Initialize Git and Set Branch
```bash
cd lab2_Genereuse3
git init
git branch -M main
```

### Step 2: Configure Git Identity
```bash
git config user.name "Genereuse3"
git config user.email "your_email@example.com"
```

### Step 3: Add and Commit All Files
```bash
git add .
git commit -m "feat(lab2): complete ALU Transcript Generator and Shell Organizer"
```

### Step 4: Link Remote Repository and Push
```bash
git remote add origin https://github.com/Genereuse3/lab2_Genereuse3.git
git push -u origin main
```

*(If prompted for credentials, use your GitHub username and Personal Access Token (PAT) with `repo` scope as the password).*
