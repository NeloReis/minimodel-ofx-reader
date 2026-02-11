# Implementation Plan
## OFX Reader - Code Mapping Document

---

## Overview

This document maps every requirement from the PCD and Design Spec to specific code sections and functions.

---

## Code Organization Structure

```
index.html
├── <head>
│   ├── <meta> tags
│   ├── <title>
│   └── <style> [Complete CSS from design-spec.md Section 2]
└── <body>
    ├── [Complete HTML from design-spec.md Section 3]
    └── <script>
        ├── Section 1: Config & Constants
        ├── Section 2: State Management
        ├── Section 3: Utility Functions
        ├── Section 4: Core Logic (OFX Parsing)
        ├── Section 5: UI Rendering
        ├── Section 6: Event Handlers
        └── Section 7: Initialization
```

---

## Section 1: Configuration & Constants

```javascript
const CONFIG = {
  fileAccept: '.ofx',
  dateFormat: 'DD-MM-YYYY',
  currency: 'BRL',
  locale: 'pt-BR',
};
```

**Source:** PCD Section 2.1 (Requirements)

---

## Section 2: State Management

**From:** coding-patterns.md

```javascript
const AppState = {
  status: 'initial',  // 'initial' | 'loading' | 'success' | 'error'
  ofxData: null,      // Parsed OFX data (metadata + transactions)
  error: null,        // Error message string
  fileName: null,     // Currently loaded file name
};

function updateState(newState) {
  Object.assign(AppState, newState);
  render();
}
```

**Application States from Design Spec Section 4:**
- initial: No file loaded
- loading: Processing file
- success: Data loaded and displayed
- error: Error occurred

---

## Section 3: Utility Functions

### 3.1 Safe Data Access

**From:** coding-patterns.md

```javascript
// safeGet(obj, path, defaultValue)
// safeNumber(value, defaultValue)
// safeString(value, defaultValue)
// safeArray(value)
```

### 3.2 XSS Prevention

**From:** coding-patterns.md

```javascript
// escapeHtml(str)
// createElement(tag, attributes, children)
```

### 3.3 Error Boundaries

**From:** coding-patterns.md

```javascript
// safeHandler(fn)
```

---

## Section 4: Core Logic (OFX Parsing)

### 4.1 fixPortugueseEncoding(text)

**From:** PCD Section 5.2

**Purpose:** Fix Windows-1252 corruption in "ASCII" declared files

**Implementation:**
```javascript
function fixPortugueseEncoding(text) {
  if (!text) return '';
  
  const replacements = {
    'Ã§': 'ç', 'Ã£': 'ã', 'Ã¡': 'á', 'Ã©': 'é',
    'Ã­': 'í', 'Ã³': 'ó', 'Ãº': 'ú', 'Ã ': 'à',
    'Ã¢': 'â', 'Ãª': 'ê', 'Ã´': 'ô', 'Ã£o': 'ão',
    // Add uppercase versions
    'Ã‡': 'Ç', 'Ã': 'Ã', 'Ã': 'Á', 'Ã‰': 'É',
    'Ã': 'Í', 'Ã"': 'Ó', 'Ãš': 'Ú', 'Ã€': 'À',
    'Ã‚': 'Â', 'ÃŠ': 'Ê', 'Ã"': 'Ô',
  };
  
  let result = text;
  for (const [corrupt, correct] of Object.entries(replacements)) {
    result = result.replaceAll(corrupt, correct);
  }
  
  return result;
}
```

**Test Cases:**
- "D�B.CONV" → "DÉB.CONV"
- "ANTECIPA��O" → "ANTECIPAÇÃO"

### 4.2 extractTag(content, tagName)

**From:** PCD Section 5.1 (used in parseOFXFile)

**Purpose:** Extract single tag value from OFX content

**Implementation:**
```javascript
function extractTag(content, tagName) {
  if (!content || !tagName) return null;
  
  // Pattern: <TAG>value or <TAG>value</TAG>
  const regex = new RegExp(`<${tagName}>([^<]*(?:</[^>]+>)?)`, 'i');
  const match = content.match(regex);
  
  if (!match) return null;
  
  let value = match[1].trim();
  // Remove closing tag if present
  value = value.replace(/<\/[^>]+>$/, '').trim();
  
  return value || null;
}
```

### 4.3 extractAllBetween(content, startTag, endTag)

**From:** PCD Section 5.1 (used in parseOFXFile)

**Purpose:** Extract all blocks between start and end tags

