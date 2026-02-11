# Project Context Document (PCD)
## OFX Reader for Brazilian Banks

---

## 1. Project Overview

**Project Type:** A (Single-File HTML App)

**Summary:**  
A self-contained HTML web application that reads and displays OFX (Open Financial Exchange) bank statement files from Brazilian banks. The application handles encoding issues common in Brazilian bank exports where files incorrectly declare USASCII encoding but contain Portuguese text encoded in Windows-1252 or similar.

**Users:**  
Anyone who needs to view Brazilian bank OFX statements in a readable format. No technical knowledge required.

**Core Value:**  
Transform cryptic OFX files with encoding issues into a clean, professional dark-themed interface showing account metadata and transaction history.

---

## 2. Requirements

### 2.1 Must-Have Features

1. **Single self-contained HTML file** - entire app in one file with inline CSS and JavaScript
2. **File loading interface:**
   - Drop zone area for drag-and-drop file upload
   - Click to open file picker dialog
   - Accept `.ofx` files only
3. **Encoding handling:**
   - Treat all file content as Portuguese text regardless of declared encoding
   - Properly decode Windows-1252 characters (ç, á, ã, é, í, ó, ú, â, ê, ô, à)
4. **Metadata display:**
   - Display as cards above the transaction table
   - Show: Bank Name, Bank ID, Account ID, Branch ID (if available), Account Type, Currency, Date Range, Balance (if available)
5. **Transaction table:**
   - Columns: Date (DD-MM-YYYY format), Amount, Description
   - One transaction per row
   - No text wrapping in columns
   - Debit amounts in red color
   - Credit amounts in green color
6. **Visual design:**
   - Dark blue theme
   - Ultra-professional appearance
   - Clean typography
   - Good contrast for readability
7. **UI Header:**
   - Application title/branding
   - Professional appearance

### 2.2 Explicitly NOT in Scope

- Multiple file loading
- Filtering transactions
- Searching transactions
- Sorting transactions (display in original file order)
- Editing transactions
- Exporting to other formats
- User accounts or authentication
- Data persistence (no localStorage)
- Transaction categorization
- Charts or graphs

---

## 3. Technology Decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| **Architecture** | Single HTML file | User requirement: self-contained, portable, offline-capable |
| **JavaScript** | Vanilla ES6+ | No dependencies needed, full browser support |
| **CSS** | Inline in `<style>` tag | Keep everything in one file |
| **File Reading** | FileReader API | Native browser API, no dependencies |
| **Parsing** | Custom regex-based parser | OFX is SGML-like with inconsistent structure, regex most reliable |
| **Encoding** | Force UTF-8 read with Windows-1252 character mapping | Handle declared encoding mismatches |
| **Date Handling** | Custom parsing function | OFX dates vary: YYYYMMDD or YYYYMMDDHHMMSS[TZ] |

**Rationale for Custom Parser:**  
OFX files use SGML format (not XML), with optional closing tags and inconsistent structure across Brazilian banks. A regex-based parser provides most reliable extraction.

---

## 4. Data Structures

### 4.1 OFXData Schema

```json
{
  "metadata": {
    "bankName": "string | null",
    "bankId": "string | null", 
    "accountId": "string | null",
    "branchId": "string | null",
    "accountType": "string | null",
    "currency": "string | null",
    "startDate": "string (DD-MM-YYYY) | null",
    "endDate": "string (DD-MM-YYYY) | null",
    "balance": "number | null",
    "balanceDate": "string (DD-MM-YYYY) | null"
  },
  "transactions": [
    {
      "type": "string (CREDIT|DEBIT|PAYMENT|OTHER)",
      "date": "string (DD-MM-YYYY)",
      "amount": "number",
      "description": "string"
    }
  ]
}
```

**Field Definitions:**

