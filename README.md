# excel-anon

You have a real Excel report full of employee names, client organizations, and revenue figures. You need a test fixture that matches its structure. `excel-anon` strips the sensitive data and replaces it with realistic fakes, preserving column layouts, data types, and numeric relationships like Gross Margin = Revenue - Cost.

## Install

```bash
pip install git+https://github.com/your-org/excel-anonymizer.git
```

```bash
uv add git+https://github.com/your-org/excel-anonymizer.git
```

## How it works

`excel-anon` uses an LLM to author the anonymization config, so you never write column mappings by hand. Run `analyze` on your file and it prints a prompt describing the file's columns and sample data. Paste that prompt into Claude or ChatGPT and the LLM returns a config dict you save to a `.py` file. Then run `process` with that config and the tool writes a `_SYNTHETIC.xlsx` beside the original.

## Quick start

**Step 1: Analyze the file**

```bash
excel-anon analyze "Q4 Expense Report.xlsx"
```

The command prints a prompt to your terminal. Copy it.

**Step 2: Paste into an LLM, save the config**

The LLM returns a config dict. Save it:

```python
# q4_expense_config.py
from excel_anonymizer import PercentageVarianceRule, PreserveRelationshipRule

config = {
    "version": "1.0.0",
    "sheets_to_keep": ["Expenses"],
    "entity_columns": {
        "Employee Name": "PERSON",
        "Department": "ORGANIZATION",
        "Manager": "PERSON",
    },
    "numeric_rules": {
        "Reimbursement": PercentageVarianceRule(variance_pct=0.2),
        "Net Amount": PreserveRelationshipRule(
            formula="context['Reimbursement'] - context['Deduction']",
            dependent_columns=["Reimbursement", "Deduction"],
        ),
    },
    "preserve_columns": ["Date", "Category"],
}
```

**Step 3: Process the file**

```bash
excel-anon process "Q4 Expense Report.xlsx" --config q4_expense_config.py
# Output: Q4 Expense Report_SYNTHETIC.xlsx
```

## Config reference

### Entity types

Assign an entity type to each column containing identifiable data. The tool generates a consistent fake value per unique input, so the same name always maps to the same fake name within a file.

| Type | Generates |
|------|-----------|
| `PERSON` | Full name |
| `PERSON_FIRST_NAME` | First name only |
| `PERSON_LAST_NAME` | Last name only |
| `ORGANIZATION` | Company name |
| `EMAIL_ADDRESS` | Email address |
| `PHONE_NUMBER` | Phone number |
| `PROJECT_NAME` | Project name |
| `LOCATION` | City, State |

### Numeric rules

**`PercentageVarianceRule`** replaces each value with a random value within a percentage band of the original. Use it for independent figures like headcount or individual expenses.

```python
"Headcount": PercentageVarianceRule(variance_pct=0.15)
# A value of 100 becomes a random number between 85 and 115.
```

**`PreserveRelationshipRule`** derives a value from other already-anonymized columns using a formula. Use it for any column that is computed from other columns, so the arithmetic stays consistent in the output file.

```python
"Gross Margin": PreserveRelationshipRule(
    formula="context['Revenue'] - context['Cost']",
    dependent_columns=["Revenue", "Cost"],
)
# Gross Margin will always equal the anonymized Revenue minus the anonymized Cost.
```

## Commands

| Command | Description |
|---------|-------------|
| `excel-anon analyze <file>` | Analyze file and print LLM prompt |
| `excel-anon analyze <file> -o prompt.txt` | Save LLM prompt to a file |
| `excel-anon analyze-multi f1 f2 f3` | Analyze multiple files for shared schema patterns |
| `excel-anon process <file> --config config.py` | Anonymize file using config |
| `excel-anon process <file> out.xlsx --config config.py --seed 42` | Write to named output with fixed random seed |
