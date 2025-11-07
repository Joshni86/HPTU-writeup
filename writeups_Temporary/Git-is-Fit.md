# Git is Fit

**Points**: 500  
**Solves**: 1  
**Flag Format**: HPTU{__}

## Challenge Description

Are you able to takeover Admin Console?

## URLs

- http://13.49.174.131:6017/
- http://13.50.143.225:6017/
- http://16.170.145.145:6017/
- http://51.21.12.121:6017/

## Solution

### Step 1: Discovery of Exposed .git Directory

I started by exploring the web application and discovered an exposed .git directory at:
http://13.50.143.225:6017/.git/

### Step 2: Repository Dumping

Using git-dumper, I downloaded the entire Git repository:
git-dumper http://13.50.143.225:6017/.git/ git_repo

### Step 3: Analyzing Commit History

After examining the commit history, I found critical information about credential evolution:
cd git_repo git log --oneline

Commit History Found:
- `5052cc0` Initial secure authentication implementation
- `b71ac97` Add session management and update credentials
- `8c47272` Temporarily simplify credentials for development testing - REVERT BEFORE PROD
- `5abcdf6` Add news categories and article management
- `99c8d89` Security hardening: Restore production credentials and add 2FA placeholder

### Step 4: Extracting Development Credentials

I examined the commit where credentials were simplified:
git show 8c47272

Found Credentials:
- **Username:** `admin`
- **Password:** `admin.admin.admin@123`

### Step 5: Gaining Access

Using the discovered credentials, I logged into the admin panel:
URL: http://13.50.143.225:6017/admin Username: admin Password: admin.admin.admin@123

### Step 6: Retrieving the Flag

After successful authentication, I accessed the admin dashboard where the flag was displayed.

**Flag:** `HPTU{1H4#kNO2OwT}`

## Vulnerability Exploited

- **Exposed .git Directory**: Allowed access to complete version control history
- **Hardcoded Credentials in Version Control**: Sensitive credentials were committed to Git
- **Insecure Development Practices**: Temporary credentials were never properly removed

## Security Lessons

1. Never expose version control directories in production environments
2. Avoid committing sensitive credentials to repositories
3. Implement proper credential rotation and management
4. Use environment variables for configuration instead of hardcoded values
