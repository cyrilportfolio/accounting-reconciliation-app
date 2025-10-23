# Accounting Reconciliation Application - Agent Instructions

## Project Overview

This is a professional accounting reconciliation web application that validates Excel transaction data against defined business rules.

**Purpose:** Portfolio project demonstrating full-stack development with AI assistance  
**Target Users:** Accountants and bookkeepers  
**Key Feature:** Automated validation of accounting data with detailed error reporting

---

## Core Commands

### Installation

```bash
# Create virtual environment (recommended) 
python3 -m venv venv 
source venv/bin/activate # On Windows: venv\Scripts\activate 

# Upgrade pip and install dependencies 
python -m pip install --upgrade pip 
python -m pip install -r requirements.txt
```
### Development
```bash
# Run development server (ASGI)
python3 -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Alternative: run module (only if app.main has a runnable __main__)
python -m app.main
```

### Testing
```bash
# Run all tests
python -m pytest tests/ -v

# Run with coverage
python -m pytest tests/ --cov=app --cov-report=html

# Run specific test file
python -m pytest tests/test_reconciliation.py -v
```

### Code Quality
```bash
# Linting
python -m flake8 app/ tests/

# Type checking
python -m mypy app/

# Format code
python -m black app/ tests/
```

### Docker
```bash
# Build image
docker build -t accounting-reconciliation-app .

# Run container
docker run -p 8000:8000 accounting-reconciliation-app

# Docker Compose
docker-compose up
```

---

## Project Structure
```tree
accounting-reconciliation-app/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI application entry
│   ├── config.py            # Configuration settings
│   ├── models.py            # Pydantic data models
│   ├── reconciliation.py    # Core business logic
│   ├── report_generator.py  # Report generation
│   └── utils.py             # Helper functions
├── static/
│   ├── css/style.css
│   ├── js/main.js
│   └── images/
├── templates/
│   ├── index.html           # Upload interface
│   └── results.html         # Results display
├── tests/
│   ├── test_reconciliation.py
│   └── test_api.py
├── docs/
│   ├── reconciliation_logic.md
│   └── API_DOCUMENTATION.md
├── sample_data/             # Test data (gitignored)
├── uploads/                 # Temp uploads (gitignored)
├── outputs/                 # Generated reports (gitignored)
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
├── .gitignore
├── AGENTS.md               # This file
└── README.md
```

---

## Development Patterns & Rules

### Code Style
- **Python Style Guide:** PEP 8
- **Line Length:** Maximum 100 characters
- **Type Hints:** Required for all function signatures
- **Docstrings:** Required for all public functions (Google style)
- **Imports:** Organize as: standard library, third-party, local
- **Naming:**
  - Functions: `snake_case`
  - Classes: `PascalCase`
  - Constants: `UPPER_SNAKE_CASE`
  - Private methods: `_leading_underscore`

### Excel Processing Rules
- **File Validation:**
  - Accept only .xlsx and .xls formats
  - Maximum file size: 10MB
  - Validate structure before processing
- **Data Handling:**
  - Use Pandas for all Excel operations
  - Always close file handles properly
  - Handle encoding issues gracefully
- **Memory Management:**
  - Process large files in chunks if needed
  - Clear DataFrames after processing

### Reconciliation Logic (CRITICAL)
- **Monetary Calculations:**
  - ⚠️ **NEVER use Python float for money**
  - ✅ **ALWAYS use Decimal type from decimal module**
  - Round all calculations to 2 decimal places
  - Example: `from decimal import Decimal, ROUND_HALF_UP`
- **Validation Checks:**
  - All 8 checks defined in docs/reconciliation_logic.md must be implemented
  - Each check must be independent (can run separately)
  - Log all validation failures with transaction IDs
  - Maintain error severity levels: CRITICAL, WARNING, INFO
- **Error Handling:**
  - Never silently fail validation checks
  - Always log errors with context
  - Provide actionable error messages to users

### API Design
- **Endpoints:**
  - `/` - Upload interface (GET)
  - `/api/upload` - File upload (POST)
  - `/api/reconcile` - Process file (POST)
  - `/api/download/{report_id}` - Download reports (GET)
  - `/api/health` - Health check (GET)
