# Accounting Reconciliation Logic Specification

## Overview

This document defines the complete reconciliation logic that the accounting reconciliation application must implement. The system should perform automated validation checks on uploaded accounting data and generate comprehensive error reports.

---

## Input Data Format

### Expected Excel Structure

| Column Name    | Data Type | Description                           | Required |
|----------------|-----------|---------------------------------------|----------|
| Transaction ID | Integer   | Unique transaction identifier         | Yes      |
| Date           | Date      | Transaction date (YYYY-MM-DD format)  | Yes      |
| Account Code   | String    | 4-digit numeric account code          | Yes      |
| Description    | String    | Transaction description               | Yes      |
| Debit          | Float     | Debit amount (positive values only)   | Yes      |
| Credit         | Float     | Credit amount (positive values only)  | Yes      |

---

## Reconciliation Checks

### 1. Debit/Credit Balance Verification

**Purpose**: Ensure that for each unique transaction group, total debits equal total credits.

**Logic**:
- Group transactions by `Description` and `Date`
- Sum all `Debit` amounts in the group
- Sum all `Credit` amounts in the group
- Compare totals: If `Σ Debits ≠ Σ Credits`, flag as error

**Error Details**:
- **Severity**: `Critical`
- **Error Code**: `BAL-001`
- **Message**: "Unbalanced entry: Debits ({debit_sum}) do not equal Credits ({credit_sum})"
- **Affected Transactions**: List all Transaction IDs in the unbalanced group

**Example**:
```
Transaction 27: Debit = 500.00
Transaction 28: Credit = 450.00
ERROR: Unbalanced by 50.00
```

---

### 2. Duplicate Transaction Detection

**Purpose**: Identify exact duplicate entries that may indicate data entry errors.

**Logic**:
- Compare all rows based on: `Date`, `Account Code`, `Description`, `Debit`, `Credit`
- If two or more rows have identical values for ALL these fields, flag as duplicate
- Group duplicates together and report all instances

**Error Details**:
- **Severity**: `Warning`
- **Error Code**: `DUP-001`
- **Message**: "Duplicate transaction detected ({count} instances)"
- **Affected Transactions**: List all duplicate Transaction IDs

**Example**:
```
Transactions 29 and 31: Identical entries
Date: 2024-10-11, Account: 5300, Description: "Internet Service - Comcast"
```

---

### 3. Missing Account Codes

**Purpose**: Ensure all transactions have valid account codes for proper categorization.

**Logic**:
- Check if `Account Code` field is empty, null, or contains only whitespace
- Flag any transaction with missing account code

**Error Details**:
- **Severity**: `Critical`
- **Error Code**: `ACC-001`
- **Message**: "Missing account code"
- **Affected Transactions**: Transaction ID with missing code

**Example**:
```
Transaction 33: Account Code is empty
```

---

### 4. Account Code Format Validation

**Purpose**: Ensure account codes follow the standard 4-digit numeric format.

**Logic**:
- Account codes must be exactly 4 characters long
- Account codes must contain only numeric digits (0-9)
- No letters, special characters, or spaces allowed

**Error Details**:
- **Severity**: `Warning`
- **Error Code**: `ACC-002`
- **Message**: "Invalid account code format (expected 4-digit numeric)"
- **Affected Transactions**: Transaction ID with invalid format

**Valid Examples**: `1000`, `5300`, `4100`
**Invalid Examples**: `ABC123`, `10`, `1000A`, `10 00`

---

### 5. Date Validation

**Purpose**: Ensure all transaction dates are valid and properly formatted.

**Logic**:
- Dates must be in valid calendar format
- Dates must be parseable (valid day, month, year combinations)
- Optionally check if dates are not in the future (configurable)

**Error Details**:
- **Severity**: `Critical`
- **Error Code**: `DATE-001`
- **Message**: "Invalid or unparseable date format"
- **Affected Transactions**: Transaction ID with invalid date

