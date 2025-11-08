# Privileged Cookies

## Challenge Information

| Property | Value |
|----------|-------|
| **Category** | Web |
| **Points** | 134 |
| **Solves** | 45 |
| **Flag Format** | `HPTU{_}` |

## Challenge Description

As a new customer, your regular account shows limited options. But rumors whisper of a secret "Master Baker" tier with exclusive recipes locked away in the vault. Your digital loyalty card, stored in the browser's kitchen.

Forge your identity to become the ultimate "Master Baker" and you'll gain the keys to the entire bakery, unlocking the vault and its hidden recipes.

---

## Initial Reconnaissance

Visiting the challenge URL presents **"Star Bakery"** - a beautifully designed bakery website featuring:

- Professional navbar with contact, track order, cart, and signup options
- Login page with bakery-themed design (light yellow cards)
- Marketing copy: "Freshly Baked, Made with Care!"
- Standard login/signup functionality

### Analyzing the Challenge Hints

The challenge description provides crucial clues:

- **"Digital loyalty card, stored in the browser's kitchen"** - clearly refers to cookies
- **"Master Baker tier"** - suggests role-based access control
- **"Forge your identity"** - implies manipulation of credentials
- **"Exclusive recipes locked away"** - indicates privileged content requiring access

---

## Creating an Account

First step involves creating a regular user account to explore the application:

1. Navigate to the signup page (`/signup`)
2. Create an account:
   - Username: `testuser`
   - Password: `password123`
3. Successfully registered and redirected to login

---

## Logging In and Initial Exploration

After logging in with the newly created account:

- Successfully authenticated
- Redirected to `/dashboard`

### Dashboard Observations

The dashboard displays:

- Welcome message with username
- **"Your current role: user"** ← Critical information!
- Order statistics (Total Orders: 0, Pending: 0, Completed: 0)
- Available bakery products with prices
- Regular products like cookies, pastries, cakes, muffins

The dashboard explicitly displays the role as **"user"**, suggesting there might be other roles like **"admin"** or **"Master Baker"**.

---

## Analyzing Browser Cookies

Opening the browser's Developer Tools (F12) and checking the Application/Storage tab under Cookies reveals:

```
username: testuser
role: user
```

**Critical Discovery:** The application stores the user's role in a client-side cookie!

This represents a significant security vulnerability - the application trusts client-side data for authorization decisions.

---

## Understanding the Vulnerability

### Insecure Authorization via Client-Side Cookies

The application makes authorization decisions based on a cookie value that the client controls. This violates a fundamental security principle: **Never trust client-side data for security decisions.**

### Key Observations

- The `role` cookie is set to `user` after login
- The dashboard displays different content based on this role
- There's likely an `/admin` endpoint that checks this cookie
- Authorization logic relies on untrusted client input

---

## Exploitation Strategy

Since cookies are stored client-side and fully controllable, the `role` cookie value can be modified from `user` to `admin` to gain elevated privileges.

### Method 1: Using Browser Developer Tools

1. Open Developer Tools (F12)
2. Navigate to **Application/Storage** → **Cookies**
3. Find the `role` cookie (currently set to `user`)
4. Double-click the value and change it to `admin`
5. Refresh the page or navigate to `/admin`

### Method 2: Using Browser Console

Execute JavaScript in the browser console:

```javascript
document.cookie = "role=admin; path=/";
```

Then refresh the page or navigate to `/admin`.

### Method 3: Using Burp Suite

1. Intercept a request to `/dashboard` or `/admin`
2. Modify the Cookie header:
   ```
   Cookie: username=testuser; role=admin
   ```
3. Forward the request

---

## Accessing the Admin Panel

After changing the `role` cookie to `admin`, navigating to `/admin` reveals:

### Admin Panel Features

- Welcome message: *"Welcome, testuser — Now, you have the admin access."*
- Secret Flag box prominently displayed at the top
- Special Products section showing privileged recipes:
  - Secret Chocolate Truffle Cake (₹999)
  - Special Truffle (₹499)
  - Golden Heritage Bread (₹399)
- Order statistics (Total Orders: 128, Pending: 34, Completed: 94)

---

## Flag Retrieval

The admin panel displays the secret flag:

**Flag:** `HPTU{tgB1D!7R1lH&}`

---

## Alternative Discovery Methods

If the role hadn't been displayed on the dashboard, the vulnerability could have been discovered through:

1. **Directory Enumeration** - Trying common admin paths like `/admin`, `/administrator`, `/panel`
2. **Cookie Analysis** - Noticing the suspicious `role` cookie during inspection
3. **Error Messages** - The forbidden page mentions "Admin role required: Take the permission before entering"
4. **Fuzzing** - Testing different cookie values to observe behavior changes

---

## Solution Summary

### Attack Chain

