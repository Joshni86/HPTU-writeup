# Git is Fit

**Points**: 500  
**Solves**: 1  
**Flag Format**: HPTU{___}

## Challenge

Git is Fit

## Flag

HPTU{3a5Y_G1t_3xP10i7}

## Vulnerability

Exposed .git directory allowing source code reconstruction.

## Attack Methodology

1. **Git Discovery**: Found exposed `.git` directory at `/.git/`
2. **Source Reconstruction**: Used Git objects to reconstruct application source
3. **Credential Extraction**: Found hardcoded credentials in source code
4. **Admin Access**: Used credentials to login to admin panel
5. **Flag Retrieval**: Flag displayed in admin dashboard

## Technical Details

- **Exposed Path**: `/.git/`
- **Key Files Found**:  
- **Credentials**: Found in application source code

## Attack Steps

1. Access `/.git/` directory
2. Download Git objects
3. Reconstruct source code
4. Extract credentials
5. Login to admin panel