- `metadata.bankName`: From `<ORG>` or `<FI><ORG>` tag. Default: "Unknown Bank"
- `metadata.bankId`: From `<BANKID>` or `<FID>` tag. Default: null
- `metadata.accountId`: From `<ACCTID>` tag. Default: null
- `metadata.branchId`: From `<BRANCHID>` tag (optional). Default: null
- `metadata.accountType`: From `<ACCTTYPE>` tag. Default: null
- `metadata.currency`: From `<CURDEF>` tag. Default: "BRL"
- `metadata.startDate`: From `<DTSTART>` tag, converted to DD-MM-YYYY. Default: null
- `metadata.endDate`: From `<DTEND>` tag, converted to DD-MM-YYYY. Default: null
- `metadata.balance`: From `<LEDGERBAL><BALAMT>` tag, parsed as float. Default: null
- `metadata.balanceDate`: From `<LEDGERBAL><DTASOF>` tag, converted to DD-MM-YYYY. Default: null
- `transactions[].type`: From `<TRNTYPE>`. Maps to display label ("Crédito", "Débito", "Pagamento")
- `transactions[].date`: From `<DTPOSTED>`, converted to DD-MM-YYYY. Required
- `transactions[].amount`: From `<TRNAMT>`, parsed as float. Required
- `transactions[].description`: From `<MEMO>` or `<NAME>` tag. Prefer `<MEMO>`. Default: "No description"

**Null/Missing Handling:**

- Missing metadata fields → display "N/A" in UI
- Missing transaction description → display "No description"
- Invalid date → skip transaction with console warning
- Invalid amount → skip transaction with console warning
- Empty file → show error message "No valid data found"
- Not an OFX file → show error message "Invalid OFX file"

---

## 5. Algorithms

### 5.1 parseOFXFile(fileContent: string) → OFXData

**Purpose:** Extract structured data from raw OFX file text.

**Pseudocode:**

```
FUNCTION parseOFXFile(fileContent):
    IF fileContent is null OR fileContent is empty:
        THROW Error "Empty file content"
    
    // Fix encoding issues
    content = fixPortugueseEncoding(fileContent)
    
    // Validate OFX header
    IF content does not contain "OFXHEADER" OR "OFX":
        THROW Error "Invalid OFX file"
    
    // Initialize result
    result = {
        metadata: {},
        transactions: []
    }
    
    // Extract metadata
    result.metadata.bankName = extractTag(content, "<ORG>") OR "Unknown Bank"
    result.metadata.bankId = extractTag(content, "<BANKID>") OR extractTag(content, "<FID>")
    result.metadata.accountId = extractTag(content, "<ACCTID>")
    result.metadata.branchId = extractTag(content, "<BRANCHID>")
    result.metadata.accountType = extractTag(content, "<ACCTTYPE>")
    result.metadata.currency = extractTag(content, "<CURDEF>") OR "BRL"
    
    startDateRaw = extractTag(content, "<DTSTART>")
    IF startDateRaw is not null:
        result.metadata.startDate = parseOFXDate(startDateRaw)
    
    endDateRaw = extractTag(content, "<DTEND>")
    IF endDateRaw is not null:
        result.metadata.endDate = parseOFXDate(endDateRaw)
    
    balanceRaw = extractTag(content, "<BALAMT>")
    IF balanceRaw is not null:
        result.metadata.balance = parseFloat(balanceRaw)
    
    balanceDateRaw = extractTag(content, "<DTASOF>")
    IF balanceDateRaw is not null:
        result.metadata.balanceDate = parseOFXDate(balanceDateRaw)
    
    // Extract transactions
    transactionBlocks = extractAllBetween(content, "<STMTTRN>", "</STMTTRN>")
    
    FOR EACH transactionBlock IN transactionBlocks:
        transaction = parseTransaction(transactionBlock)
        IF transaction is valid:
            result.transactions.push(transaction)
    
    IF result.transactions.length is 0:
        THROW Error "No valid transactions found"
    
    RETURN result
END FUNCTION
```

**Walk-through Test:**

**Input:** ITAU sample file (407 lines)
**Expected:**
- Metadata: bankName="Unknown Bank" (no ORG tag), bankId="0341", accountId="3839196312", currency="BRL", startDate="01-10-2025", endDate="31-10-2025", balance=25282.87
- Transactions: 75 transactions, first is CREDIT 1410.00 on "01-10-2025", last is DEBIT -1163.52 on "31-10-2025"

**Input:** Empty file
**Expected:** Error "Empty file content"

**Input:** Text file with no OFX header
**Expected:** Error "Invalid OFX file"

---

### 5.2 fixPortugueseEncoding(text: string) → string

**Purpose:** Convert incorrectly encoded Portuguese characters to proper UTF-8.

**Pseudocode:**

