# ADR-002: Custom OFX Parser vs XML Parser Library

**Status:** Accepted  
**Date:** 2026-02-10  
**Deciders:** Architect

## Context

OFX (Open Financial Exchange) files use SGML format, which is similar to but not quite XML. Brazilian bank OFX files have inconsistent structure, optional closing tags, and encoding issues. Need to decide how to parse these files.

## Decision Drivers

1. OFX is SGML, not XML (XML parsers may fail)
2. Brazilian bank files have inconsistent tag closing
3. Must work without external libraries (single HTML file requirement)
4. Need to handle encoding issues gracefully
5. Simplicity over complexity

## Options Considered

### Option 1: Custom Regex-Based Parser (SELECTED)

Extract data using regular expressions and string manipulation.

**Approach:**
```javascript
function extractTag(content, tagName) {
    const regex = new RegExp(`<${tagName}>([^<]*)`);
    const match = content.match(regex);
    return match ? match[1].trim() : null;
}
```

**Pros:**
- ✅ No external dependencies
- ✅ Works with malformed SGML
- ✅ Handles optional closing tags
- ✅ Simple to understand and debug
- ✅ Can handle encoding issues before parsing
- ✅ Tolerant of variations across banks
- ✅ Fast for small files
- ✅ Easy to customize for specific issues

**Cons:**
- ❌ Not theoretically "correct" parsing
- ❌ Could break with deeply nested or complex structures
- ❌ Requires manual pattern for each field
- ❌ Harder to maintain if OFX structure changes significantly

### Option 2: DOMParser (REJECTED)

Use browser's built-in DOMParser API.

**Approach:**
```javascript
const parser = new DOMParser();
const doc = parser.parseFromString(content, "text/xml");
```

**Pros:**
- ✅ Native browser API (no external library)
- ✅ Proper XML parsing
- ✅ Can navigate DOM tree

**Cons:**
- ❌ OFX is SGML, not XML - may fail on unclosed tags
- ❌ Brazilian bank files often invalid XML
- ❌ Less tolerant of malformed content
- ❌ Error handling complex
- ❌ Testing showed failures on real files

### Option 3: External XML/SGML Parser Library (REJECTED)

Import library like xml2js, node-html-parser, etc.

**Examples:**
- `xml2js`
- `fast-xml-parser`
- `node-html-parser`

**Pros:**
- ✅ Robust parsing
- ✅ Handles edge cases
- ✅ Well-tested

**Cons:**
- ❌ Violates single HTML file requirement
- ❌ External dependency
- ❌ User requirement: no dependencies
- ❌ Increases file size significantly
- ❌ Requires CDN link (breaks offline capability) or embedding entire library
- ❌ Overkill for this use case

### Option 4: String Split and Manual Navigation (REJECTED)

Split file by tags and manually find matching pairs.

**Pros:**
- ✅ No dependencies
- ✅ Simple concept

**Cons:**
- ❌ Very error-prone
- ❌ Difficult to handle nested structures
- ❌ Hard to debug
- ❌ Regex is cleaner for tag extraction

## Decision

**Selected: Option 1 - Custom Regex-Based Parser**

Analysis of the 6 sample files shows:
1. All files have inconsistent closing tags
2. Some tags self-close, others don't
3. Structure is flat enough that regex works well
4. Need tolerance for malformed content
5. Single HTML file requirement eliminates external libraries

The regex approach is pragmatic and fits the reality of Brazilian bank OFX files. It's not theoretically pure, but it works reliably for the actual data format we need to handle.

## Consequences

### Positive
- Simple implementation
- No dependencies
- Works with all 6 sample files
- Easy to debug issues
- Can add encoding fixes before parsing
- Fast enough for typical file sizes

### Negative
- Not reusable for complex XML/SGML beyond OFX
- Requires regex knowledge to maintain
- Each new field needs explicit extraction logic

### Mitigation
- Document all regex patterns clearly
- Test against all 6 sample files
- Add console warnings for unparseable sections
- Keep extraction functions small and focused

## Implementation Notes

**Required extraction functions:**
```javascript
extractTag(content, tagName)           // Single value
extractAllBetween(content, start, end) // Multiple blocks
extractTagWithDefault(content, tag, defaultValue)
```

**Critical patterns:**
- Tags may or may not have closing tags: `<TAG>value` or `<TAG>value</TAG>`
- Must handle both formats: `<TAG>([^<]*?)</TAG>` or `<TAG>([^<]*)`
- Transaction blocks use `<STMTTRN>...</STMTTRN>` with closing tags
- Extract all transaction blocks, then parse each individually

## Compliance

All downstream agents must:
- ✅ Use regex-based extraction as specified in PCD Section 5
- ✅ Test against all 6 sample files
- ❌ NO external parsing libraries
- ❌ NO DOMParser for OFX content
- ❌ NO assumptions about well-formed XML
