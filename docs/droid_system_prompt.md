# System Prompt for Droid: Accounting Reconciliation Application

## Project Overview

You are tasked with building a **professional accounting reconciliation web application** that allows accountants to upload Excel files containing accounting transactions, automatically validates the data against defined rules, and generates comprehensive error reports.

This is a portfolio project for an accountant learning AI coding agents. The final application should be production-quality, well-documented, and showcase best practices in software development.

---

## Project Context

**What you're building**: A web-based accounting reconciliation tool that automates the tedious process of checking accounting data for errors.

**Who it's for**: Accountants and bookkeepers who need to validate transaction data before financial reporting.

**Why it matters**: Manual reconciliation is time-consuming and error-prone. This tool will save hours of work and catch errors that humans might miss.

---

## Complete Technical Specification

### Technology Stack

**Backend**:
- **Python 3.10+** - Primary programming language
- **FastAPI** - Modern web framework (preferred for its automatic API documentation)
- **Pandas** - Data processing and validation
- **OpenPyXL** - Excel file handling
- **ReportLab or FPDF2** - PDF generation for error reports
- **Pydantic** - Data validation and settings management

**Frontend**:
- **HTML5/CSS3/JavaScript** - Core web technologies
- **Bootstrap 5** or **Tailwind CSS** - Professional styling framework
- **Vanilla JavaScript** or **Alpine.js** - For interactivity (keep it simple)
- **DataTables.js** or similar - For displaying data in tables
- **Chart.js** (optional) - For visualizing error statistics

**Deployment**:
- **Docker** - Containerization
- **Docker Compose** - Multi-container orchestration (if needed)
- **Nginx** (optional) - Reverse proxy (can be added later)

---

## Project Structure

Create the following directory structure:

```
accounting-reconciliation-app/
│
├── app/
│   ├── __init__.py
│   ├── main.py                 # FastAPI application entry point
│   ├── config.py               # Configuration settings
│   ├── models.py               # Pydantic models
│   ├── reconciliation.py       # Core reconciliation logic
│   ├── report_generator.py     # PDF report generation
│   └── utils.py                # Helper functions
│
├── static/
│   ├── css/
│   │   └── style.css           # Custom styles
│   ├── js/
│   │   └── main.js             # Frontend JavaScript
│   └── images/
│       └── logo.png            # Application logo (optional)
│
├── templates/
│   ├── index.html              # Upload page
│   └── results.html            # Results display page
│
├── uploads/                    # Temporary file storage (gitignored)
├── outputs/                    # Generated reports (gitignored)
│
├── tests/
│   ├── __init__.py
│   ├── test_reconciliation.py  # Unit tests
│   └── test_api.py             # API endpoint tests
│
├── docs/
│   └── API_DOCUMENTATION.md    # API documentation
│
├── Dockerfile                  # Docker container definition
├── docker-compose.yml          # Docker Compose configuration
├── requirements.txt            # Python dependencies
├── .dockerignore               # Docker ignore file
├── .gitignore                  # Git ignore file
├── README.md                   # Project documentation
└── LICENSE                     # MIT or Apache 2.0 license
```

---

## Detailed Implementation Requirements

### 1. Backend Implementation (FastAPI)

#### File: `app/main.py`

**Requirements**:
- Create FastAPI application instance
- Configure CORS for development
- Set up static files and templates
- Implement the following endpoints:

```python
# Endpoints to implement:

GET  /                          # Serve upload page (index.html)
POST /upload                    # Handle file upload
GET  /results/{session_id}      # Display results page
GET  /api/process/{session_id}  # Get processing results (JSON)
GET  /download/excel/{session_id}  # Download validated Excel file
GET  /download/pdf/{session_id}    # Download PDF error report
GET  /health                    # Health check endpoint
```

**Key Features**:
- Use `UploadFile` from FastAPI for file handling
- Generate unique session IDs for each upload (use UUID)
- Store uploaded files temporarily with session ID
- Implement proper error handling and validation
- Use async/await where appropriate
- Include detailed logging

#### File: `app/reconciliation.py`

**Requirements**:
Implement all 8 validation checks from `reconciliation_logic.md`:

1. **Debit/Credit Balance Verification** (Critical)
   - Group by Description and Date
   - Sum debits and credits
   - Flag if unbalanced

