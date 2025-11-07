# Privileged Cookies

**Points**: 500  
**Solves**: 1  
**Flag Format**: HPTU{___}

## Challenge

Privileged Cookies

## Flag

HPTU{tgB1D!7R1lH&}

## Vulnerability

Insecure session management allowing cookie manipulation.

## Attack Methodology

1. **Session Analysis**: Examined current session cookie structure
2. **Cookie Manipulation**: Modified session data to elevate privileges
3. **Role Escalation**: Changed user role to "master_baker" or "admin"
4. **Admin Access**: Gained access to privileged admin console
5. **Flag Discovery**: Flag displayed after successful privilege escalation

## Technical Details

- **Cookie Structure**: Flask session cookie (header.payload.signature)
- **Manipulated Field**: Role/privilege level in session data
- **Technique**: Session variable manipulation
- **Result**: Unauthorized admin access

## Proof of Concept

// Cookie manipulation to gain admin access document.cookie = "session=eyJyb2xlIjoiYWRtaW4iLCJsb2dnZWRfaW4iOnRydWV9.[signature]";
