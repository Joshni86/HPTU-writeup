# HPTU CTF Writeups

This repository contains writeups for various challenges from the HPTU CTF event. Each challenge is documented in its own Markdown file within the `writeups/` directory.

## Challenges

### Forensics
- [Pig](writeups/Pig.md) - Coordinate plotting challenge
- [Disguised File](writeups/Disguised-File.md) - File signature analysis and steganography
- [Strings](writeups/Strings.md) - Binary analysis with IDA Pro
- [ELF Enigma](writeups/ELF-Enigma.md) - Reverse engineering ELF binary
- [Buried on-chain address](writeups/Buried-on-chain-address.md) - Smart contract forensic analysis

### OSINT
- [Sprawling Amusement Park](writeups/Sprawling-Amusement-Park.md) - Geolocation challenge
- [Altered Security](writeups/Altered-Security.md) - Forum post investigation

### Crypto
- [$1.5 Billion](writeups/1.5-Billion.md) - Cryptocurrency transaction analysis
- [Phonic Tonic](writeups/Phonic-Tonic.md) - Phone keypad cipher (T9)
- [Phonic Tonic (Detailed)](writeups/Phonic-Tonic-2.md) - Extended analysis of phone keypad cipher

### Web Security
- [Git is Fit](writeups/Git-is-Fit.md) - Exposed .git directory exploitation
- [Git is Fit (Alternative)](writeups/Git-is-Fit-2.md) - Source code reconstruction
- [Robots](writeups/Robots.md) - Robots.txt analysis
- [R0b0 Riddle](writeups/R0b0-Riddle.md) - Base64 encoded paths in robots.txt
- [LFI](writeups/LFI.md) - Local File Inclusion vulnerability
- [Dictionary Attack](writeups/Dictionary-Attack.md) - Brute force authentication
- [SQL Injection Authentication Bypass](writeups/SQL-Injection-Authentication-Bypass.md) - SQL injection in login
- [Privileged Cookies](writeups/Privileged-Cookies.md) - Session cookie manipulation
- [Upload and Conquer](writeups/Upload-and-Conquer.md) - File upload RCE

## Structure

Each writeup follows a consistent format:
- Challenge name and points/solves
- Brief description
- Step-by-step solution
- Key findings and vulnerabilities
- Flag (where applicable)

## Tools Used

- IDA Pro / Ghidra for binary analysis
- Git-dumper for repository extraction
- Binwalk for file analysis
- Steghide for steganography
- Various web exploitation tools

## Contributing

Feel free to contribute additional writeups or improvements to existing ones.

## Disclaimer

These writeups are for educational purposes only. Always follow ethical guidelines when participating in CTFs or security research.