1. **Account Creation** - Created a regular user account via `/signup`
2. **Authentication** - Logged in and was assigned `role=user` cookie
3. **Discovery** - Identified the application uses client-side cookies for authorization
4. **Exploitation** - Modified the `role` cookie value from `user` to `admin`
5. **Access** - Accessed the `/admin` endpoint with elevated privileges
6. **Flag Extraction** - Retrieved the flag from the admin panel

---

## Security Analysis

### Vulnerability Classification

This challenge demonstrates:
- **CWE-639:** Authorization Bypass Through User-Controlled Key
- **CWE-807:** Reliance on Untrusted Inputs in a Security Decision

### What Went Wrong

1. **Trust in Client-Side Data** - The application trusts the `role` cookie value sent by the client
2. **No Server-Side Validation** - Authorization decisions are made without verifying against server-side session data
3. **Predictable Cookie Values** - The role values (`user`, `admin`) are obvious and easily guessable
4. **No Integrity Protection** - Cookies lack cryptographic signatures or encryption

### Real-World Impact

In a real bakery/e-commerce application, this vulnerability could allow:

- **Unauthorized Admin Access** - Access to admin panels and management functions
- **Price Manipulation** - Modifying product prices or discounts
- **Customer Data Access** - Viewing customer information and order details
- **Financial Fraud** - Manipulating orders and payment information
- **Business Logic Bypass** - Accessing premium features without payment
- **Data Breach** - Extracting sensitive business and customer data

---

## Security Best Practices

### 1. Server-Side Session Management

**Vulnerable Code:**
```python
# BAD (vulnerable):
role = request.cookies.get('role')
if role == 'admin':
    return admin_panel()
```

**Secure Code:**
```python
# GOOD (secure):
user = get_user_from_session(session['user_id'])
if user.role == 'admin':
    return admin_panel()
```

### 2. Never Trust Client Data

All authorization decisions must be made server-side using trusted data sources:

- Store user roles in a database
- Verify roles on every privileged request
- Use server-side sessions, not client-side cookies

### 3. Use Signed/Encrypted Sessions

Implement secure session management:

- **Flask's secure session cookies** - Cryptographically signed
- **JWT with proper verification** - Token-based authentication
- **Server-side session storage** - Redis, Memcached, or database

### 4. Implement Role-Based Access Control (RBAC)

```python
# Secure authorization example
@app.route('/admin')
def admin():
    # Get user_id from cryptographically signed session
    user_id = session.get('user_id')
    if not user_id:
        return redirect('/login')
    
    # Query database for user's actual role
    user = db.query("SELECT role FROM users WHERE id = ?", user_id)
    
    # Make authorization decision based on database
    if user.role != 'admin':
        return "Access Denied", 403
    
    return render_admin_panel()
```

### 5. Cookie Security Flags

Set appropriate security flags on cookies:

```python
response.set_cookie('session', value, 
    httponly=True,     # Prevent JavaScript access
    secure=True,       # HTTPS only
    samesite='Strict'  # CSRF protection
)
```

### 6. Audit Logging

- Log all access attempts to privileged endpoints
- Monitor for suspicious authorization patterns
- Implement alerting for unauthorized access attempts

---

## Why This Attack Works

The HTTP protocol is stateless, and cookies are the primary mechanism for maintaining state. However:

- Cookies are stored client-side and fully controlled by the user
- Users can read, modify, and delete any cookies
- Browser developer tools make cookie manipulation trivial
- Any security decision based solely on cookies is fundamentally flawed

**Key Principle:** Client-side data should **never** be trusted for security decisions.

---

## Detection and Prevention

### How to Detect This Vulnerability

- **Code Review** - Look for authorization logic using cookies directly
- **Pattern Matching** - Search for `request.cookies` in authorization code
- **Manual Testing** - Modify cookies and observe behavior changes
- **Security Scanners** - Use OWASP ZAP or Burp Suite for automated detection
- **Penetration Testing** - Regular security assessments

### Prevention Checklist

- [ ] Store roles in database, not cookies
- [ ] Use server-side sessions for authentication state
- [ ] Implement cryptographically signed session cookies
- [ ] Validate all authorization decisions server-side
- [ ] Apply principle of least privilege
- [ ] Set secure cookie flags (HttpOnly, Secure, SameSite)
- [ ] Implement comprehensive audit logging
- [ ] Regular security code reviews
- [ ] Penetration testing before production deployment

---

## Summary

**Challenge:** Privileged Cookies  
**Difficulty:** Medium  
**Time to Solve:** 3-5 minutes  
**Tools Used:** Browser Developer Tools (Cookie inspection)  
**Flag:** `HPTU{tgB1D!7R1lH&}`

**Key Skills Demonstrated:**
- Cookie analysis and manipulation
- Understanding client-side vs server-side security
- Authorization testing and bypass techniques
- Web application security assessment
