# SQL Injection Authentication Bypass

**Points**: 500  
**Solves**: 1  
**Flag Format**: HPTU{___}

## Challenge

Authentication Bypass 0

## Flag

HPTU{yd9%QzZH9u}

## Vulnerability

SQL injection in authentication mechanism.

## Attack Methodology

1. **Input Testing**: Tested various SQL injection payloads in login form
2. **Bypass Discovery**: Used `admin' OR '1'='1` payload to bypass authentication
3. **Dashboard Access**: Successfully accessed admin dashboard
4. **Flag Retrieval**: Flag displayed on successful authentication page

## Technical Details

- **Vulnerable Parameter**: `username` field
- **Injection Payload**: `admin' OR '1'='1`
- **Result**: Always true condition bypassed authentication
- **Database Type**: Likely MySQL based on payload syntax

## Proof of Concept

-- Original query (assumed): SELECT * FROM users WHERE username='[input]' AND password='[input]'  -- Injected query: SELECT * FROM users WHERE username='admin' OR '1'='1' AND password='anything'