2. **Duplicate Transaction Detection** (Warning)
   - Check for exact duplicates based on: Date, Account Code, Description, Debit, Credit
   - Group duplicates together

3. **Missing Account Codes** (Critical)
   - Check for empty, null, or whitespace-only account codes

4. **Account Code Format Validation** (Warning)
   - Must be exactly 4 characters
   - Must be numeric only

5. **Date Validation** (Critical)
   - Must be valid calendar dates
   - Must be parseable by pandas

6. **Negative Value Detection** (Warning)
   - Check both Debit and Credit columns
   - Flag any negative values

7. **Zero Amount Validation** (Info)
   - Check if both Debit and Credit are zero

8. **Missing Description** (Info)
   - Check for empty or whitespace-only descriptions

**Implementation Pattern**:
```python
class ReconciliationEngine:
    def __init__(self, dataframe: pd.DataFrame):
        self.df = dataframe
        self.errors = []
    
    def run_all_checks(self) -> dict:
        """Run all validation checks and return results"""
        self.check_date_validation()
        self.check_account_codes()
        self.check_negative_values()
        self.check_balance_verification()
        self.check_duplicates()
        # ... etc
        
        return self.generate_report()
    
    def check_balance_verification(self):
        """Check if debits equal credits per transaction group"""
        # Implementation here
        pass
    
    # ... implement other check methods
```

#### File: `app/report_generator.py`

**Requirements**:
- Generate Excel file with validation results
  - Add columns: `Validation Status`, `Error Code(s)`, `Error Message(s)`
  - Apply conditional formatting (red for errors, green for pass, yellow for warnings)
  
- Generate PDF error report with:
  - Executive summary (total transactions, error counts by severity)
  - Detailed error breakdown by type
  - Transaction-level details table
  - Professional formatting with headers/footers
  - Include timestamp and report metadata

### 2. Frontend Implementation

#### File: `templates/index.html`

**Requirements**:
- Professional, modern design using Bootstrap 5 or Tailwind CSS
- File upload interface with drag-and-drop support
- File validation (accept only .xlsx files)
- Upload progress indicator
- Clear instructions for users
- Responsive design (mobile-friendly)

**Key Elements**:
```html
<!-- Required sections -->
- Header with application title and logo
- Upload area (drag-drop + click to browse)
- File requirements note (Excel .xlsx, max 50MB)
- Sample file download link
- Upload button (disabled until file selected)
- Progress spinner during upload
- Error message display area
```

#### File: `templates/results.html`

**Requirements**:
- Display uploaded data in an interactive table
- Show validation results with color coding
- Display error summary statistics
- Provide download buttons for:
  - Validated Excel file
  - PDF error report
- Show error breakdown by severity (Critical, Warning, Info)
- Include "Upload Another File" button

**Key Features**:
- Use DataTables.js for sortable, searchable table
- Color-code rows: red (errors), green (pass), yellow (warnings)
- Show error codes and messages inline
- Display summary cards with counts
- Responsive layout

#### File: `static/js/main.js`

**Requirements**:
- Handle file upload with FormData API
- Show upload progress
- Handle success/error responses
- Implement drag-and-drop file upload
- Fetch and display results dynamically
- Handle download button clicks
- Form validation before submission

### 3. Docker Implementation

#### File: `Dockerfile`

**Requirements**:
```dockerfile
# Use official Python slim image
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements and install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create necessary directories
RUN mkdir -p uploads outputs

# Expose port
EXPOSE 8000

# Run application
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

#### File: `docker-compose.yml`

**Requirements**:
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - ./uploads:/app/uploads
      - ./outputs:/app/outputs
    environment:
      - ENVIRONMENT=production
    restart: unless-stopped
```

### 4. Configuration and Dependencies

#### File: `requirements.txt`

**Required packages**:
```
fastapi>=0.104.0
uvicorn[standard]>=0.24.0
python-multipart>=0.0.6
pandas>=2.1.0
openpyxl>=3.1.0
pydantic>=2.4.0
pydantic-settings>=2.0.3
python-dateutil>=2.8.2
reportlab>=4.0.0
jinja2>=3.1.2
aiofiles>=23.2.1
pytest>=7.4.0
httpx>=0.25.0
```

#### File: `.gitignore`

```
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
ENV/
.venv

# IDEs
.vscode/
.idea/
*.swp
*.swo

# Application
uploads/
outputs/
*.log

# OS
.DS_Store
Thumbs.db

# Testing
.pytest_cache/
.coverage
htmlcov/

# Docker
.dockerignore
```

