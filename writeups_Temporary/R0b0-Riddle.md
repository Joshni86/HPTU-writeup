# R0b0 Riddle

**Points**: 500  
**Solves**: 1  
**Flag Format**: HPTU{___}

## Challenge

R0b0 Riddle

## Flag

HPTU{r0b0ts_txt_s3cr3ts}

## Vulnerability

Information disclosure via robots.txt with Base64 encoded paths.

## Attack Methodology

1. **Robots.txt Analysis**: Checked `/robots.txt` for hidden paths
2. **Base64 Decoding**: Discovered paths were Base64 encoded
3. **Path Decryption**: Decoded all disallowed paths
4. **Hidden Content Access**: Accessed decoded paths to find flag
5. **Flag Extraction**: Found flag in one of the hidden paths

## Technical Details

- **Robots.txt Content**: Disallow: /dHJlYXN1cmUudHh0 Disallow: /c2VjcmV0LnR4dA== Disallow: /Y2F0Y2hfbWVfaWZfdV9jYW4vZmxhZy50eHQ= Disallow: /YWRtaW4udHh0
- **Decoded Paths**: 
- **Proof of Concept**: // Base64 decoding of hidden paths atob('Y2F0Y2hfbWVfaWZfdV9jYW4vZmxhZy50eHQ='); // Returns: "catch_me_if_u_can/flag.txt"