```
FUNCTION fixPortugueseEncoding(text):
    IF text is null OR text is empty:
        RETURN ""
    
    // Map common Windows-1252 byte sequences appearing in "ASCII" files
    replacements = {
        "�": "ã",  // Common corruption patterns
        "�": "ç",
        "�": "á",
        "�": "é",
        "�": "í",
        "�": "ó",
        "�": "ú",
        "�": "â",
        "�": "ê",
        "�": "ô",
        "�": "à",
        // Add more as discovered in testing
    }
    
    result = text
    FOR EACH (corruptedChar, correctChar) IN replacements:
        result = result.replaceAll(corruptedChar, correctChar)
    
    RETURN result
END FUNCTION
```

**Walk-through Test:**

**Input:** "D�B.CONV.TRIBUTOS FEDERAIS"
**Expected:** "DÉB.CONV.TRIBUTOS FEDERAIS"

**Input:** "ANTECIPA��O MASTERCARD"
**Expected:** "ANTECIPAÇÃO MASTERCARD"

---

### 5.3 parseOFXDate(dateStr: string) → string (DD-MM-YYYY)

**Purpose:** Convert OFX date formats to DD-MM-YYYY display format.

**Pseudocode:**

```
FUNCTION parseOFXDate(dateStr):
    IF dateStr is null OR dateStr is empty:
        RETURN null
    
    // Remove timezone and time portions
    // Formats seen: YYYYMMDD, YYYYMMDDHHMMSS, YYYYMMDDHHMMSS[TZ]
    cleanDate = dateStr split by "[" take first part
    cleanDate = cleanDate take first 8 characters
    
    IF cleanDate length is not 8:
        RETURN null
    
    year = cleanDate substring(0, 4)
    month = cleanDate substring(4, 6)
    day = cleanDate substring(6, 8)
    
    // Validate
    IF year is not numeric OR month is not numeric OR day is not numeric:
        RETURN null
    IF month < 1 OR month > 12:
        RETURN null
    IF day < 1 OR day > 31:
        RETURN null
    
    RETURN day + "-" + month + "-" + year
END FUNCTION
```

**Walk-through Test:**

**Input:** "20251001100000[-03:EST]"
**Expected:** "01-10-2025"

**Input:** "20260129"
**Expected:** "29-01-2026"

**Input:** "invalid"
**Expected:** null

---

### 5.4 parseTransaction(block: string) → Transaction | null

**Purpose:** Extract transaction data from STMTTRN block.

**Pseudocode:**

```
FUNCTION parseTransaction(block):
    IF block is null OR block is empty:
        RETURN null
    
    typeRaw = extractTag(block, "<TRNTYPE>")
    dateRaw = extractTag(block, "<DTPOSTED>")
    amountRaw = extractTag(block, "<TRNAMT>")
    memo = extractTag(block, "<MEMO>") OR extractTag(block, "<NAME>") OR "No description"
    
    // Validate required fields
    IF dateRaw is null OR amountRaw is null:
        console.warn("Skipping transaction: missing date or amount")
        RETURN null
    
    date = parseOFXDate(dateRaw)
    IF date is null:
        console.warn("Skipping transaction: invalid date format")
        RETURN null
    
    amount = parseFloat(amountRaw)
    IF amount is NaN:
        console.warn("Skipping transaction: invalid amount")
        RETURN null
    
    // Map transaction type
    IF typeRaw is null:
        type = "OTHER"
    ELSE IF typeRaw equals "CREDIT":
        type = "CREDIT"
    ELSE IF typeRaw equals "DEBIT":
        type = "DEBIT"
    ELSE IF typeRaw equals "PAYMENT":
        type = "PAYMENT"
    ELSE:
        type = "OTHER"
    
    RETURN {
        type: type,
        date: date,
        amount: amount,
        description: memo trim whitespace
    }
END FUNCTION
```

**Walk-through Test:**

**Input:** Block with CREDIT, date "20251001100000", amount "1410.00", memo "BOLETOS RECEBIDOS"
**Expected:** { type: "CREDIT", date: "01-10-2025", amount: 1410.00, description: "BOLETOS RECEBIDOS" }

**Input:** Block missing <TRNAMT>
**Expected:** null (with console warning)

---

### 5.5 formatCurrency(value: number) → string

**Purpose:** Format transaction amounts as Brazilian Real.

**Pseudocode:**