---

## Implementation Steps (Recommended Order)

### Phase 1: Backend Foundation (Day 1)
1. Set up project structure
2. Create `requirements.txt` with all dependencies
3. Implement `app/main.py` with basic FastAPI setup
4. Implement `app/models.py` with Pydantic models
5. Test basic server startup

### Phase 2: Core Reconciliation Logic (Day 1-2)
1. Implement `app/reconciliation.py` with all 8 validation checks
2. Create unit tests in `tests/test_reconciliation.py`
3. Test with provided `sample_accounting_data.xlsx`
4. Verify all intentional errors are caught

### Phase 3: Report Generation (Day 2)
1. Implement Excel output with validation columns
2. Implement PDF report generation
3. Add conditional formatting to Excel
4. Test report outputs

### Phase 4: API Endpoints (Day 2-3)
1. Implement file upload endpoint
2. Implement processing endpoint
3. Implement download endpoints
4. Add error handling and validation
5. Test all endpoints with Postman or curl

### Phase 5: Frontend Development (Day 3)
1. Create `templates/index.html` (upload page)
2. Create `templates/results.html` (results page)
3. Implement `static/js/main.js` for interactivity
4. Style with `static/css/style.css`
5. Test user workflows end-to-end

### Phase 6: Docker & Deployment (Day 4)
1. Create `Dockerfile`
2. Create `docker-compose.yml`
3. Test Docker build and run
4. Create comprehensive `README.md`
5. Add API documentation

### Phase 7: Testing & Polish (Day 4)
1. Write comprehensive tests
2. Test with multiple Excel files
3. Verify error handling
4. Check responsive design
5. Performance testing (1000+ rows)

---

## Using Context7 for Documentation

**IMPORTANT**: Before you start coding, use Context7 to research and understand:

1. **FastAPI Best Practices**
   - Search for: "FastAPI file upload handling best practices"
   - Search for: "FastAPI serving static files and templates"
   - Search for: "FastAPI error handling patterns"

2. **Pandas Data Validation**
   - Search for: "Pandas data validation techniques"
   - Search for: "Pandas groupby and aggregation for accounting"
   - Search for: "Pandas duplicate detection methods"

3. **Excel Manipulation**
   - Search for: "OpenPyXL conditional formatting Python"
   - Search for: "Pandas to Excel with formatting"

4. **PDF Generation**
   - Search for: "ReportLab PDF table generation Python"
   - Search for: "Python PDF report generation best practices"

5. **Docker for Python Apps**
   - Search for: "Docker best practices for FastAPI applications"
   - Search for: "Python Docker multi-stage builds"

**How to use Context7**:
- Use it BEFORE implementing each major component
- Look for recent examples (2023-2024)
- Focus on production-ready patterns, not just tutorials
- Check for security best practices

---

## Testing Requirements

### Unit Tests

Create `tests/test_reconciliation.py`:

```python
import pytest
import pandas as pd
from app.reconciliation import ReconciliationEngine

def test_balance_verification():
    """Test that unbalanced entries are detected"""
    # Create test data with unbalanced entry
    data = {
        'Date': ['2024-10-01', '2024-10-01'],
        'Account Code': ['1000', '5000'],
        'Description': ['Test Entry', 'Test Entry'],
        'Debit': [100.0, 0.0],
        'Credit': [0.0, 90.0]  # Unbalanced!
    }
    df = pd.DataFrame(data)
    engine = ReconciliationEngine(df)
    results = engine.run_all_checks()
    assert len(results['errors']) > 0
    assert any('BAL-001' in error['code'] for error in results['errors'])

def test_duplicate_detection():
    """Test that duplicate transactions are detected"""
    # Implement test
    pass

def test_missing_account_code():
    """Test that missing account codes are flagged"""
    # Implement test
    pass

# ... implement tests for all 8 validation checks
```

### Integration Tests

Create `tests/test_api.py`:

```python
import pytest
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_upload_valid_file():
    """Test uploading a valid Excel file"""
    with open('sample_accounting_data.xlsx', 'rb') as f:
        response = client.post(
            '/upload',
            files={'file': ('test.xlsx', f, 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet')}
        )
    assert response.status_code == 200
    assert 'session_id' in response.json()

def test_upload_invalid_file():
    """Test uploading invalid file type"""
    # Implement test
    pass

# ... more integration tests
```