**Implementation:**
```javascript
function extractAllBetween(content, startTag, endTag) {
  if (!content || !startTag || !endTag) return [];
  
  const blocks = [];
  const regex = new RegExp(`${startTag}([\\s\\S]*?)${endTag}`, 'gi');
  let match;
  
  while ((match = regex.exec(content)) !== null) {
    blocks.push(match[1]);
  }
  
  return blocks;
}
```

### 4.4 parseOFXDate(dateStr)

**From:** PCD Section 5.3

**Purpose:** Convert OFX date formats to DD-MM-YYYY

**Implementation:**
```javascript
function parseOFXDate(dateStr) {
  if (!dateStr) return null;
  
  // Remove timezone: YYYYMMDDHHMMSS[-03:EST] → YYYYMMDD
  const cleanDate = dateStr.split('[')[0].substring(0, 8);
  
  if (cleanDate.length !== 8) return null;
  
  const year = cleanDate.substring(0, 4);
  const month = cleanDate.substring(4, 6);
  const day = cleanDate.substring(6, 8);
  
  // Validate
  const y = parseInt(year, 10);
  const m = parseInt(month, 10);
  const d = parseInt(day, 10);
  
  if (isNaN(y) || isNaN(m) || isNaN(d)) return null;
  if (m < 1 || m > 12) return null;
  if (d < 1 || d > 31) return null;
  
  return `${day}-${month}-${year}`;
}
```

**Test Cases:**
- "20251001100000[-03:EST]" → "01-10-2025"
- "20260129" → "29-01-2026"
- "invalid" → null

### 4.5 parseTransaction(block)

**From:** PCD Section 5.4

**Purpose:** Extract transaction from STMTTRN block

**Implementation:**
```javascript
function parseTransaction(block) {
  if (!block) return null;
  
  const typeRaw = extractTag(block, 'TRNTYPE');
  const dateRaw = extractTag(block, 'DTPOSTED');
  const amountRaw = extractTag(block, 'TRNAMT');
  const memo = extractTag(block, 'MEMO') || extractTag(block, 'NAME') || 'No description';
  
  // Validate required fields
  if (!dateRaw || !amountRaw) {
    console.warn('[parseTransaction] Skipping: missing date or amount');
    return null;
  }
  
  const date = parseOFXDate(dateRaw);
  if (!date) {
    console.warn('[parseTransaction] Skipping: invalid date', dateRaw);
    return null;
  }
  
  const amount = parseFloat(amountRaw);
  if (isNaN(amount)) {
    console.warn('[parseTransaction] Skipping: invalid amount', amountRaw);
    return null;
  }
  
  // Map transaction type
  let type = 'OTHER';
  if (typeRaw) {
    const typeUpper = typeRaw.toUpperCase();
    if (['CREDIT', 'DEBIT', 'PAYMENT', 'OTHER'].includes(typeUpper)) {
      type = typeUpper;
    }
  }
  
  return {
    type,
    date,
    amount,
    description: memo.trim(),
  };
}
```

**Test Cases:**
- Valid block → transaction object
- Missing TRNAMT → null (with warning)
- Invalid date → null (with warning)

### 4.6 parseOFXFile(fileContent)

**From:** PCD Section 5.1

**Purpose:** Main parser - extract all data from OFX file

**Implementation:**
```javascript
function parseOFXFile(fileContent) {
  if (!fileContent || typeof fileContent !== 'string') {
    throw new Error('Empty file content');
  }
  
  // Fix encoding issues first
  const content = fixPortugueseEncoding(fileContent);
  
  // Validate OFX header
  if (!content.includes('OFXHEADER') && !content.includes('<OFX>')) {
    throw new Error('Invalid OFX file');
  }
  
  // Initialize result
  const result = {
    metadata: {},
    transactions: [],
  };
  
  // Extract metadata
  result.metadata.bankName = extractTag(content, 'ORG') || 'Unknown Bank';
  result.metadata.bankId = extractTag(content, 'BANKID') || extractTag(content, 'FID');
  result.metadata.accountId = extractTag(content, 'ACCTID');
  result.metadata.branchId = extractTag(content, 'BRANCHID');
  result.metadata.accountType = extractTag(content, 'ACCTTYPE');
  result.metadata.currency = extractTag(content, 'CURDEF') || 'BRL';
  
  // Extract dates
  const startDateRaw = extractTag(content, 'DTSTART');
  if (startDateRaw) {
    result.metadata.startDate = parseOFXDate(startDateRaw);
  }
  
  const endDateRaw = extractTag(content, 'DTEND');
  if (endDateRaw) {
    result.metadata.endDate = parseOFXDate(endDateRaw);
  }
  
  // Extract balance
  const balanceRaw = extractTag(content, 'BALAMT');
  if (balanceRaw) {
    result.metadata.balance = parseFloat(balanceRaw);
  }
  
  const balanceDateRaw = extractTag(content, 'DTASOF');
  if (balanceDateRaw) {
    result.metadata.balanceDate = parseOFXDate(balanceDateRaw);
  }
  
  // Extract transactions
  const transactionBlocks = extractAllBetween(content, '<STMTTRN>', '</STMTTRN>');
  
  for (const block of transactionBlocks) {
    const transaction = parseTransaction(block);
    if (transaction) {
      result.transactions.push(transaction);
    }
  }
  
  if (result.transactions.length === 0) {
    throw new Error('No valid transactions found');
  }
  
  return result;
}
```

