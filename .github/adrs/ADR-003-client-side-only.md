# ADR-003: Client-Side Only vs Backend Processing

**Status:** Accepted  
**Date:** 2026-02-10  
**Deciders:** User, Architect

## Context

Processing OFX files requires reading file content, parsing data, and displaying results. Need to decide whether to handle this entirely in the browser (client-side) or involve a backend server.

## Decision Drivers

1. User requirement: single self-contained HTML file
2. Privacy concerns with financial data
3. Offline capability requirement
4. Simplicity for end users
5. No server infrastructure wanted

## Options Considered

### Option 1: Pure Client-Side JavaScript (SELECTED)

All processing happens in the browser using FileReader API and vanilla JavaScript.

**Architecture:**
```
User Browser
    ↓ (select file)
FileReader API
    ↓ (read as text)
JavaScript Parser
    ↓ (extract data)
DOM Manipulation
    ↓ (display)
User sees results
```

**Pros:**
- ✅ Perfect match for single HTML file requirement
- ✅ Works completely offline
- ✅ Financial data never leaves user's computer (privacy)
- ✅ No server costs or maintenance
- ✅ Instant processing (no network latency)
- ✅ Works from local filesystem
- ✅ No authentication/authorization needed
- ✅ Scales infinitely (each user's own browser)
- ✅ No CORS issues

**Cons:**
- ❌ Limited by browser capabilities
- ❌ Could struggle with very large files (10MB+)
- ❌ No server-side validation or sanitization
- ❌ Can't save results to database
- ❌ Each user processes their own files (no sharing)

### Option 2: Backend Server Processing (REJECTED)

Upload file to server, process there, return results.

**Architecture:**
```
User Browser
    ↓ (upload file via HTTP)
Backend Server (Node.js/Python/etc)
    ↓ (parse OFX)
    ↓ (return JSON)
User Browser
    ↓ (display results)
```

**Pros:**
- ✅ Can handle any file size
- ✅ Server-side validation possible
- ✅ Could store results in database
- ✅ Could offer additional features (export, history)

**Cons:**
- ❌ Violates single HTML file requirement
- ❌ Requires server infrastructure
- ❌ Server hosting costs
- ❌ Privacy concerns (financial data uploaded to server)
- ❌ Requires internet connection
- ❌ Network latency
- ❌ Authentication/security complexity
- ❌ CORS configuration needed
- ❌ Much more complex deployment

### Option 3: Hybrid - Optional Backend (REJECTED)

Client-side by default, optional backend for advanced features.

**Pros:**
- ✅ Best of both worlds
- ✅ Privacy by default
- ✅ Advanced features available

**Cons:**
- ❌ Significantly more complex
- ❌ Two codebases to maintain
- ❌ Still violates single file requirement
- ❌ User said no backend features needed
- ❌ Not requested by user

## Decision

**Selected: Option 1 - Pure Client-Side JavaScript**

User requirements explicitly state:
- "single self contained html web app"
- "no limits" on file size (trust browser to handle)
- Must work offline

Financial data privacy is paramount. Processing files entirely in the browser means:
1. Data never transmitted over network
2. No server to secure or maintain
3. No data retention or storage concerns
4. Complete user control

The FileReader API is mature and well-supported across all modern browsers. File sizes for bank statements are typically small (under 1MB), well within browser capabilities.

## Consequences

### Positive
- Maximum privacy and security
- Zero backend costs
- Works offline
- Simple distribution (just share HTML file)
- No server maintenance
- No attack surface for data breaches

### Negative
- Limited to browser capabilities
- No persistent storage (unless localStorage added later)
- Each user processes independently
- Can't aggregate or analyze across users

### Neutral
- File size limits depend on user's device
- Performance varies by user's browser/computer
- No telemetry or usage analytics (could be seen as positive for privacy)

## Technical Implementation

**Required Browser APIs:**
- `FileReader` - Read file content
- `File` API - Handle file objects
- `Blob` API - Optional, for advanced handling
- DOM APIs - Display results

**FileReader approach:**
```javascript
const reader = new FileReader();
reader.onload = (e) => {
    const content = e.target.result;
    // Process content
};
reader.readAsText(file, 'UTF-8');
```

**No server endpoints, no AJAX calls, no fetch() needed.**

## Security Considerations

Client-side processing is MORE secure for this use case:
- ✅ No data transmission (no man-in-the-middle attacks)
- ✅ No server storage (no database breaches)
- ✅ No authentication tokens to steal
- ✅ No API endpoints to attack
- ✅ User has complete control

Standard web security still applies:
- Must sanitize data before inserting into DOM (XSS prevention)
- Use `textContent` instead of `innerHTML` where possible
- Escape HTML entities in descriptions

## Compliance

All downstream agents must:
- ✅ Use FileReader API to read files
- ✅ Process all data in browser JavaScript
- ✅ Never make HTTP requests (no fetch, no AJAX, no external resources)
- ✅ Implement XSS prevention for user data
- ❌ NO backend server or API calls
- ❌ NO file uploads to external services
- ❌ NO data transmission over network
- ❌ NO external resources (except browser APIs)