```
FUNCTION formatCurrency(value):
    IF value is null OR value is NaN:
        RETURN "R$ 0,00"
    
    // Format with 2 decimal places, comma for decimal, dot for thousands
    absValue = absolute value of value
    
    formatted = absValue.toFixed(2)  // "1234.56"
    formatted = formatted.replace(".", ",")  // "1234,56"
    
    // Add thousand separators
    parts = formatted.split(",")
    integerPart = parts[0]
    decimalPart = parts[1]
    
    integerWithSeparators = ""
    FOR i = integerPart.length - 1 down to 0 step -1:
        integerWithSeparators = integerPart[i] + integerWithSeparators
        position = integerPart.length - i
        IF position mod 3 equals 0 AND i > 0:
            integerWithSeparators = "." + integerWithSeparators
    
    RETURN "R$ " + integerWithSeparators + "," + decimalPart
END FUNCTION
```

**Walk-through Test:**

**Input:** 1410.00
**Expected:** "R$ 1.410,00"

**Input:** -856.47
**Expected:** "R$ 856,47"

**Input:** 25282.87
**Expected:** "R$ 25.282,87"

---

### 5.6 handleFileUpload(file: File) → void

**Purpose:** Process uploaded OFX file and update UI.

**Pseudocode:**

```
FUNCTION handleFileUpload(file):
    IF file is null:
        showError("No file selected")
        RETURN
    
    IF file name does not end with ".ofx":
        showError("Please select an OFX file")
        RETURN
    
    showLoadingState()
    
    TRY:
        fileContent = readFileAsText(file)
        data = parseOFXFile(fileContent)
        renderUI(data)
        hideError()
    CATCH error:
        showError("Error reading file: " + error.message)
        hideLoadingState()
END FUNCTION
```

**Walk-through Test:**

**Input:** Valid OFX file
**Expected:** UI updates with metadata cards and transaction table

**Input:** .txt file
**Expected:** Error message "Please select an OFX file"

**Input:** Corrupted OFX file
**Expected:** Error message with description

---

## 6. UI Specification

### 6.1 Component Inventory

| Component | Purpose | Data Displayed | User Actions |
|-----------|---------|----------------|--------------|
| **App Header** | Branding and title | "OFX Reader" title | None |
| **Upload Zone** | File input area | Drop zone text, file picker button | Drag-drop file, click to browse |
| **Metadata Cards** | Account information | Bank, account, date range, balance | None (read-only) |
| **Transaction Table** | List of transactions | Date, Amount, Description columns | Scroll |
| **Error Message** | Error feedback | Error text | None (dismisses on new upload) |
| **Loading State** | Processing indicator | "Loading..." text | None |

### 6.2 Page Layout

```
┌─────────────────────────────────────────────────┐
│ [OFX Reader Header with logo/title]             │
├─────────────────────────────────────────────────┤
│                                                 │
│  [Upload Zone - drag/drop or click]            │
│                                                 │
├─────────────────────────────────────────────────┤
│ [Metadata Cards Row - horizontal grid]          │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐           │
│  │ Bank │ │ Acct │ │ Dates│ │ Bal. │           │
│  └──────┘ └──────┘ └──────┘ └──────┘           │
├─────────────────────────────────────────────────┤
│ [Transaction Table - full width]                │
│ ┌──────┬───────────┬─────────────────────────┐  │
│ │ Date │  Amount   │     Description        │  │
│ ├──────┼───────────┼─────────────────────────┤  │
│ │ ...  │  ...      │     ...                │  │
│ └──────┴───────────┴─────────────────────────┘  │
│                                                 │
└─────────────────────────────────────────────────┘
```

### 6.3 Application States

**State 1: Initial (No File Loaded)**
- Upload Zone: Visible, prominent
- Metadata Cards: Hidden
- Transaction Table: Hidden
- Error Message: Hidden

**State 2: Loading**
- Upload Zone: Visible, disabled
- Loading indicator: Visible
- Metadata Cards: Hidden
- Transaction Table: Hidden
- Error Message: Hidden

**State 3: Success (File Loaded)**
- Upload Zone: Visible, smaller, at top
- Metadata Cards: Visible with populated data
- Transaction Table: Visible with all transactions
- Error Message: Hidden
- Can upload new file to replace current

**State 4: Error**
- Upload Zone: Visible
- Error Message: Visible with error text
- Metadata Cards: Hidden
- Transaction Table: Hidden

### 6.4 Visual Design Specification

