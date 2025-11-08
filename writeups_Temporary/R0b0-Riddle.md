# R0b0 Riddle

## Challenge Information

| Property | Value |
|----------|-------|
| **Category** | Web |
| **Points** | 100 |
| **Solves** | 100 |
| **Flag Format** | `HPTU{_}` |

## Challenge Description

It is 2084. Robots rule the world, and humanity is in danger of going extinct. From the last resistance, you are the only hacker. One of the robot control servers has gone online. Your mission is to infiltrate it and recover the secret key that could save mankind. The fate of humanity depends on you.

---

## Initial Reconnaissance

Upon visiting the challenge URL, the page presents a futuristic-themed interface titled **"R0booooO roB0oo"** with the following features:

- Animated background with colorful gradient blobs
- Central robot image
- Text message: *"Welcome to the world of robots! Can you uncover their hidden secrets?"*
- Minimalist design with no obvious links or navigation elements

The page appears simple at first glance, but the challenge description hints at "infiltrating" the server and "recovering a secret key."

---

## Information Gathering

### Step 1: Checking the Browser Console

Opening the browser's Developer Tools (F12) and examining the Console tab reveals an interesting message:

```
Hint: Have you checked robots.txt for hidden paths?
```

This console message originates from a JavaScript file (`static/hidden.js`) and provides a crucial hint for the next step.

### Step 2: Understanding robots.txt

The `robots.txt` file is a standard protocol used by websites to communicate with web crawlers and search engine bots. It specifies which parts of the site should or shouldn't be crawled. While intended for legitimate purposes, security misconfigurations often result in sensitive paths being leaked through this file.

**Action:** Navigate to `http://[challenge-url]/robots.txt`

### Step 3: Analyzing robots.txt

The `robots.txt` file reveals several "Disallow" entries:

```
User-agent: *
Disallow: /dHJlYXN1cmUudHh0
Disallow: /c2VjcmV0LnR4dA==
Disallow: /Y2F0Y2hfbWVfaWZfdV9jYW4vZmxhZy50eHQ=
Disallow: /YWRtaW4udHh0
```

**Observation:** These paths appear unusual and don't follow typical URL patterns. The `==` padding at the end of some entries is a strong indicator of **Base64 encoding**.

---

## Decoding the Hidden Paths

### Decoding Process

Using a Base64 decoder (online tool or command line):

```bash
echo "dHJlYXN1cmUudHh0" | base64 -d
# Output: treasure.txt

echo "c2VjcmV0LnR4dA==" | base64 -d
# Output: secret.txt

echo "Y2F0Y2hfbWVfaWZfdV9jYW4vZmxhZy50eHQ=" | base64 -d
# Output: catch_me_if_u_can/flag.txt

echo "YWRtaW4udHh0" | base64 -d
# Output: admin.txt
```

### Decoded Paths

1. `/treasure.txt`
2. `/secret.txt`
3. `/catch_me_if_u_can/flag.txt`
4. `/admin.txt`

---

## Exploring the Discovered Paths

### Path 1: /treasure.txt

Navigating to `http://[challenge-url]/treasure.txt`:

```
Hidden Treasure!
Another flag? Nope, just a decoy. Look elsewhere!
```

**Result:** Decoy - no flag here.

### Path 2: /secret.txt

Navigating to `http://[challenge-url]/secret.txt`:

```
They secretly laugh when autocorrect ruins your text.
```

**Result:** Another decoy with a humorous message.

### Path 3: /admin.txt

Navigating to `http://[challenge-url]/admin.txt`:

```
[Decoy content]
```

**Result:** Still no flag.

### Path 4: /catch_me_if_u_can/flag.txt

Navigating to `http://[challenge-url]/catch_me_if_u_can/flag.txt`:

```
Here is the flag: HPTU{Robots_File_is_Hacked!}
```

**Result:** Flag successfully retrieved!

---

## Flag Retrieval

**Flag:** `HPTU{Robots_File_is_Hacked!}`

The flag was hidden in a subdirectory path (`/catch_me_if_u_can/flag.txt`) that was obfuscated using Base64 encoding in the `robots.txt` file.

---

## Solution Summary

### Attack Chain

1. **Console Inspection** - Visited the challenge page and found a hint in the browser console
2. **robots.txt Analysis** - Checked `/robots.txt` as suggested by the hint
3. **Pattern Recognition** - Identified Base64-encoded paths in the Disallow directives
4. **Decoding** - Decoded all Base64 strings to reveal actual file paths
5. **Path Enumeration** - Navigated through each decoded path
6. **Flag Extraction** - Located the flag in `/catch_me_if_u_can/flag.txt`

---

## Security Analysis

### Vulnerability Exploited

**Information Disclosure via robots.txt**

This challenge demonstrates a common security misconfiguration where sensitive paths are exposed in the `robots.txt` file. The vulnerability is compounded by:

- **Publicly Accessible File** - `robots.txt` is meant to be public and accessible to all
- **Security Through Obscurity** - Base64 encoding provides obfuscation, not security
- **Path Enumeration** - Disallowed paths create a roadmap for attackers
- **Lack of Access Controls** - No authentication required to access the hidden files

### What This Challenge Teaches

**Key Lessons:**

1. **Always check robots.txt** - This file often contains valuable information about a website's structure, including paths developers want to hide from search engines but inadvertently make discoverable to attackers

2. **Recognize Base64 encoding** - The `==` padding and alphanumeric patterns are telltale signs of Base64 encoding

3. **JavaScript console hints** - Developer comments and console.log statements can leak valuable information

4. **Security by obscurity fails** - Encoding paths in Base64 doesn't provide real security - it only provides obfuscation that's trivial to bypass

### Real-World Security Implications

**For Organizations:**

- **Don't rely on robots.txt for security** - This file is publicly accessible and should never contain sensitive paths
- **Avoid encoding sensitive information** - Base64 is encoding, not encryption - it provides zero security
- **Remove debug code** - Console hints and developer comments should be removed from production environments
- **Implement proper access controls** - Use authentication and authorization mechanisms, not obscure URLs
- **Follow principle of least privilege** - Sensitive files should not be accessible via direct URL

**Common Mistakes:**

- Using `robots.txt` to "hide" admin panels or sensitive directories
- Believing Base64 encoding provides any security benefit
- Leaving debug messages and hints in production code
- Not implementing authentication on sensitive endpoints

### Recommended Mitigations

1. **Access Control Implementation**
   - Implement authentication for all sensitive endpoints
   - Use authorization checks before serving sensitive content
   - Apply IP whitelisting for administrative interfaces

2. **robots.txt Best Practices**
   - Only use `robots.txt` for legitimate SEO purposes
   - Never include sensitive or administrative paths
   - Assume anything in `robots.txt` will be discovered

3. **Code Cleanup**
   - Remove all debug console messages from production
   - Strip developer comments and hints
   - Implement proper logging instead of console outputs

4. **Security Headers**
   - Implement proper HTTP security headers
   - Use Content Security Policy (CSP)
   - Enable HSTS for HTTPS enforcement

---

## Summary

**Challenge:** R0b0 Riddle  
**Difficulty:** Easy  
**Time to Solve:** 3-5 minutes  
**Tools Used:** Web Browser (Developer Tools), Base64 decoder  
**Flag:** `HPTU{Robots_File_is_Hacked!}`

**Key Skills Demonstrated:**
- Web reconnaissance techniques
- Understanding of `robots.txt` functionality
- Base64 encoding/decoding
- Path enumeration and discovery