**Invalid Examples**: 
- `2024-13-45` (invalid month/day)
- `2024-02-30` (February doesn't have 30 days)
- `NotADate` (unparseable)

---

### 6. Negative Value Detection

**Purpose**: Ensure debit and credit amounts are positive values (negative amounts in wrong columns indicate errors).

**Logic**:
- Check if `Debit` column contains any negative values
- Check if `Credit` column contains any negative values
- Both columns should only contain zero or positive amounts

**Error Details**:
- **Severity**: `Warning`
- **Error Code**: `VAL-001`
- **Message**: "Negative value detected in {column_name} column"
- **Affected Transactions**: Transaction ID with negative value

**Note**: In proper double-entry bookkeeping, negative amounts should be represented by swapping debit/credit, not using negative numbers.

---

### 7. Zero Amount Validation

**Purpose**: Identify transactions where both debit and credit are zero (likely data errors).

**Logic**:
- Check if both `Debit` and `Credit` are zero
- Flag as potential data entry error

**Error Details**:
- **Severity**: `Info`
- **Error Code**: `VAL-002`
- **Message**: "Both debit and credit are zero"
- **Affected Transactions**: Transaction ID with zero amounts

---

### 8. Missing Description

**Purpose**: Ensure all transactions have meaningful descriptions for audit purposes.

**Logic**:
- Check if `Description` field is empty, null, or contains only whitespace
- Flag any transaction without proper description

**Error Details**:
- **Severity**: `Info`
- **Error Code**: `DESC-001`
- **Message**: "Missing transaction description"
- **Affected Transactions**: Transaction ID with missing description

---

## Error Severity Levels

### Critical
- **Impact**: Prevents accurate financial reporting
- **Action Required**: Must be fixed before reconciliation can be considered complete
- **Examples**: Unbalanced entries, missing account codes, invalid dates

### Warning
- **Impact**: May indicate errors but doesn't prevent basic reconciliation
- **Action Required**: Should be reviewed and corrected
- **Examples**: Duplicate transactions, invalid account code formats, negative values

### Info
- **Impact**: Minor issues or informational notices
- **Action Required**: Optional review
- **Examples**: Zero amounts, missing descriptions

---

## Expected Outputs

### 1. Validated Data Excel File

**Filename**: `reconciliation_results_{timestamp}.xlsx`

**Structure**:
- All original columns from input file
- Additional column: `Validation Status` (✓ Pass / ✗ Fail)
- Additional column: `Error Code(s)` (comma-separated if multiple)
- Additional column: `Error Message(s)` (detailed description)

**Conditional Formatting**:
- Rows with errors highlighted in red
- Rows that pass all checks highlighted in green
- Warning-level errors highlighted in yellow

---

### 2. Error Report

**Format**: Both Excel and PDF

**Content**:

#### Executive Summary
- Total transactions processed
- Total errors found (by severity)
- Overall reconciliation status (Pass/Fail)
- Summary statistics table

#### Detailed Error Breakdown
For each error type:
- Error code and description
- Count of occurrences
- List of affected transaction IDs
- Severity level
- Recommended corrective actions

#### Transaction-Level Details
Table with:
- Transaction ID
- Error Code
- Severity
- Error Message
- Original Values
- Suggested Correction (where applicable)

---

## Reconciliation Rules Summary

| Check                    | Error Code | Severity | Description                                    |
|--------------------------|------------|----------|------------------------------------------------|
| Balance Verification     | BAL-001    | Critical | Debits must equal credits per transaction      |
| Duplicate Detection      | DUP-001    | Warning  | Exact duplicate entries detected               |
| Missing Account Code     | ACC-001    | Critical | Account code is empty or missing               |
| Account Code Format      | ACC-002    | Warning  | Account code must be 4-digit numeric           |
| Date Validation          | DATE-001   | Critical | Date must be valid calendar date               |
| Negative Values          | VAL-001    | Warning  | Debit/Credit amounts must be positive          |
| Zero Amount              | VAL-002    | Info     | Both debit and credit are zero                 |
| Missing Description      | DESC-001   | Info     | Transaction description is missing             |

---

## Processing Workflow

```
1. Upload Excel File
   ↓
2. Parse and Load Data
   ↓
3. Run Validation Checks (in order)
   - Date Validation
   - Account Code Validation
   - Missing Field Checks
   - Value Validation
   - Balance Verification
   - Duplicate Detection
   ↓
4. Generate Error Report
   ↓
5. Create Validated Excel Output
   ↓
6. Display Results to User
   ↓
7. Provide Download Options
   - Validated Excel file
   - PDF Error Report
```

---

## Testing Requirements

### Test Data Should Include:
1. ✓ Clean, balanced transactions (majority)
2. ✗ Unbalanced entries (debits ≠ credits)
3. ✗ Exact duplicate transactions
4. ✗ Missing account codes
5. ✗ Invalid account code formats
6. ✗ Invalid dates
7. ✗ Negative values
8. ℹ Zero amount entries
9. ℹ Missing descriptions

### Expected Behavior:
- Application should catch ALL intentional errors
- Error report should clearly identify each issue
- No false positives on valid entries
- Performance: Process 1000+ transactions in < 5 seconds

---

## Additional Features (Nice-to-Have)

### Auto-Correction Suggestions
For certain error types, suggest corrections:
- Duplicate transactions → "Consider removing duplicate entries"
- Invalid account codes → "Did you mean: 1000?" (fuzzy matching)
- Date typos → Suggest valid date if pattern recognizable

### Configurable Rules
Allow users to configure:
- Account code format (4-digit vs other patterns)
- Date range validation
- Custom severity levels
- Which checks to enable/disable

### Batch Processing
- Upload multiple files at once
- Compare files for reconciliation between periods
- Aggregate reports across multiple files

---

## Implementation Notes

### Data Handling
- Preserve original data integrity (never modify input file)
- Handle large files efficiently (chunk processing for 10,000+ rows)
- Support Excel files up to 50MB

### Error Messaging
- Clear, non-technical language for end users
- Specific enough to identify exact issue
- Include transaction IDs for easy reference

### User Experience
- Real-time progress indicators during processing
- Preview of data before processing
- Ability to download results immediately
- Clear navigation between upload, results, and reports

---

## Success Criteria

The reconciliation application is considered successful when it:

1. ✓ Correctly identifies all 6 types of intentional errors in sample data
2. ✓ Processes files with 40+ transactions in under 3 seconds
3. ✓ Generates properly formatted Excel and PDF reports
4. ✓ Has zero false positives on valid transactions
5. ✓ Provides clear, actionable error messages
6. ✓ Has an intuitive, professional-looking interface
7. ✓ Can be deployed via Docker with a single command
8. ✓ Includes comprehensive documentation

---

**Document Version**: 1.0  
**Last Updated**: October 2024  
**Author**: Demo Case Specification  