**Color Palette (Dark Blue Theme):**
- Primary Background: `#0a1929` (very dark blue)
- Secondary Background: `#132f4c` (dark blue for cards)
- Surface: `#1e3a5f` (medium blue for table rows)
- Primary Text: `#e3f2fd` (light blue/white)
- Secondary Text: `#90caf9` (lighter blue)
- Credit Color: `#4caf50` (green)
- Debit Color: `#f44336` (red)
- Border Color: `#2c5282` (medium blue)
- Accent: `#1976d2` (bright blue)

**Typography:**
- Font Family: `'Segoe UI', Tahoma, Geneva, Verdana, sans-serif`
- Header: 32px, bold
- Card Title: 12px, uppercase, secondary text color
- Card Value: 18px, bold, primary text color
- Table Header: 14px, bold, uppercase
- Table Cell: 14px, regular
- No wrapping: `white-space: nowrap; overflow: hidden; text-overflow: ellipsis`

**Spacing:**
- Container padding: 24px
- Card spacing: 16px gap
- Table cell padding: 12px vertical, 16px horizontal
- Section margins: 32px

**Interactive Elements:**
- Upload Zone: Border-dashed, hover state brightens border
- Table rows: Hover state slightly brightens background

### 6.5 User Flow

```
START
  ↓
[User opens HTML file in browser]
  ↓
[Sees Upload Zone with instructions]
  ↓
User Action: Drags OFX file OR clicks to browse
  ↓
[File selected]
  ↓
[App shows loading state]
  ↓
[App reads and parses file]
  ↓
SUCCESS?
├─ YES → [Display metadata cards]
│         ↓
│        [Display transaction table]
│         ↓
│        [User scrolls to view transactions]
│         ↓
│        User can upload new file → [Return to file selection]
│
└─ NO → [Display error message]
        ↓
       [User can try again with different file]
```

---

## 7. Edge Cases

| Situation | Expected Behavior | User-Facing Message |
|-----------|-------------------|---------------------|
| Empty file | Show error | "Empty file. Please select a valid OFX file." |
| Not OFX file | Show error | "Invalid file format. Please select an OFX file." |
| Corrupted OFX | Show error | "Unable to read file. The OFX file may be corrupted." |
| No transactions in file | Show error | "No transactions found in this file." |
| Missing metadata fields | Display "N/A" | None (graceful degradation) |
| Transaction with no date | Skip transaction | Console warning only |
| Transaction with no amount | Skip transaction | Console warning only |
| Invalid date format | Skip transaction | Console warning only |
| Very long description | Truncate with ellipsis | None (CSS handles) |
| Very large file (10MB+) | Process normally | None (no size limit) |
| Special characters in description | Display as-is after encoding fix | None |
| Negative balance | Display normally | None (just a number) |
| Multiple accounts in one file | Show only first account | None (OFX typically has one account per file) |

---

## 8. Success Criteria

Each criterion is testable with a YES/NO procedure:

| # | Criterion | Test Procedure |
|---|-----------|----------------|
| 1 | Single HTML file works offline | Open file in browser with network disabled. Can load and use app? YES/NO |
| 2 | Loads ITAU sample correctly | Drop "EXTRATO - 10 - ITAU - OFX.ofx". See 75 transactions? YES/NO |
| 3 | Loads Nubank sample correctly | Drop "NU_93139634_01JAN2026_28JAN2026.ofx". See 7 transactions? YES/NO |
| 4 | Loads Banco Inter sample correctly | Drop "Extrato-29-12-2025-a-29-01-2026-OFX.ofx". See all transactions? YES/NO |
| 5 | Loads SICOOB sample 1 correctly | Drop "extrato-conta-corrente-ofx-unix_202601_20260109153954.ofx". See all transactions? YES/NO |
| 6 | Loads SICOOB sample 2 correctly | Drop "extrato-conta-corrente-ofx-unix_202601_20260114095112.ofx". See all transactions? YES/NO |
| 7 | Loads Vanguarda sample correctly | Drop "extrato 022025.ofx". See all transactions? YES/NO |
| 8 | Portuguese chars display correctly | Check any MEMO field contains "ç", "ã", "á" etc. Characters render properly? YES/NO |
| 9 | Dates in DD-MM-YYYY | Check transaction dates. All in format DD-MM-YYYY? YES/NO |
| 10 | Debits in red | Check negative amount transaction. Text is red? YES/NO |
| 11 | Credits in green | Check positive amount transaction. Text is green? YES/NO |
| 12 | No text wrapping in columns | Resize browser window narrow. Text truncates with ellipsis, never wraps? YES/NO |
| 13 | Drag-drop works | Drag OFX file onto upload zone. File loads? YES/NO |
| 14 | Click to browse works | Click upload zone. File picker opens? YES/NO |
| 15 | Metadata cards show correctly | Check all metadata cards display proper info or "N/A"? YES/NO |
| 16 | Balance displays when available | Check file with balance. Balance card shows amount? YES/NO |
| 17 | Dark theme is professional | Visual inspection. Looks professional and dark blue theme? YES/NO |
| 18 | Error on non-OFX file | Drop .txt file. Error message displays? YES/NO |

