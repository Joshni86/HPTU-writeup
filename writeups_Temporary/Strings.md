# Strings

**Points**: 219  
**Solves**: 24  
**Flag Format**: HPTU{___}

## Challenge Description

A compact binary hides its secret in the decompiled code.

## Solution

1. **Initial Analysis**: The binary was analyzed using IDA Pro to examine the decompiled code.
2. **String Search**: Using IDA's strings window (Shift+F12), the flag was found directly in the decompiled source code.
3. **Flag Extraction**: The flag was visible as a hardcoded string in the binary's data section.
4. **Flag:** `HPTU{overflow_success}`
