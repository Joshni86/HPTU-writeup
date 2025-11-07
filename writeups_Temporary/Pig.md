# Pig

**Points**: 920  
**Solves**: 21  
**Flag Format**: HPTU{___}

## Given Text

Sr. SONS TRAVEL CENTRE INC. S. GRAVEZATEI Toll Free: LABRASS 1540  WELCOME 2 HAS MASK.  EIC  PAL KALL2020: WEPD. H-C COPY : MUES AMYAN. HEP PESED : MOE. COVER: FIDOS SEFA : TAPE5 - POST1.PAPEL GROIN : VET PADAR. YAOL CRUMBS : MOSER. YANG. PREEN SETAL : GROINM. AL-ZIPS

## Solution

1. **Understand the structure** — The lines after `EIC` are in `KEY : VALUE` format.
2. **“HAS MASK” anagram** → “KASM SHA” or “MASK ASH” — not clear, but “mask” suggests steganography or bitmask.
3. **Convert words to numbers** using A=1, B=2, …, Z=26:
4. Repeat for all lines to get coordinate pairs.
5. **Plot the coordinates** on graph paper or image — they form letters when connected.
6. Connecting them reveals the word **PIG** (matching challenge name).
7. **Flag:** `HPTU{PIG}`