**Test Cases:**
- Valid ITAU file → 75 transactions
- Empty file → Error "Empty file content"
- Non-OFX file → Error "Invalid OFX file"
- No transactions → Error "No valid transactions found"

### 4.7 formatCurrency(value)

**From:** PCD Section 5.5

**Purpose:** Format amounts as Brazilian Real (R$ X.XXX,XX)

**Implementation:**
```javascript
function formatCurrency(value) {
  if (value == null || isNaN(value)) {
    return 'R$ 0,00';
  }
  
  const absValue = Math.abs(value);
  
  // Format to 2 decimal places
  let formatted = absValue.toFixed(2); // "1234.56"
  
  // Replace decimal point with comma
  const parts = formatted.split('.');
  const integerPart = parts[0];
  const decimalPart = parts[1];
  
  // Add thousand separators
  let withSeparators = '';
  for (let i = integerPart.length - 1, count = 0; i >= 0; i--, count++) {
    if (count > 0 && count % 3 === 0) {
      withSeparators = '.' + withSeparators;
    }
    withSeparators = integerPart[i] + withSeparators;
  }
  
  return `R$ ${withSeparators},${decimalPart}`;
}
```

**Test Cases:**
- 1410.00 → "R$ 1.410,00"
- -856.47 → "R$ 856,47" (absolute value)
- 25282.87 → "R$ 25.282,87"

### 4.8 handleFileUpload(file)

**From:** PCD Section 5.6

**Purpose:** Process uploaded file

**Implementation:**
```javascript
function handleFileUpload(file) {
  if (!file) {
    updateState({ status: 'error', error: 'No file selected' });
    return;
  }
  
  if (!file.name.toLowerCase().endsWith('.ofx')) {
    updateState({ status: 'error', error: 'Please select an OFX file' });
    return;
  }
  
  updateState({ status: 'loading', fileName: file.name });
  
  const reader = new FileReader();
  
  reader.onload = function(e) {
    try {
      const content = e.target.result;
      const ofxData = parseOFXFile(content);
      updateState({ 
        status: 'success', 
        ofxData,
        error: null,
      });
    } catch (error) {
      console.error('[handleFileUpload] Parse error:', error);
      updateState({ 
        status: 'error', 
        error: error.message || 'Unable to read file. The OFX file may be corrupted.',
      });
    }
  };
  
  reader.onerror = function() {
    updateState({ 
      status: 'error', 
      error: 'Failed to read file. Please try again.',
    });
  };
  
  reader.readAsText(file, 'UTF-8');
}
```

---

## Section 5: UI Rendering Functions

**From:** Design Spec Section 3 (HTML Structure)

### 5.1 render()

**Master render function - dispatches to state-specific renderers**

```javascript
function render() {
  // Show/hide sections based on state
  const uploadSection = document.querySelector('.upload-section');
  const loadingContainer = document.getElementById('loadingContainer');
  const errorMessage = document.getElementById('errorMessage');
  const dataSection = document.getElementById('dataSection');
  
  // Reset visibility
  loadingContainer.classList.remove('visible');
  errorMessage.classList.remove('visible');
  dataSection.classList.remove('visible');
  
  switch (AppState.status) {
    case 'initial':
      // Upload section visible, others hidden (default state)
      break;
      
    case 'loading':
      loadingContainer.classList.add('visible');
      break;
      
    case 'success':
      dataSection.classList.add('visible');
      renderMetadata();
      renderTransactions();
      break;
      
    case 'error':
      errorMessage.classList.add('visible');
      document.getElementById('errorText').textContent = AppState.error || 'An error occurred';
      break;
  }
}
```

