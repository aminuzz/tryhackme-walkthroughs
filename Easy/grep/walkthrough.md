# grep -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** Web Exploitation / Source Code Review

## Introduction
grep is an easy-rated TryHackMe room centered on a custom content management system (SearchMeCMS) whose full source code is published on GitHub. The room rewards reading that source: the API key needed to register is recoverable from the repository's commit history, and an insecure file upload that validates only a file's magic bytes can be bypassed to gain a reverse shell. From there, a database backup and a public breach-lookup service lead to the admin credentials.

## Enumeration
### Nmap Scan
The first step was an `nmap` scan. The only immediately accessible page was the default Apache page, with nothing useful in its HTML:

<!-- SCREENSHOT: nmap scan output -->

### Certificate Inspection
Inspecting the site's SSL/TLS certificate revealed two domains:

<!-- SCREENSHOT: certificate details showing domain 1 -->

<!-- SCREENSHOT: certificate details showing domain 2 -->

These domains were added to `/etc/hosts` so the local machine could resolve them:

<!-- SCREENSHOT: editing /etc/hosts with the discovered domains -->

### Finding the API Key
Attempting to register a new user on the site failed because the request requires a valid API key:

<!-- SCREENSHOT: failed registration due to missing/incorrect API key -->

A quick search led to the GitHub repository that powers the site. Reviewing the commits to `api/register` exposed the API key required for registration:

<!-- SCREENSHOT: API key found in the GitHub repository commit history -->

## Gaining Access
### First Flag via API Key
Using Burp Suite to intercept the POST request to the API and substituting the recovered API key produced a successful login:

<!-- SCREENSHOT: Burp Suite editing the API key in the POST request (1) -->

<!-- SCREENSHOT: Burp Suite editing the API key in the POST request (2) -->

A successful login returned the first flag:

<!-- SCREENSHOT: first flag after successful login -->

### How the Exploit Works: Magic-Byte Upload Bypass
Reviewing the same repository, the `api/uploads.php` file contained the upload logic:

<!-- SCREENSHOT: uploads.php source showing allowed extensions and magic-byte check -->

The script only permits `jpg`, `jpeg`, `png`, and `bmp` files, but it validates the upload by checking its **magic bytes**, the first few bytes of a file that identify its type, rather than its actual content. This is the flaw: a PHP reverse shell can be prefixed with the magic bytes of an allowed image type, so the check sees a valid image while the server still executes the PHP inside. Editing the reverse shell's leading bytes with `hexeditor` to match a `.jpg` signature produces a file that passes validation but runs as code:

<!-- SCREENSHOT: editing the reverse shell's magic bytes with hexeditor -->

### Uploading and Triggering the Shell
The crafted file was uploaded through `upload.php`:

<!-- SCREENSHOT: uploading the crafted file -->

The first attempt did not return a shell, because prepending the magic bytes had shifted the opening `<?php` tag and broken the script's execution. Moving the `<?php` tag below the inserted bytes (and re-adjusting the hex), and renaming the file to `shell.jpg.php` so the server would execute it as PHP, fixed it:

<!-- SCREENSHOT: reverse shell attempt (1) -->

<!-- SCREENSHOT: reverse shell attempt (2) -->

<!-- SCREENSHOT: corrected PHP tag placement -->

With the shell working, enumeration of the file system turned up a SQL file in the `backup` folder containing information about the `admin` user:

<!-- SCREENSHOT: reverse shell established -->

<!-- SCREENSHOT: SQL backup file revealing admin information -->

## Recovering the Admin Password
### Breach-Lookup Service
One of the answers points to a web application that lets a user check an email address for a password leak:

<!-- SCREENSHOT: breach-lookup service hostname -->

Visiting that service and checking the admin's email returned the admin user's password:

<!-- SCREENSHOT: admin password recovered from the breach-lookup service -->

## Flag Capture
The first flag was obtained after bypassing the API key check at registration, and the remaining answers, the admin email and password, were recovered from the database backup and the breach-lookup service after gaining a reverse shell through the magic-byte upload bypass.

## Lessons Learned
- **Open-source applications expose their own weaknesses.** The API key lived in the repository's commit history, and the upload flaw was visible in `uploads.php`. When source is available, read it first.
- **Commit history is part of the attack surface.** Secrets removed from current code often remain in earlier commits. Reviewing history recovered the registration API key.
- **Magic-byte validation is not content validation.** Checking only a file's leading bytes lets an attacker disguise executable code as an image. Proper upload handling must validate content and enforce a safe server-side extension.
- **Mind the payload structure after tampering.** Prepending magic bytes shifted the `<?php` tag and broke execution. Understanding why the shell failed, not just that it failed, was what led to the fix.

## Tools Used
- `nmap`
- Burp Suite
- `hexeditor`
- `nc`