---

## 9. Sample Data Analysis

### 9.1 File: EXTRATO - 10 - ITAU - OFX.ofx

**Format:** OFX SGML
**Encoding Declared:** USASCII / CHARSET:1252
**Actual Encoding:** Windows-1252 (Portuguese characters present)
**Structure:** OFXHEADER section + `<OFX>` tags

**Metadata:**
- Bank Name: Not specified (no `<ORG>` tag)
- Bank ID: 0341 (Itaú)
- Account ID: 3839196312
- Account Type: CHECKING
- Currency: BRL
- Date Range: 01/10/2025 to 31/10/2025
- Balance: R$ 25.282,87 (as of 08/01/2026)

**Transactions:** 75 total
- Transaction Fields: `<TRNTYPE>`, `<DTPOSTED>`, `<TRNAMT>`, `<FITID>`, `<CHECKNUM>`, `<MEMO>`
- Date Format: YYYYMMDDHHMMSS[-03:EST]
- Types: CREDIT, DEBIT
- Description in `<MEMO>` field

**Edge Cases Found:**
- No `<ORG>` tag (bank name must be inferred or shown as "Unknown")
- Inconsistent tag closing (some tags have no closing tag)

---

### 9.2 File: extrato 022025.ofx

**Format:** OFX SGML
**Encoding Declared:** USASCII / CHARSET:1252
**Actual Encoding:** Windows-1252 (Portuguese)

**Metadata:**
- Bank Name: CCPI VANGUARDA PR/SP/RJ
- Bank ID: 748
- Account ID: 7100000000845312
- Branch ID: Not specified
- Account Type: CHECKING
- Currency: BRL
- Date Range: 01/02/2025 to 28/02/2025
- Balance: R$ 3.706,58 (as of 28/02/2025)

**Transactions:** ~50
- Fields include `<REFNUM>` in addition to standard fields
- Date Format: YYYYMMDDHHMMSS[-3:GMT]

**Edge Cases Found:**
- Some transactions have very long MEMO text
- Some MEMO fields have special characters and numbers (CPF/CNPJ)

---

### 9.3 File: Extrato-29-12-2025-a-29-01-2026-OFX.ofx

**Format:** OFX SGML
**Encoding Declared:** USASCII / CHARSET:1252
**Actual Encoding:** Windows-1252

**Metadata:**
- Bank Name: Banco Intermedium S/A
- Bank ID: 077
- Account ID: 6839045
- Branch ID: 0001-9
- Account Type: CHECKING
- Currency: BRL
- Date Range: 29/12/2025 to 29/01/2026
- Balance: R$ 65,23 (as of 29/01/2026)

**Transactions:** ~100
- Transaction type: PAYMENT (in addition to CREDIT/DEBIT)
- Has `<NAME>` field in addition to `<MEMO>`
- Date Format: YYYYMMDD (no time)

**Edge Cases Found:**
- Some `<NAME>` fields are empty
- PAYMENT type needs to be handled

---

### 9.4 File: NU_93139634_01JAN2026_28JAN2026.ofx

**Format:** OFX SGML
**Encoding Declared:** UTF-8 / CHARSET:NONE
**Actual Encoding:** UTF-8 (proper encoding!)

**Metadata:**
- Bank Name: NU PAGAMENTOS S.A. (Nubank)
- Bank ID: 0260
- Account ID: 9313963-4
- Branch ID: 1
- Account Type: CHECKING
- Currency: BRL
- Date Range: 01/01/2026 to 28/01/2026
- Balance: R$ 5.408,47 (as of 28/01/2026)

**Transactions:** 7
- Date Format: YYYYMMDDHHMMSS[-3:BRT]
- Has additional balance info in `<BALLIST><BAL>` section

**Edge Cases Found:**
- Only file with proper UTF-8 encoding
- UUIDs used for `<FITID>` instead of numeric IDs

