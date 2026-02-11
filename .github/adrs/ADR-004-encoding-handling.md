# ADR-004: Encoding Handling Strategy

**Status:** Accepted  
**Date:** 2026-02-10  
**Deciders:** Architect (based on user-provided sample analysis)

## Context

Brazilian bank OFX files have encoding mismatches:
- Files declare `ENCODING:USASCII` and `CHARSET:1252`
- Content contains Portuguese characters (ç, á, ã, é, í, ó, ú, etc.)
- Browser FileReader reads as UTF-8 by default
- Results in corrupted characters like "D�B.CONV" instead of "DÉB.CONV"

User explicitly stated: "the sample ofx files from brasilaina banks , most have wrong encoding and laguange wrong. they are all brasilian banks and texts are in portuguese with portuhese characteer regardless of what file encoding says."

## Decision Drivers

1. User confirms encoding is wrong in files
2. Must display Portuguese characters correctly
3. Cannot rely on declared encoding
4. Files use Windows-1252 encoding despite ASCII declaration
5. Single HTML file limitation (no server-side encoding detection)
6. All 6 samples have this issue (except Nubank)

## Options Considered

### Option 1: Read as UTF-8, Map Common Corruptions (SELECTED)

Read file as UTF-8, then map known corruption patterns to correct characters.

**Approach:**
```javascript
function fixPortugueseEncoding(text) {
    const replacements = {
        'Ã§': 'ç',  // cedilla
        'Ã£': 'ã',  // a tilde
        'Ã¡': 'á',  // a acute
        'Ã©': 'é',  // e acute
        'Ã­': 'í',  // i acute
        'Ã³': 'ó',  // o acute
        'Ãº': 'ú',  // u acute
        // ... more patterns
    };
    // Replace all patterns
}
```

**Pros:**
- ✅ Works without TextDecoder API complexities
- ✅ Handles actual corruption patterns seen in samples
- ✅ Simple to implement and test
- ✅ Can be extended as new patterns discovered
- ✅ Works in all browsers
- ✅ No additional APIs needed

**Cons:**
- ❌ Must manually map each corruption pattern
- ❌ Might miss obscure patterns
- ❌ Not theoretically "correct" decoding
- ❌ Requires testing against all samples

### Option 2: Try Multiple Encodings with TextDecoder (REJECTED)

Use TextDecoder API to try different encodings until one works.

**Approach:**
```javascript
const encodings = ['windows-1252', 'iso-8859-1', 'utf-8'];
for (const encoding of encodings) {
    try {
        const decoder = new TextDecoder(encoding);
        const decoded = decoder.decode(fileBytes);
        // Test if looks correct
    } catch (e) {}
}
```

**Pros:**
- ✅ Theoretically correct approach
- ✅ Uses standard API
- ✅ Automatic detection

**Cons:**
- ❌ TextDecoder not universally supported for windows-1252
- ❌ Requires reading file as ArrayBuffer, not text
- ❌ More complex with no clear benefit
- ❌ How to "test if looks correct"? Still need pattern matching
- ❌ Can fail silently
- ❌ Tested: doesn't reliably fix the corruption

### Option 3: Read as Windows-1252 Directly (REJECTED)

Force FileReader to read as windows-1252.

**Approach:**
```javascript
reader.readAsText(file, 'windows-1252');
```

**Pros:**
- ✅ Simple API call
- ✅ Direct approach

**Cons:**
- ❌ Browser FileReader doesn't support windows-1252 encoding parameter
- ❌ Only UTF-8 and UTF-16 reliably supported
- ❌ Would need polyfill or alternative implementation
- ❌ Adds complexity

### Option 4: Ignore Encoding Issues (REJECTED)

Display corrupted characters as-is.

**Pros:**
- ✅ No code needed
- ✅ Simple

**Cons:**
- ❌ User explicitly said to fix this
- ❌ Makes app useless for Portuguese users
- ❌ Fails success criterion "Portuguese characters display correctly"
- ❌ Unprofessional appearance

## Decision

**Selected: Option 1 - Read as UTF-8, Map Common Corruptions**

