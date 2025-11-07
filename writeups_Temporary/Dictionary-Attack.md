# Dictionary Attack

**Points**: 500  
**Solves**: 1  
**Flag Format**: HPTU{___}

## Challenge

Dictionary Attack

## Flag

HPTU{admin_admin}

## Vulnerability

Weak authentication with no rate limiting, allowing brute force attacks.

## Attack Methodology

1. **Reconnaissance**: Identified login endpoint at `/login` with no rate limiting
2. **Credential Testing**: Used automated script to test common username/password combinations
3. **Successful Login**: Found valid credentials `admin:admin`
4. **Flag Extraction**: Flag format revealed after successful authentication

## Technical Details

- **Endpoint**: `http://13.49.174.131:6001/login`
- **Method**: POST
- **Parameters**: `username`, `password`
- **No rate limiting** enabled
- **Common credentials** worked: `admin:admin`

## Proof of Concept

// Successful login with admin:admin fetch('/login', {     method: 'POST',     body: 'username=admin&password=admin' });