---

### 9.5 File: extrato-conta-corrente-ofx-unix_202601_20260109153954.ofx

**Format:** OFX SGML
**Encoding Declared:** USASCII / CHARSET:1252
**Actual Encoding:** Windows-1252

**Metadata:**
- Bank Name: Banco Cooperativo do Brasil (SICOOB)
- Bank ID: 756
- Account ID: 37888-7
- Branch ID: 4343-5
- Account Type: CHECKING
- Currency: BRL
- Date Range: 01/01/2025 to 31/01/2025

**Transactions:** ~150+
- Heavy use of corrupted characters: "D�B.CONV", "ANTECIPA��O"
- Date Format: YYYYMMDDHHMMSS[-3:BRT]

**Edge Cases Found:**
- Most corrupted encoding of all samples
- Many transactions with special character issues

---

### 9.6 File: extrato-conta-corrente-ofx-unix_202601_20260114095112.ofx

**Format:** OFX SGML
**Encoding Declared:** USASCII / CHARSET:1252
**Actual Encoding:** Windows-1252

**Metadata:**
- Bank Name: Banco Cooperativo do Brasil (SICOOB)
- Bank ID: 756
- Account ID: 55964-4
- Branch ID: 4343-5
- Account Type: CHECKING
- Currency: BRL
- Date Range: 01/01/2025 to 31/01/2025

**Transactions:** ~20
- Similar corruption patterns to previous SICOOB file

---

### 9.7 Cross-File Field Analysis

**Common Metadata Fields (present in all):**
- `<CURDEF>` - Currency
- `<BANKID>` - Bank identifier
- `<ACCTID>` - Account number
- `<ACCTTYPE>` - Account type
- `<DTSTART>` - Statement start date
- `<DTEND>` - Statement end date

**Optional Metadata Fields:**
- `<ORG>` - Bank name (missing in ITAU file)
- `<BRANCHID>` - Branch/agency (missing in some)
- `<LEDGERBAL>` - Balance information

**Common Transaction Fields (all transactions):**
- `<TRNTYPE>` - Transaction type
- `<DTPOSTED>` - Transaction date
- `<TRNAMT>` - Amount
- `<FITID>` - Transaction ID

**Optional Transaction Fields:**
- `<MEMO>` - Description (most common)
- `<NAME>` - Payee name (some banks)
- `<CHECKNUM>` - Check/reference number
- `<REFNUM>` - Reference number

**Date Format Variations:**
1. YYYYMMDDHHMMSS[-03:EST] (ITAU)
2. YYYYMMDDHHMMSS[-3:GMT] (Vanguarda)
3. YYYYMMDD (Banco Inter - no time)
4. YYYYMMDDHHMMSS[-3:BRT] (Nubank, SICOOB)
5. YYYYMMDDHHMMSS[0:GMT] (Nubank server time)

**All date formats:** First 8 characters are always YYYYMMDD

---

## 10. Architecture Decision Records (ADR) Index

ADRs are stored in `.github/adrs/` directory:

- **ADR-001:** Single HTML File vs Multi-File Structure
- **ADR-002:** Custom OFX Parser vs XML Parser Library
- **ADR-003:** Client-Side Only vs Backend Processing
- **ADR-004:** Encoding Handling Strategy

---

## ⚠️ WARNINGS FOR DOWNSTREAM AGENTS

1. **DO NOT add features not in Section 2** - No filtering, searching, sorting, or export features
2. **DO NOT change data structures from Section 4** - The OFXData schema is final
3. **DO NOT simplify algorithms from Section 5** - All edge case handling is mandatory
4. **DO NOT skip edge cases from Section 7** - Every edge case must be handled
5. **DO NOT use external libraries or dependencies** - Pure vanilla JavaScript only
6. **DO NOT create separate CSS/JS files** - Everything must be inline in single HTML
7. **DO NOT assume encoding** - Must handle Windows-1252 in "ASCII" files
8. **DO NOT ignore malformed data** - Must gracefully handle and log issues
9. **IF ANYTHING IS UNCLEAR** - ASK THE USER, do not make assumptions

---

## Document Metadata

- **Version:** 1.0
- **Created:** 2026-02-10
- **Author:** Architect Agent
- **Status:** Complete - Ready for Designer handoff
- **Next Agent:** Designer (for Type A project, Designer creates design-spec.md if needed)
