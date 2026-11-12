# U.A. High School -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** Web Exploitation / Steganography / Linux Privilege Escalation

## Introduction
U.A. High School is an easy-rated, My Hero Academia themed TryHackMe room that chains together several techniques. Recursive web enumeration uncovers a hidden page that acts as an unsanitized command shell, which is turned into a reverse shell. From there, a repaired image file hides credentials via steganography, and privilege escalation abuses a root-run script that passes user input to `eval`, allowing an arbitrary sudoers rule to be written and root to be obtained.

## Enumeration
### Nmap Scan
A thorough scan identified the open ports and services:

<!-- SCREENSHOT: nmap scan output -->

- Port 22 is open for SSH.
- Port 80 is open for HTTP.

### Web Enumeration
The website itself had nothing obviously useful on the front end:

<!-- SCREENSHOT: the website front page -->

Directory enumeration with `gobuster` found an `assets` directory, but it led nowhere on its own, and the contact page was a dead end too:

<!-- SCREENSHOT: gobuster results -->

Switching to `feroxbuster`, which enumerates recursively, surfaced a hidden page that a single-level scan would miss: `http://<target>/assets/index.php`:

<!-- SCREENSHOT: feroxbuster finding assets/index.php -->

### How the Exploit Works: Unsanitized Command Parameter
The discovered page rendered empty, with no obvious functionality:

<!-- SCREENSHOT: the empty index.php page -->

Parameter enumeration with `arjun` returned nothing:

<!-- SCREENSHOT: arjun returning no parameters -->

Testing the theory that the page might be a hidden command shell, a `cmd` parameter was supplied, and the page returned a Base64-encoded string, decoded with CyberChef:

<!-- SCREENSHOT: cmd parameter returning base64 output -->

This behavior comes from an unsanitized parameter: the `index.php` script takes the value of the `cmd` parameter and executes it through the shell, then Base64-encodes the output. Viewing the script's own source confirms this:

<!-- SCREENSHOT: index.php source -->

<!-- SCREENSHOT: CyberChef decoding of index.php source -->

Running `?cmd=ls` and decoding the result confirmed command execution:

<!-- SCREENSHOT: cmd=ls decoded output (1) -->

<!-- SCREENSHOT: cmd=ls decoded output (2) -->

## Gaining Access
### Reverse Shell
With confirmed command execution through the browser, a reverse shell was set up for easier interaction. An interactive Bash reverse shell failed, so an `nc` named-pipe (`mkfifo`) reverse shell was used instead, which worked. The shell was then stabilized:
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

<!-- SCREENSHOT: reverse shell established -->

### Finding and Repairing the Hidden Credentials
Exploring the file system turned up a directory of interest:

<!-- SCREENSHOT: directory of interest -->

<!-- SCREENSHOT: contents revealing a passphrase -->

- A passphrase was found: `AllmightForEver!!!` (likely an SSH-key or steganography passphrase).

<!-- SCREENSHOT: the passphrase -->

In the `images` directory were two images. Since `index.php` lived in `assets` alongside `images`, each image could simply be requested with `wget`:

<!-- SCREENSHOT: downloading the images with wget (1) -->

<!-- SCREENSHOT: downloading the images with wget (2) -->

### How the Exploit Works: File Signature Repair and Steganography
`oneforall.jpg` would not display, a sign its bytes had been tampered with. Inspecting it in a hex editor showed the file signature was set to a PNG (`89 50 4E 47`) rather than a JPEG. Correcting the leading bytes to a valid JPEG signature (`FF D8 FF E0 00 10 4A 46 49 46 00 01`) repaired the file so it would open:

<!-- SCREENSHOT: oneforall.jpg not displaying -->

<!-- SCREENSHOT: fixing the file signature in hexeditor -->

<!-- SCREENSHOT: the repaired image -->

With the image fixed, `steghide` extracted hidden data using the passphrase found earlier, writing it to `creds.txt`:

<!-- SCREENSHOT: steghide extracting the hidden data -->

<!-- SCREENSHOT: creds.txt contents -->

- This revealed the login for user `deku`: `One?For?All_!!one1/A`

The second image was noted for later:

<!-- SCREENSHOT: the second image -->

Logging in as `deku` and reading the user flag:

<!-- SCREENSHOT: logging in and reading the user flag -->

## Privilege Escalation
### How the Exploit Works: eval Command Injection in a Sudo Script
Checking the sudoers configuration showed that `deku` can run `/opt/NewComponent/feedback.sh` as root:

<!-- SCREENSHOT: sudoers entry for deku -->

Reading the script, it takes a feedback string, checks it against a blocklist of special characters (`` ` ``, `)`, `$(`, `|`, `;`, `?`, `!`, `\`), and, if none are present, passes the string to `eval` before appending it to a log file:

<!-- SCREENSHOT: feedback.sh source -->

The flaw is the `eval`: it executes the feedback string as a shell command. The blocklist is incomplete, it does not block the `>>` redirect or ordinary text, so a feedback string can append an arbitrary line to `/etc/sudoers`. Because the script runs as root, that write succeeds. Adding a rule that grants `deku` passwordless sudo makes it possible to switch to root:
```
deku NOPASSWD: ALL >> /etc/sudoers
```
(The `ALL=(ALL)` portion was omitted because its parentheses are on the blocklist, and only this machine needed to be targeted.)

### Exploiting It
Supplying the crafted feedback appended the sudoers rule through the root-run `eval`:

<!-- SCREENSHOT: injecting the sudoers rule via feedback.sh -->

With the new rule in place, running `su` under sudo granted a root shell and the flag:

<!-- SCREENSHOT: switching to root and reading the flag -->

## Flag Capture
Both flags were recovered: the user flag after logging in as `deku` with the steganography-recovered credentials, and the root flag after abusing the `eval` in the root-run `feedback.sh` to append a passwordless sudoers rule and switch to root.

## Lessons Learned
- **Recursive enumeration finds what shallow scans miss.** `gobuster` stalled at `assets`, but `feroxbuster` recursed into it and found `assets/index.php`, the entire foothold.
- **Unsanitized parameters are remote code execution.** A page that passes `cmd` straight to the shell is a web shell, whether intended or left in by mistake. Probing for hidden parameters is worth the effort.
- **A file that will not open is suspicious.** `oneforall.jpg` had a mismatched signature on purpose. Checking magic bytes in a hex editor and repairing them recovered an image that hid credentials.
- **`eval` on user input is dangerous even behind a blocklist.** The script's character filter missed the `>>` redirect, which was enough to write a sudoers rule. Blocklists are fragile; a root-run `eval` should never touch attacker-controlled input.

## Tools Used
- `nmap`
- `gobuster`
- `feroxbuster`
- `arjun`
- CyberChef
- `nc`
- `wget`
- `hexeditor`
- `steghide`
