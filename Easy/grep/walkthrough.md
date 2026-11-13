# grep -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** Web Exploitation / Source Code Review

## Introduction
grep is an easy-rated TryHackMe room centered on a custom content management system (SearchMeCMS) whose full source code is published on GitHub. The room rewards reading that source: the API key needed to register is recoverable from the repository's commit history, and an insecure file upload that validates only a file's magic bytes can be bypassed to gain a reverse shell. From there, a database backup and a public breach-lookup service lead to the admin credentials.

## Enumeration
### Nmap Scan
The first step was an `nmap` scan. The only immediately accessible page was the default Apache page, with nothing useful in its HTML:
<img width="1040" height="511" alt="image" src="https://github.com/user-attachments/assets/c307b4a0-f5f9-4e4c-9783-8c8992e07ce9" />

### Certificate Inspection
Inspecting the site's SSL/TLS certificate revealed two domains:

<img width="807" height="290" alt="image" src="https://github.com/user-attachments/assets/e158e386-042d-4f46-a68b-45d87af6aabb" />

<img width="806" height="300" alt="image" src="https://github.com/user-attachments/assets/ddd56fea-484f-4825-b6cf-b2bc89be8b9a" />

These domains were added to `/etc/hosts` so the local machine could resolve them:

<img width="359" height="28" alt="image" src="https://github.com/user-attachments/assets/b3e0aa4e-3cde-4bc4-8b33-7276d6c4f46c" />

### Finding the API Key
Attempting to register a new user on the site failed because the request requires a valid API key:

<img width="1917" height="876" alt="image" src="https://github.com/user-attachments/assets/6642ee29-f795-43a3-aa63-f5e284f1490c" />

A quick search led to the GitHub repository that powers the site. Reviewing the commits to `api/register` exposed the API key required for registration:

<img width="1908" height="945" alt="image" src="https://github.com/user-attachments/assets/75e4a36a-7ce2-4b5e-96f3-585ba5d076fd" />

## Gaining Access
### First Flag via API Key
Using Burp Suite to intercept the POST request to the API and substituting the recovered API key produced a successful login:

<img width="1053" height="762" alt="image" src="https://github.com/user-attachments/assets/c2b726ad-dab4-4e66-9730-c526cacdb022" />

<img width="1056" height="764" alt="image" src="https://github.com/user-attachments/assets/54feb2a1-e503-41da-8829-28eaf4649750" />

A successful login returned the first flag:

<img width="1908" height="883" alt="image" src="https://github.com/user-attachments/assets/65d69557-9b99-4b15-bcfa-b56a41a30f55" />

### How the Exploit Works: Magic-Byte Upload Bypass
Reviewing the same repository, the `api/uploads.php` file contained the upload logic:

<img width="1915" height="945" alt="image" src="https://github.com/user-attachments/assets/496b90e7-46fa-4974-be2f-f6f47975824b" />

The script only permits `jpg`, `jpeg`, `png`, and `bmp` files, but it validates the upload by checking its **magic bytes**, the first few bytes of a file that identify its type, rather than its actual content. This is the flaw: a PHP reverse shell can be prefixed with the magic bytes of an allowed image type, so the check sees a valid image while the server still executes the PHP inside. Editing the reverse shell's leading bytes with `hexeditor` to match a `.jpg` signature produces a file that passes validation but runs as code:

<img width="526" height="38" alt="image" src="https://github.com/user-attachments/assets/d510adf6-a2b3-4bdf-a8d9-dd6052ab4f81" />

### Uploading and Triggering the Shell
The crafted file was uploaded through `upload.php`:

<img width="893" height="158" alt="image" src="https://github.com/user-attachments/assets/508c3920-210f-40c5-afa6-81e35b27e6a9" />

The first attempt did not return a shell, because prepending the magic bytes had shifted the opening `<?php` tag and broken the script's execution. Moving the `<?php` tag below the inserted bytes (and re-adjusting the hex), and renaming the file to `shell.jpg.php` so the server would execute it as PHP, fixed it.

<img width="1910" height="887" alt="image" src="https://github.com/user-attachments/assets/4d7b78c9-9f4d-4268-ac39-9038b6a03afc" />

<img width="342" height="88" alt="image" src="https://github.com/user-attachments/assets/d9829d48-d0da-4bff-8f7e-22f8daacda40" />


With the shell working, enumeration of the file system turned up a SQL file in the `backup` folder containing information about the `admin` user:

<img width="609" height="325" alt="image" src="https://github.com/user-attachments/assets/1f53d0c1-de0b-4b08-95fe-12add0729151" />


<img width="1095" height="79" alt="image" src="https://github.com/user-attachments/assets/7ae565d1-e7a6-4519-851f-d68ea210e880" />

## Recovering the Admin Password
### Breach-Lookup Service
One of the answers points to a web application that lets a user check an email address for a password leak:

<img width="806" height="300" alt="image" src="https://github.com/user-attachments/assets/c023cd39-6e27-4269-95f2-cde1eca6557d" />

Visiting that service and checking the admin's email returned the admin user's password:

<img width="1910" height="880" alt="image" src="https://github.com/user-attachments/assets/d051560d-850a-46f4-8871-f864af43a069" />

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
