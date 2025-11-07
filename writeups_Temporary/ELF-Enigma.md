# ELF Enigma

**Points**: 500  
**Solves**: 0  
**Flag Format**: HPTU{___}

## Challenge Description

A compact ELF binary hides its secret in the decompiled code; find the exact input that makes the program print "success".

## Solution

1. **Binary Analysis**: Used Ghidra to decompile the ELF binary and analyze the `main` function.
2. **Input Logic**: Discovered the program uses `scanf("DoYouEven%sCTF", buffer)` which expects input in a specific format.
3. **Password Extraction**: The actual comparison was against the string `"_init"`, but due to the scanf format, the full input needed to be `DoYouEven_initCTF`.
4. **Validation**: The program compares the extracted string (content between "DoYouEven" and "CTF") with `"_init"`.
5. **Flag:** `HPTU{_init}`

## Key Insight

The scanf format string "DoYouEven%sCTF" required wrapping the actual password _init between "DoYouEven" and "CTF".
