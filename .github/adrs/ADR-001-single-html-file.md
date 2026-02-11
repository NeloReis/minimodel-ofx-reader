# ADR-001: Single HTML File vs Multi-File Structure

**Status:** Accepted  
**Date:** 2026-02-10  
**Deciders:** User, Architect

## Context

Building an OFX reader application for Brazilian bank statements. Need to decide on file structure: single self-contained HTML file vs traditional multi-file structure (separate HTML, CSS, JS files).

## Decision Drivers

1. User explicitly requested "single self contained html web app"
2. Portability and ease of sharing
3. Offline capability
4. No server or build process required
5. Simplicity for end users

## Options Considered

### Option 1: Single HTML File (SELECTED)

**Structure:**
```html
<!DOCTYPE html>
<html>
<head>
    <style>/* All CSS here */</style>
</head>
<body>
    <!-- All HTML here -->
    <script>/* All JavaScript here */</script>
</body>
</html>
```

**Pros:**
- ✅ User explicitly requested this
- ✅ Maximum portability (single file to share)
- ✅ No dependencies or external resources
- ✅ Works offline by default
- ✅ No build process needed
- ✅ Can be opened directly in any browser
- ✅ Easy to email or share via messaging

**Cons:**
- ❌ Harder to maintain/debug (no file separation)
- ❌ No syntax highlighting separation in editors
- ❌ Larger single file size
- ❌ Browser must load everything at once (though small for this project)

### Option 2: Multi-File Structure (REJECTED)

**Structure:**
```
index.html
styles.css
script.js
```

**Pros:**
- ✅ Better separation of concerns
- ✅ Easier development and debugging
- ✅ Better syntax highlighting
- ✅ Could use modules

**Cons:**
- ❌ User did not request this
- ❌ Requires multiple files (harder to share)
- ❌ Needs correct relative paths
- ❌ Some browsers block local file access (CORS)
- ❌ More complex for non-technical users

### Option 3: Single HTML + Base64 Encoded Resources (REJECTED)

Could embed any external resources as base64 data URIs.

**Pros:**
- ✅ Single file
- ✅ Could include images/fonts

**Cons:**
- ❌ Unnecessary complexity for this project (no images needed)
- ❌ Larger file size with base64 encoding
- ❌ No clear benefit over Option 1

## Decision

**Selected: Option 1 - Single HTML File**

The user explicitly requested "single self contained html web app", making this a requirement rather than a preference. The application is small enough that combining everything into one file won't cause maintenance issues, and the benefits for end users (portability, sharing, offline use) far outweigh the developer convenience of separated files.

## Consequences

### Positive
- Perfect match for user requirements
- Maximum portability and ease of use
- Zero configuration or setup for end users
- No dependency management

### Negative
- Development requires discipline to organize code within single file
- Must use comments extensively for code organization
- No ability to use CSS preprocessors or JS transpilers without build step

### Neutral
- File size ~50-100KB (acceptable for single file)
- Could create development version with separate files if needed, then concatenate for production

## Compliance

This decision is mandatory for all downstream agents:
- ✅ Implementer MUST create single HTML file
- ✅ ALL CSS must be in `<style>` tag in `<head>`
- ✅ ALL JavaScript must be in `<script>` tag before `</body>`
- ❌ NO external file references (except for standard browser APIs)
- ❌ NO CDN links for libraries
- ❌ NO separate .css or .js files
