# Robots

## Challenge Information

| Property | Value |
|----------|-------|
| **Category** | Web |
| **Points** | 100 |
| **Solves** | 83 |
| **Flag Format** | `HPTU{_}` |

## Challenge Description

A secret path is tucked away in the site's crawler instructions. Inspect the site root and any hidden/disallowed paths to uncover the flag. Do not interact with real users.

---

## Initial Reconnaissance

The challenge description provides several key hints:

- **"Crawler instructions"** - Refers to `robots.txt`, the file that guides web crawlers
- **"Site root"** - Suggests checking common root-level files
- **"Hidden/disallowed paths"** - Paths that are blocked from search engine indexing

This immediately points toward checking the **robots.txt file**.

---

## Understanding robots.txt

The `robots.txt` file is a standard used by websites to communicate with web crawlers and search engines. It specifies which parts of the site should or shouldn't be crawled.

### Why Security Practitioners Check robots.txt

1. **Reveals directory structures** - Shows internal organization
2. **Exposes sensitive paths** - Developers often "hide" paths from search engines
3. **Publicly accessible** - No authentication required
4. **Often overlooked** - Developers forget it's world-readable

---

## Solution Approach

### Step 1: Check robots.txt

Navigate to:
```
http://[challenge-url]/robots.txt
```

Examining the robots.txt file reveals disallowed paths or hidden endpoints:

```
User-agent: *
Disallow: /admin
Disallow: /secret
Disallow: /robo
Disallow: /hidden
```

### Step 2: Explore Discovered Paths

Based on the robots.txt file (or through common endpoint testing), a suspicious path is discovered: `/robo`

### Step 3: Access the Hidden Endpoint

Navigate to:
```
http://[challenge-url]/robo
```

The page loads successfully and displays in the main content area:

```
HPTU{SFBUVXtSMGIwdDVfNHIzX2g0Y2szZH0=}
```

**Observation:** This doesn't match the final flag format. The content inside the curly braces appears to be **Base64 encoded**.

---

## Decoding the Flag

### Recognizing Base64 Encoding

The string `SFBUVXtSMGIwdDVfNHIzX2g0Y2szZH0=` has telltale signs of Base64:

- Contains alphanumeric characters
- Ends with `=` (padding character)
- No special characters except the padding
- Length is a multiple of 4

### Decoding Process

**Using Command Line:**
```bash
echo "SFBUVXtSMGIwdDVfNHIzX2g0Y2szZH0=" | base64 -d
```

**Using Browser Console:**
```javascript
atob("SFBUVXtSMGIwdDVfNHIzX2g0Y2szZH0=")
```

**Using Online Tool:**
- Visit any Base64 decoder website
- Paste the encoded string
- Click decode

### Decoded Result

```
HPTU{R0b0t5_4r3_h4ck3d}
```

This matches the expected flag format!

---

## Flag Retrieval

**Flag:** `HPTU{R0b0t5_4r3_h4ck3d}`

The flag message "R0b0t5_4r3_h4ck3d" (Robots are hacked) is a clever reference to the robots.txt file used to discover it.

---

## Alternative Discovery Methods

If robots.txt didn't exist or was empty, the `/robo` endpoint could have been discovered through:

### Method 1: Directory Bruteforcing

**Using ffuf:**
```bash
ffuf -u http://[challenge-url]/FUZZ -w wordlist.txt
```

**Using gobuster:**
```bash
gobuster dir -u http://[challenge-url]/ -w /usr/share/wordlists/dirb/common.txt
```

**Using dirsearch:**
```bash
dirsearch -u http://[challenge-url]/ -w wordlist.txt
```

### Method 2: Common Endpoints Testing

Manually testing common paths related to the theme:

- `/robots` - Not found
- `/robot` - Not found
- `/robo` - Success!
- `/bot` - Not found
- `/crawler` - Not found

---

## Solution Summary

### Attack Chain

1. **Hint Analysis** - Read the challenge description and identified "crawler instructions" hint
2. **robots.txt Discovery** - Checked the `robots.txt` file at the site root
3. **Path Enumeration** - Discovered the `/robo` endpoint from robots.txt
4. **Endpoint Access** - Accessed the hidden endpoint and found encoded flag
5. **Encoding Recognition** - Recognized Base64 encoding pattern
6. **Decoding** - Decoded `SFBUVXtSMGIwdDVfNHIzX2g0Y2szZH0=` to get the final flag
7. **Flag Retrieval** - Retrieved flag: `HPTU{R0b0t5_4r3_h4ck3d}`

---

## Security Analysis

### Vulnerability: Information Disclosure

This challenge demonstrates **CWE-200: Exposure of Sensitive Information to an Unauthorized Actor** through:

- Publicly accessible hidden endpoints
- Lack of authentication on sensitive paths
- Reliance on "security through obscurity"

### Why This is a Problem

**robots.txt is NOT a security mechanism:**

- It's a guideline for well-behaved crawlers, not access control
- Any attacker can read it and discover "hidden" paths
- Never rely on robots.txt to protect sensitive content

**Encoding is NOT encryption:**

- Base64 is encoding, not encryption
- It provides zero security
- Anyone can decode Base64 instantly
- Never use encoding to "hide" sensitive data

---

## Security Best Practices

### 1. Proper Access Control

**Bad Practice:**
```
❌ Hiding admin panel in robots.txt
User-agent: *
Disallow: /admin
Disallow: /secret-panel
```

**Good Practice:**
```
✅ Implementing authentication
- Require login credentials
- Implement role-based access control (RBAC)
- Use session management
- Apply IP whitelisting for sensitive areas
```

### 2. Don't Expose Sensitive Paths

**Common Mistakes in robots.txt:**

