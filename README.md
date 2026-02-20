# excel-anonymizer

CLI for creating PII-safe Excel test fixtures.

Analyzes Excel files and anonymizes sensitive data (names, organizations, financials)
while preserving structure and statistical properties.

## Install

```bash
pip install git+https://github.com/your-org/excel-anonymizer.git
```

Or with uv:
```bash
uv add git+https://github.com/your-org/excel-anonymizer.git
```

## Workflow

### Step 1: Analyze your file

```bash
excel-anon analyze "My Report.xlsx"
```

This prints a prompt. Copy it into Claude or ChatGPT.

### Step 2: Save the config

The LLM returns a config dict. Save it to a Python file:

```python
# my_config.py
from excel_anonymizer import PercentageVarianceRule, PreserveRelationshipRule

config = {
    "version": "1.0.0",
    "sheets_to_keep": ["Sheet1"],
    "entity_columns": {
        "Manager": "PERSON",
        "Client": "ORGANIZATION",
    },
    "numeric_rules": {
        "Revenue": PercentageVarianceRule(variance_pct=0.3),
        "Margin": PreserveRelationshipRule(
            formula="context['Revenue'] - context['Cost']",
            dependent_columns=["Revenue", "Cost"],
        ),
    },
    "preserve_columns": ["Date", "Project Type"],
}
```

### Step 3: Anonymize

```bash
excel-anon process "My Report.xlsx" --config my_config.py
# Output: My Report_SYNTHETIC.xlsx
```

## Entity Types

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

## Commands

```
excel-anon analyze <file>              Analyze file, generate LLM prompt
excel-anon analyze <file> -o out.txt   Save prompt to file
excel-anon analyze-multi f1 f2 f3      Analyze multiple files for schema patterns
excel-anon process <file> --config c   Anonymize file using config
excel-anon process <file> out.xlsx --config c --seed 123
```