- **Response Format:**
```json
  {
    "success": boolean,
    "data": object,
    "errors": array,
    "metadata": object
  }
```
- **Status Codes:**
  - 200: Success
  - 400: Bad request (validation failed)
  - 422: Unprocessable entity (invalid data)
  - 500: Server error

### Security & Data Handling
- **File Upload Security:**
  - Validate file extensions (whitelist: .xlsx, .xls)
  - Scan for malicious content patterns
  - Limit file size to 10MB
  - Use secure temporary storage
  - Delete uploaded files after processing
- **Data Privacy:**
  - Never log sensitive financial data
  - Redact account numbers in logs
  - Clear temporary files on errors
- **Error Messages:**
  - Don't expose internal paths
  - Don't reveal stack traces to users
  - Sanitize all user-facing error messages
- **Environment Variables:**
  - Store sensitive config in .env file (gitignored)
  - Never commit API keys or passwords
  - Use python-dotenv for loading config

### Testing Requirements
- **Minimum Coverage:** 80% code coverage required
- **Test Organization:**
  - Unit tests: Test individual functions
  - Integration tests: Test API endpoints
  - End-to-end tests: Test full workflow
- **Test Data:**
  - Use sample_data/sample_accounting_data.xlsx for integration tests
  - Create fixtures for unit tests
  - Mock external dependencies
- **Validation Testing:**
  - Test each of the 8 validation checks independently
  - Test with valid data (should pass)
  - Test with invalid data (should fail correctly)
  - Test edge cases (empty files, malformed data, etc.)

### Frontend Guidelines
- **Framework:** Vanilla JavaScript or Alpine.js (keep it simple)
- **Styling:** Bootstrap 5 or Tailwind CSS
- **Responsiveness:** Mobile-first design
- **User Experience:**
  - Clear upload instructions
  - Progress indicators during processing
  - Sortable, searchable results table
  - Clear error messages
  - Download buttons for reports
- **File Upload:**
  - Support drag-and-drop
  - Show file name and size before upload
  - Validate client-side before submission
  - Show upload progress

### Git Workflow
- **Branch Strategy:**
  - `main` - Production-ready code
  - `develop` - Integration branch
  - `feature/<name>` - New features
  - `bugfix/<name>` - Bug fixes
- **Commit Messages:**
  - Format: `Type: Brief description`
  - Types: `Add`, `Fix`, `Update`, `Refactor`, `Test`, `Docs`
  - Example: `Add: Excel file upload validation`
  - Example: `Fix: Decimal precision in reconciliation`
- **Before Committing:**
  - Run tests: `pytest tests/`
  - Run linter: `flake8 app/`
  - Review changes: `git diff`
- **Never Commit:**
  - `uploads/` directory
  - `outputs/` directory
  - `.env` file
  - `__pycache__/` directories
  - `.pyc` files
  - Development databases

---

## External Services & Dependencies

### Required Python Packages
```
fastapi>=0.104.0
uvicorn[standard]>=0.24.0
pandas>=2.0.0
openpyxl>=3.1.0
pydantic>=2.0.0
python-multipart>=0.0.6
reportlab>=4.0.0
pytest>=7.4.0
pytest-cov>=4.1.0
httpx>=0.25.0  # For testing
```

### Development Dependencies
```
black>=23.0.0
flake8>=6.1.0
mypy>=1.7.0
python-dotenv>=1.0.0
```

### File Storage
- **Temporary Uploads:** `./uploads/` directory
- **Generated Reports:** `./outputs/` directory
- **Cleanup:** Delete files older than 1 week

### Database
- **Development:** SQLite (optional, for storing history)
- **Production:** PostgreSQL (future enhancement)

---

## Domain Knowledge: Accounting Reconciliation

### Double-Entry Bookkeeping Basics
- Every transaction has debits and credits
- Total debits must equal total credits
- This is the foundation of Rule 1 validation