### 5.2 renderMetadata()

**Render metadata cards**

```javascript
function renderMetadata() {
  const container = document.getElementById('metadataGrid');
  if (!container || !AppState.ofxData) return;
  
  container.innerHTML = '';
  const metadata = AppState.ofxData.metadata;
  
  // Define cards
  const cards = [
    {
      label: 'Banco',
      value: metadata.bankName || 'N/A',
      subvalue: metadata.bankId ? `Código: ${metadata.bankId}` : null,
    },
    {
      label: 'Conta',
      value: metadata.accountId || 'N/A',
      subvalue: metadata.accountType || null,
    },
    {
      label: 'Agência',
      value: metadata.branchId || 'N/A',
      subvalue: null,
    },
    {
      label: 'Período',
      value: metadata.startDate && metadata.endDate 
        ? `${metadata.startDate} a ${metadata.endDate}`
        : 'N/A',
      subvalue: null,
    },
    {
      label: 'Saldo',
      value: metadata.balance != null ? formatCurrency(metadata.balance) : 'N/A',
      subvalue: metadata.balanceDate ? `Em ${metadata.balanceDate}` : null,
      highlight: metadata.balance != null,
    },
    {
      label: 'Moeda',
      value: metadata.currency || 'N/A',
      subvalue: null,
    },
  ];
  
  // Create cards
  for (const card of cards) {
    const cardEl = document.createElement('div');
    cardEl.className = 'metadata-card';
    
    const label = document.createElement('div');
    label.className = 'card-label';
    label.textContent = card.label;
    cardEl.appendChild(label);
    
    const value = document.createElement('div');
    value.className = card.highlight ? 'card-value highlight' : 'card-value';
    value.textContent = card.value;
    cardEl.appendChild(value);
    
    if (card.subvalue) {
      const subvalue = document.createElement('div');
      subvalue.className = 'card-subvalue';
      subvalue.textContent = card.subvalue;
      cardEl.appendChild(subvalue);
    }
    
    container.appendChild(cardEl);
  }
}
```

### 5.3 renderTransactions()

**Render transaction table**

```javascript
function renderTransactions() {
  const tbody = document.getElementById('transactionTableBody');
  const countText = document.getElementById('countText');
  
  if (!tbody || !AppState.ofxData) return;
  
  tbody.innerHTML = '';
  const transactions = AppState.ofxData.transactions;
  
  // Update count
  if (countText) {
    const count = transactions.length;
    countText.textContent = `${count} transação${count !== 1 ? 'ões' : ''}`;
  }
  
  // Create rows
  for (const transaction of transactions) {
    const row = document.createElement('tr');
    
    // Date cell
    const dateCell = document.createElement('td');
    dateCell.className = 'date-cell';
    dateCell.textContent = transaction.date;
    row.appendChild(dateCell);
    
    // Amount cell
    const amountCell = document.createElement('td');
    const amountClass = transaction.amount >= 0 ? 'credit' : 'debit';
    amountCell.className = `amount-cell ${amountClass}`;
    amountCell.textContent = formatCurrency(transaction.amount);
    row.appendChild(amountCell);
    
    // Description cell
    const descCell = document.createElement('td');
    descCell.className = 'description-cell';
    descCell.textContent = transaction.description;
    row.appendChild(descCell);
    
    tbody.appendChild(row);
  }
}
```

---

## Section 6: Event Handlers

**All handlers wrapped in safeHandler from coding-patterns.md**

### 6.1 Upload Zone Click

```javascript
const uploadZone = document.getElementById('uploadZone');
const fileInput = document.getElementById('fileInput');

uploadZone.addEventListener('click', safeHandler(function() {
  fileInput.click();
}));
```

### 6.2 File Input Change

```javascript
fileInput.addEventListener('change', safeHandler(function(e) {
  const file = e.target.files[0];
  if (file) {
    handleFileUpload(file);
  }
  // Reset input so same file can be selected again
  e.target.value = '';
}));
```

### 6.3 Drag and Drop

```javascript
uploadZone.addEventListener('dragover', safeHandler(function(e) {
  e.preventDefault();
  uploadZone.classList.add('dragover');
}));

uploadZone.addEventListener('dragleave', safeHandler(function(e) {
  e.preventDefault();
  uploadZone.classList.remove('dragover');
}));

uploadZone.addEventListener('drop', safeHandler(function(e) {
  e.preventDefault();
  uploadZone.classList.remove('dragover');
  
  const file = e.dataTransfer.files[0];
  if (file) {
    handleFileUpload(file);
  }
}));
```