### Manual Testing Checklist

- [ ] Upload sample_accounting_data.xlsx
- [ ] Verify all 6 types of errors are detected
- [ ] Check error counts match expectations
- [ ] Download validated Excel file
- [ ] Download PDF report
- [ ] Verify Excel conditional formatting
- [ ] Test with large file (1000+ rows)
- [ ] Test with invalid file format
- [ ] Test with empty file
- [ ] Test error handling for corrupted file
- [ ] Test responsive design on mobile
- [ ] Test Docker deployment
- [ ] Verify all dependencies install correctly

---

## README.md Requirements

Your `README.md` must include:

### 1. Project Title and Description
- Clear, concise project description
- Key features list
- Screenshot or demo GIF

### 2. Features
- List all features with checkboxes
- Highlight what makes it special

### 3. Technology Stack
- List all technologies used
- Explain why each was chosen

### 4. Installation Instructions

**Local Development**:
```bash
# Clone repository
git clone <repo-url>
cd accounting-reconciliation-app

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run application
uvicorn app.main:app --reload

# Access at http://localhost:8000
```

**Docker Deployment**:
```bash
# Build and run with Docker Compose
docker-compose up --build

# Access at http://localhost:8000
```

### 5. Usage Guide
- Step-by-step instructions with screenshots
- Example workflows
- Expected outputs

### 6. API Documentation
- List all endpoints
- Example requests and responses
- Link to OpenAPI docs (FastAPI auto-generated)

### 7. Project Structure
- Explain directory organization
- Describe each major file's purpose

### 8. Testing
- How to run tests
- Coverage information

### 9. Contributing
- Guidelines for contributions
- Code style requirements

### 10. License
- Specify license type (MIT recommended)

### 11. Acknowledgments
- Credit any resources used
- Mention demo case origin

---

## Acceptance Criteria

The project is complete when:

### Functionality
- ✅ All 8 validation checks are implemented and working
- ✅ Sample data with all 6 error types is correctly identified
- ✅ Excel output includes validation results with proper formatting
- ✅ PDF report is comprehensive and professional
- ✅ File upload works with drag-and-drop
- ✅ Results display is clear and interactive
- ✅ Downloads work for both Excel and PDF

### Code Quality
- ✅ Code follows PEP 8 style guidelines
- ✅ Functions have docstrings
- ✅ Complex logic is commented
- ✅ No hardcoded values (use configuration)
- ✅ Proper error handling throughout
- ✅ Logging is implemented

### Testing
- ✅ Unit tests cover all reconciliation logic
- ✅ Integration tests cover API endpoints
- ✅ All tests pass
- ✅ Test coverage > 80%

### Documentation
- ✅ README.md is comprehensive
- ✅ Code comments explain complex logic
- ✅ API documentation is clear
- ✅ Setup instructions work on fresh system

### Deployment
- ✅ Dockerfile builds successfully
- ✅ Docker Compose works
- ✅ Application runs in container
- ✅ Can access application on localhost:8000

### User Experience
- ✅ Interface is intuitive
- ✅ Error messages are clear
- ✅ Loading states are shown
- ✅ Responsive design works on mobile
- ✅ Professional appearance

### Performance
- ✅ Processes 1000+ transactions in < 5 seconds
- ✅ File upload handles 50MB files
- ✅ No memory leaks with multiple uploads

---

## Security Considerations

Implement these security measures:

1. **File Upload Security**
   - Validate file extensions
   - Check file size limits
   - Scan for malicious content if possible
   - Store uploaded files temporarily with unique names

2. **Input Validation**
   - Validate all user inputs
   - Use Pydantic models for data validation
   - Sanitize file names

3. **Error Handling**
   - Don't expose internal paths in error messages
   - Log errors securely
   - Return generic error messages to users

4. **CORS Configuration**
   - Configure CORS properly for production
   - Restrict allowed origins

5. **Rate Limiting**
   - Consider adding rate limiting for uploads
   - Prevent abuse

---

## Best Practices to Follow

### Python Code
- Use type hints everywhere
- Follow PEP 8
- Keep functions small and focused
- Use meaningful variable names
- Avoid magic numbers

### FastAPI
- Use dependency injection
- Leverage Pydantic models
- Use async/await appropriately
- Document endpoints with docstrings
- Use status codes correctly