```
# ❌ Don't do this:
User-agent: *
Disallow: /admin.php
Disallow: /config.php
Disallow: /database-backup.sql
Disallow: /secret-api/
Disallow: /.env
```

**Instead:**
```
# ✅ Do this:
# Don't list sensitive paths in robots.txt at all
# Use .htaccess, authentication, or IP restrictions
# Implement proper authorization checks
```

### 3. Implement Defense in Depth

**Multiple layers of security:**

1. **Authentication** - Verify user identity
2. **Authorization** - Check user permissions
3. **IP Whitelisting** - Restrict access by IP (for admin areas)
4. **Rate Limiting** - Prevent brute force attacks
5. **Monitoring & Alerting** - Detect suspicious access patterns
6. **WAF Rules** - Block common attack patterns

### 4. Secure Configuration Examples

**Apache .htaccess for admin directory:**
```apache
<Directory "/var/www/html/admin">
    AuthType Basic
    AuthName "Restricted Area"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
    
    # IP whitelist
    Order Deny,Allow
    Deny from all
    Allow from 192.168.1.0/24
</Directory>
```

**Nginx configuration:**
```nginx
location /admin {
    auth_basic "Restricted Area";
    auth_basic_user_file /etc/nginx/.htpasswd;
    
    # IP whitelist
    allow 192.168.1.0/24;
    deny all;
}
```

---

## Real-World Implications

### What Can Be Exposed

In production environments, this vulnerability pattern can expose:

- **Admin Panels** - Management interfaces without authentication
- **API Endpoints** - Private APIs that should require tokens
- **Backup Files** - Database dumps, code archives
- **Development/Staging** - Non-production environments
- **Configuration Files** - Settings and secrets
- **Database Interfaces** - phpMyAdmin, Adminer, etc.
- **Internal Documentation** - API docs, architecture diagrams
- **Source Code** - Exposed Git repositories

### Example Attack Scenarios

**Scenario 1: Exposed Admin Panel**
```
robots.txt reveals: Disallow: /admin-panel
Result: Attacker gains admin access without credentials
```

**Scenario 2: API Discovery**
```
robots.txt reveals: Disallow: /api/v2/internal
Result: Attacker discovers undocumented API endpoints
```

**Scenario 3: Backup File Exposure**
```
robots.txt reveals: Disallow: /backups/
Result: Attacker downloads database backups
```

---

## Reconnaissance Checklist

### Common Files to Check

When testing web applications, always check:

| File/Path | Purpose | What to Look For |
|-----------|---------|------------------|
| `/robots.txt` | Crawler directives | Hidden paths, admin panels |
| `/sitemap.xml` | Site structure | Complete URL listing |
| `/.git/` | Git repository | Source code exposure |
| `/backup/` | Backup files | Database dumps, archives |
| `/.env` | Environment variables | API keys, credentials |
| `/admin` | Admin panels | Management interfaces |
| `/api/` | API endpoints | Undocumented APIs |
| `/phpinfo.php` | Server info | Configuration details |
| `/.htaccess` | Apache config | Security rules |
| `/web.config` | IIS config | Application settings |

### Automated Reconnaissance Tools

**Web Crawlers:**
```bash
# Hakrawler
echo "http://target.com" | hakrawler

# Gospider
gospider -s http://target.com -d 2

# Katana
katana -u http://target.com
```

**Directory Enumeration:**
```bash
# Feroxbuster
feroxbuster -u http://target.com -w wordlist.txt

# ffuf
ffuf -u http://target.com/FUZZ -w wordlist.txt

# dirb
dirb http://target.com /usr/share/wordlists/dirb/common.txt
```

---

## Detection and Prevention

### How to Detect This Issue

**Manual Testing:**
1. Review robots.txt for sensitive path disclosures
2. Test all disallowed paths for accessibility
3. Check if authentication is required
4. Verify proper authorization checks

**Automated Scanning:**
```bash
# Using Nikto
nikto -h http://target.com

# Using OWASP ZAP
zap-cli quick-scan http://target.com

# Using Nuclei
nuclei -u http://target.com -t exposures/
```

### Prevention Measures

**1. Audit robots.txt regularly**
```bash
# Check what's in your robots.txt
curl http://your-site.com/robots.txt

# Review for sensitive path disclosures
grep -i "admin\|secret\|private\|internal" robots.txt
```

**2. Implement proper authentication**
```python
# Flask example
from flask import session, redirect

@app.route('/admin')
def admin_panel():
    if not session.get('is_admin'):
        return redirect('/login')
    return render_template('admin.html')
```

**3. Use security headers**
```
X-Robots-Tag: noindex, nofollow
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
```

---

## Summary

**Challenge:** Robots  
**Difficulty:** Easy  
**Time to Solve:** 2-3 minutes  
**Tools Used:** Web browser, Base64 decoder  
**Flag:** `HPTU{R0b0t5_4r3_h4ck3d}`

**Key Skills Demonstrated:**
- Understanding robots.txt functionality
- Information gathering and reconnaissance
- Base64 encoding/decoding
- Endpoint enumeration
- Web application security assessment

---

## Conclusion

This challenge demonstrates the critical security principle: **Security through obscurity is not security.**

**Key Lessons:**

1. **robots.txt is for crawlers, not security** - It's a suggestion, not access control
2. **Assume everything is discoverable** - Don't rely on hidden paths
3. **Implement proper authentication** - Every sensitive endpoint needs protection
4. **Encoding ≠ Encryption** - Base64 provides no security
5. **Defense in depth** - Multiple security layers are essential

The exposed path in robots.txt and the use of Base64 "obfuscation" demonstrate two common mistakes that can lead to information disclosure and unauthorized access in real-world applications.