### Account Code Format
- Format: 4-digit numeric codes (e.g., "1100", "5200")
- First digit indicates account type:
  - 1xxx: Assets
  - 2xxx: Liabilities
  - 3xxx: Equity
  - 4xxx: Revenue
  - 5xxx: Expenses

### Transaction Components
- **Transaction ID:** Unique identifier
- **Date:** Transaction date (format: YYYY-MM-DD or MM/DD/YYYY)
- **Account Code:** 4-digit account identifier
- **Description:** Text description of transaction
- **Debit Amount:** Amount debited (if any)
- **Credit Amount:** Amount credited (if any)
- **Reference:** Optional reference number

### Common Errors to Detect
1. Unbalanced entries (debits ≠ credits)
2. Duplicate transactions (same ID used twice)
3. Missing required fields
4. Invalid date formats
5. Negative amounts in wrong columns
6. Invalid account codes
7. Amounts in both debit and credit columns
8. Unusual amounts (potentially typos)

---

## Common Issues & Solutions

### Issue: Large Excel Files Timeout
**Cause:** Loading entire file into memory at once  
**Solution:** Process files in chunks using pandas `chunksize` parameter

### Issue: Excel Date Format Problems
**Cause:** Excel stores dates as numbers  
**Solution:** Use `pandas.to_datetime()` with `errors='coerce'`

### Issue: Decimal Precision Lost
**Cause:** Using float for monetary calculations  
**Solution:** Use Decimal type throughout:
```python
from decimal import Decimal, ROUND_HALF_UP

amount = Decimal('123.456')
rounded = amount.quantize(Decimal('0.01'), rounding=ROUND_HALF_UP)
```

### Issue: Memory Leak with Large Files
**Cause:** Not closing file handles  
**Solution:** Use context managers:
```python
with pd.ExcelFile('file.xlsx') as xls:
    df = pd.read_excel(xls)
```

### Issue: Upload Fails Silently
**Cause:** Missing error handling  
**Solution:** Wrap file operations in try-except and log errors

---

## Performance Targets

- **File Upload:** < 2 seconds for files up to 10MB
- **Processing:** < 5 seconds for 1000 transactions
- **Report Generation:** < 3 seconds
- **API Response Time:** < 100ms for health checks
- **Memory Usage:** < 500MB for typical files

---

## Important Notes for AI Agents

### When Building This Application:
1. **Read `docs/reconciliation_logic.md` first** - Contains critical business rules
2. **Never skip validation checks** - All 8 checks are required
3. **Use Decimal for money** - Float will cause precision issues
4. **Test incrementally** - Build and test one feature at a time
5. **Ask for clarification** - If reconciliation rules are unclear, ask
6. **Follow the structure** - Use the project layout defined above
7. **Write tests first** - For each validation rule, write tests before implementation
8. **Document decisions** - Add comments explaining complex logic

### Priorities (in order):
1. Core reconciliation logic (must be 100% correct)
2. Data validation and error handling
3. API endpoints and file handling
4. Report generation
5. Frontend interface
6. Docker deployment
7. Additional features

### Quality Checklist Before Marking "Done":
- [ ] All 8 validation checks implemented
- [ ] Tests pass with sample_accounting_data.xlsx
- [ ] All intentional errors in sample data are detected
- [ ] No false positives on valid transactions
- [ ] Reports generate correctly (Excel + PDF)
- [ ] Code passes linting (flake8)
- [ ] Test coverage ≥ 80%
- [ ] API documentation is complete
- [ ] README is comprehensive
- [ ] Docker build succeeds

---

## Quick Reference Commands

```bash
# Full development workflow
python3 -m venv venv && source venv/bin/activate
python -m pip install -r requirements.txt
python -m pytest tests/ -v
uvicorn app.main:app --reload
<tool>` ensures the tool is run with the same interpreter/venv that installed it.

# Docker workflow
docker build -t accounting-app .
docker run -p 8000:8000 accounting-app

# Testing the application
curl -X POST http://localhost:8000/api/upload -F "file=@sample_data/sample_accounting_data.xlsx"

# Clean up temporary files
rm -rf uploads/* outputs/*
```