### Frontend
- Separate concerns (HTML/CSS/JS)
- Use semantic HTML
- Make it accessible (ARIA labels, keyboard navigation)
- Progressive enhancement
- Mobile-first design

### Git
- Write clear commit messages
- Use conventional commits
- Keep commits atomic
- Don't commit sensitive data
- Use .gitignore properly

---

## Common Pitfalls to Avoid

1. **Don't** hardcode file paths - use Path from pathlib
2. **Don't** ignore error cases - handle all exceptions
3. **Don't** use global variables - pass dependencies
4. **Don't** skip input validation - always validate
5. **Don't** forget to clean up temporary files
6. **Don't** use print() for logging - use logging module
7. **Don't** commit node_modules, __pycache__, etc.
8. **Don't** skip writing tests - they save time later

---

## Helpful Commands Reference

### Development
```bash
# Run with auto-reload
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Run tests
pytest tests/ -v

# Run tests with coverage
pytest tests/ --cov=app --cov-report=html

# Format code
black app/ tests/

# Lint code
flake8 app/ tests/

# Type checking
mypy app/
```

### Docker
```bash
# Build image
docker build -t accounting-reconciliation-app .

# Run container
docker run -p 8000:8000 accounting-reconciliation-app

# Docker Compose
docker-compose up --build
docker-compose down
docker-compose logs -f
```

### Git
```bash
# Initialize repository
git init
git add .
git commit -m "Initial commit"

# Create .gitignore
curl -o .gitignore https://raw.githubusercontent.com/github/gitignore/main/Python.gitignore
```

---

## Final Deliverables Checklist

Before considering the project complete:

- [ ] All code is written and tested
- [ ] All 8 validation checks work correctly
- [ ] Frontend is polished and responsive
- [ ] Docker setup works
- [ ] README.md is comprehensive
- [ ] Tests are written and passing
- [ ] API documentation is complete
- [ ] Sample data processing works end-to-end
- [ ] Error handling is robust
- [ ] Code is well-commented
- [ ] Project is ready for GitHub
- [ ] All acceptance criteria are met

---

## Portfolio Presentation Tips

When showcasing this project:

1. **Start with a demo**
   - Show file upload
   - Process sample data
   - Display results
   - Download reports

2. **Highlight technical decisions**
   - Why FastAPI?
   - Why these validation rules?
   - How you handled errors

3. **Show the code**
   - Walk through key functions
   - Explain design patterns
   - Discuss trade-offs

4. **Demonstrate testing**
   - Run test suite
   - Show coverage report
   - Explain test strategy

5. **Deploy live**
   - Show Docker deployment
   - Demonstrate production readiness
   - Discuss scalability

---

## Additional Resources

### Documentation to Reference
- FastAPI: https://fastapi.tiangolo.com/
- Pandas: https://pandas.pydata.org/docs/
- OpenPyXL: https://openpyxl.readthedocs.io/
- ReportLab: https://www.reportlab.com/docs/
- Docker: https://docs.docker.com/

### Example Projects to Study
- Search GitHub for: "FastAPI file upload example"
- Search GitHub for: "Pandas data validation"
- Search GitHub for: "Excel processing web app"

### Learning Resources
- FastAPI Tutorial: https://fastapi.tiangolo.com/tutorial/
- Real Python (Pandas): https://realpython.com/pandas-python-explore-dataset/
- Docker for Python Developers: https://docs.docker.com/language/python/

---

## Success Metrics

This project will be considered successful if:

1. **Functional**: All features work as specified
2. **Quality**: Code is clean, tested, and documented
3. **Professional**: UI/UX is polished and intuitive
4. **Deployable**: Can be deployed with Docker easily
5. **Portfolio-Ready**: Demonstrates skills to potential employers

---

## Questions to Ask Yourself During Development

- Does this code follow best practices?
- Is this function doing too much?
- Have I handled all error cases?
- Is this user-friendly?
- Would I be proud to show this to an employer?
- Is this code testable?
- Is this documentation clear?

---

**Remember**: This is a portfolio project. Quality over speed. Take time to:
- Write clean code
- Add proper error handling
- Write comprehensive tests
- Create great documentation
- Polish the UI/UX

Good luck building! 🚀

---

**Version**: 1.0  
**Last Updated**: October 2024  
**Estimated Development Time**: 3-5 days for complete implementation