---

## Section 7: Initialization

```javascript
document.addEventListener('DOMContentLoaded', function() {
  try {
    // Initialize in initial state
    updateState({ status: 'initial' });
    
    // Set up event listeners
    setupEventListeners();
  } catch (error) {
    console.error('[init] Failed to initialize:', error);
    document.body.innerHTML = '<p style="color: #f44336; padding: 2rem;">Failed to start application. Please refresh the page.</p>';
  }
});

function setupEventListeners() {
  const uploadZone = document.getElementById('uploadZone');
  const fileInput = document.getElementById('fileInput');
  
  if (!uploadZone || !fileInput) {
    throw new Error('Required DOM elements not found');
  }
  
  // Upload zone click
  uploadZone.addEventListener('click', safeHandler(function() {
    fileInput.click();
  }));
  
  // File input change
  fileInput.addEventListener('change', safeHandler(function(e) {
    const file = e.target.files[0];
    if (file) {
      handleFileUpload(file);
    }
    e.target.value = '';
  }));
  
  // Drag and drop
  uploadZone.addEventListener('dragover', safeHandler(function(e) {
    e.preventDefault();
    uploadZone.classList.add('dragover');
  }));
  
  uploadZone.addEventListener('dragleave', safeHandler(function(e) {
    e.preventDefault();
    uploadZone.classList.remove('dragover');
  }));
  
  uploadZone.addEventListener('drop', safeHandler(function(e) {
    e.preventDefault();
    uploadZone.classList.remove('dragover');
    
    const file = e.dataTransfer.files[0];
    if (file) {
      handleFileUpload(file);
    }
  }));
}
```

---

## Testing Plan

### Manual Test Checklist

Based on PCD Section 8 (Success Criteria):

1. [ ] Load ITAU sample → see 75 transactions
2. [ ] Load Nubank sample → see 7 transactions
3. [ ] Load Banco Inter sample → see all transactions
4. [ ] Load SICOOB sample 1 → see all transactions
5. [ ] Load SICOOB sample 2 → see all transactions
6. [ ] Load Vanguarda sample → see all transactions
7. [ ] Portuguese characters display correctly (ç, á, ã, é)
8. [ ] Dates in DD-MM-YYYY format
9. [ ] Debits in red
10. [ ] Credits in green
11. [ ] No text wrapping in table columns
12. [ ] Drag-drop works
13. [ ] Click to browse works
14. [ ] Metadata cards show correctly
15. [ ] Balance displays when available
16. [ ] Dark theme is professional
17. [ ] Error on non-OFX file
18. [ ] Works offline (network disabled)

---

## PCD Compliance Checklist

- [x] All algorithms from PCD Section 5 mapped to functions
- [x] All edge cases from PCD Section 7 handled
- [x] Every UI state renders (no blank screens)
- [x] Designer's CSS used unchanged
- [x] Zero console errors planned for
- [x] No features added beyond PCD Section 2
- [x] `use strict`, `const`/`let` only
- [x] Test checklist created

---

## File Structure

```
index.html (single file)
├── <!DOCTYPE html>
├── <html lang="pt-BR">
├── <head>
│   ├── <meta charset="UTF-8">
│   ├── <meta name="viewport">
│   ├── <meta name="description">
│   ├── <title>
│   └── <style>
│       └── [Complete CSS from design-spec.md Section 2]
└── <body>
    ├── <div class="app-container">
    │   ├── <header class="app-header">
    │   ├── <section class="upload-section">
    │   ├── <div class="loading-container">
    │   ├── <div class="error-message">
    │   └── <div class="data-section">
    │       ├── <section class="metadata-section">
    │       └── <section class="transactions-section">
    └── <script>
        ├── 'use strict';
        ├── Section 1: Config & Constants
        ├── Section 2: State Management
        ├── Section 3: Utility Functions
        ├── Section 4: Core Logic
        ├── Section 5: UI Rendering
        ├── Section 6: Event Handlers (via setupEventListeners)
        └── Section 7: Initialization
```

---

## Document Metadata

- **Version:** 1.0
- **Created:** 2026-02-10
- **Author:** Implementer Agent (Planning Phase)
- **Status:** Complete - Ready for implementation
- **Next Step:** Create index.html with all code
