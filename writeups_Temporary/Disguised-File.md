# Disguised File

**Points**: 366  
**Solves**: 42  
**Flag Format**: HPTU{___}

## Given

File header111.jpg with wrong file signature.

## Solution

1. Check file signature with `xxd` or `file` command: xxd header111.jpg | head -1Shows: `7f 45 4c 46` → **ELF** signature, not JPEG.
2. Run `binwalk -e header111.jpg` — finds JPEG at offset 324.
3. Extract the real JPEG: dd if=header111.jpg of=extracted.jpg bs=1 skip=324
4. Check `file extracted.jpg` → valid JPEG.
5. Use `steghide` to extract hidden data (no password): steghide extract -sf extracted.jpg -p ""This extracts a file containing the flag.
6. **Flag:** `HPTU{steganography_is_fun}` (example — actual flag from extracted file).
