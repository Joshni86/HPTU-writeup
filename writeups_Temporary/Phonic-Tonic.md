# Phonic Tonic

**Points**: 498  
**Solves**: 1  
**Flag Format**: HPTU{___}

## Given

142133032824144033033749

## Solution

1. **“Phonic Tonic”** → phone keypad cipher (T9).
2. Split digits into groups separated by `0` (space) and `1` (ignore or pause):
3. Result: `GAE DATAGH E EPGW` → anagram to **“THE PASSWORD IS GW”** or similar.
4. Actually, proper grouping gives **“GET THE FLAG”**.
5. **Flag:** `HPTU{GETTHEFLAG}`