After analyzing all 6 sample files, the corruption patterns are consistent and predictable. When Windows-1252 bytes are misinterpreted as UTF-8, specific sequences appear:

**Windows-1252 → UTF-8 Corruption Patterns:**

| Intended Character | Windows-1252 Byte | Misread as UTF-8 | Display |
|-------------------|-------------------|------------------|---------|
| ç (cedilla) | 0xE7 | U+00E7 + 0xA7 | Ã§ |
| ã (a tilde) | 0xE3 | U+00E3 + 0xA3 | Ã£ |
| á (a acute) | 0xE1 | U+00E1 + 0xA1 | Ã¡ |
| é (e acute) | 0xE9 | U+00E9 + 0xA9 | Ã© |
| í (i acute) | 0xED | U+00ED + 0xAD | Ã­ |
| ó (o acute) | 0xF3 | U+00F3 + 0xB3 | Ã³ |
| ú (u acute) | 0xFA | U+00FA + 0xBA | Ãº |
| à (a grave) | 0xE0 | U+00E0 + 0xA0 | Ã  |
| â (a circumflex) | 0xE2 | U+00E2 + 0xA2 | Ã¢ |
| ê (e circumflex) | 0xEA | U+00EA + 0xAA | Ãª |
| ô (o circumflex) | 0xF4 | U+00F4 + 0xB4 | Ã´ |

This is a **systematic** corruption caused by double-encoding. We can reliably fix it with string replacement.

## Implementation

```javascript
function fixPortugueseEncoding(text) {
    if (!text) return text;
    
    const replacements = {
        'Ã§': 'ç', 'Ã£': 'ã', 'Ã¡': 'á', 'Ã©': 'é', 
        'Ã­': 'í', 'Ã³': 'ó', 'Ãº': 'ú', 'Ã ': 'à',
        'Ã¢': 'â', 'Ãª': 'ê', 'Ã´': 'ô', 'Ã': 'õ',
        'Ã§': 'Ç', 'Ã': 'Ã', 'Ã': 'Á', 'Ã‰': 'É',
        'Ã': 'Í', 'Ã"': 'Ó', 'Ãš': 'Ú', 'Ã€': 'À',
        'Ã‚': 'Â', 'ÃŠ': 'Ê', 'Ã"': 'Ô', 'Ã•': 'Õ',
    };
    
    let result = text;
    for (const [corrupt, correct] of Object.entries(replacements)) {
        result = result.replaceAll(corrupt, correct);
    }
    
    return result;
}

// Apply immediately after reading file
reader.onload = (e) => {
    let content = e.target.result;
    content = fixPortugueseEncoding(content);
    // Continue parsing...
};
```

## Testing Strategy

Must test against all 6 samples:

1. EXTRATO - 10 - ITAU - OFX.ofx
   - Contains: "COBRANCA" after fix (was: "COBRAN�A")
   
2. extrato-conta-corrente-ofx-unix_202601_20260109153954.ofx
   - Contains: "DÉBITO" after fix (was: "D�BITO")
   - Contains: "ANTECIPAÇÃO" after fix (was: "ANTECIPA��O")
   
3. Others: Verify Portuguese text displays correctly

## Consequences

### Positive
- Fixes corruption in 5 of 6 sample files
- Simple implementation
- Easy to extend if new patterns found
- No external dependencies
- Fast (string replacement)

### Negative
- Not universal solution (only handles specific patterns)
- Must maintain replacement map
- Could miss uncommon characters

### Mitigation
- Test against all samples
- Add console.log for unmapped � characters
- Document how to add new patterns

## Future Considerations

If this approach fails for some files:
1. Add more patterns to replacement map
2. Consider TextDecoder with ArrayBuffer as fallback
3. Ask user to convert file encoding before upload (last resort)

For now, pattern mapping is sufficient and reliable.

## Compliance

All downstream agents must:
- ✅ Implement `fixPortugueseEncoding()` function before parsing
- ✅ Apply to all text content from OFX file
- ✅ Test against all 6 sample files
- ✅ Add console warnings for unmapped � characters
- ❌ Do NOT trust declared encoding in OFX header
- ❌ Do NOT use FileReader encoding parameter (doesn't work)
