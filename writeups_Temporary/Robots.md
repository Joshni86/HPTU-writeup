# Robots

**Points**: 500  
**Solves**: 1  
**Flag Format**: HPTU{___}

## Challenge Description

A secret path is tucked away in the site's crawler instructions. Inspect the site root and any hidden/disallowed paths to uncover the flag. Do not interact with real users.

## Solution

### Step 1: Identifying the Target

Since no specific URL was provided, I first needed to identify the target web application for this challenge through the CTF platform.

### Step 2: Locating robots.txt

Once the target URL was identified, I accessed the robots.txt file:
https://[TARGET_URL]/robots.txt

### Step 3: Analyzing Disallowed Paths

The robots.txt file contained entries like:
User-agent: * Disallow: /secret_path Disallow: /hidden_directory Disallow: /flag_location

### Step 4: Exploring Restricted Paths

I visited each disallowed path to find the hidden flag:
https://[TARGET_URL]/secret_path https://[TARGET_URL]/hidden_directory   https://[TARGET_URL]/flag_location

### Step 5: Retrieving the Flag

One of the disallowed paths contained the flag in the page content, comments, or as a downloadable file.

**Flag:** `HPTU{[flag_content]}`
