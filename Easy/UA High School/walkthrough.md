# U.A. High School -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** Web Exploitation / Steganography / Linux Privilege Escalation

## Introduction
U.A. High School is an easy-rated, My Hero Academia themed TryHackMe room that chains together several techniques. Recursive web enumeration uncovers a hidden page that acts as an unsanitized command shell, which is turned into a reverse shell. From there, a repaired image file hides credentials via steganography, and privilege escalation abuses a root-run script that passes user input to `eval`, allowing an arbitrary sudoers rule to be written and root to be obtained.

## Enumeration
### Nmap Scan
A thorough scan identified the open ports and services:
<img width="800" height="317" alt="image" src="https://github.com/user-attachments/assets/74673a2f-7c58-48f5-91dc-6d276643f4fa" />

- Port 22 is open for SSH.
- Port 80 is open for HTTP.

### Web Enumeration
The website itself had nothing obviously useful on the front end:
<img width="1914" height="916" alt="image" src="https://github.com/user-attachments/assets/89da6cb5-6b5d-4067-994e-eb0789d04592" />

Directory enumeration with `gobuster` found an `assets` directory, but it led nowhere on its own, and the contact page was a dead end too:
<img width="1010" height="747" alt="image" src="https://github.com/user-attachments/assets/5b50a0e3-c47b-4372-9a9b-a813d1f0461e" />

Switching to `feroxbuster`, which enumerates recursively, surfaced a hidden page that a single-level scan would miss: `http://<target>/assets/index.php`:
<img width="951" height="635" alt="image" src="https://github.com/user-attachments/assets/47b363f4-6aa5-4c58-a255-33649f09dcd6" />


### How the Exploit Works: Unsanitized Command Parameter
The discovered page rendered empty, with no obvious functionality:
<img width="950" height="911" alt="image" src="https://github.com/user-attachments/assets/c3935b99-f37a-4fa5-b853-11abe8b22453" />


Parameter enumeration with `arjun` returned nothing:

<img width="504" height="216" alt="image" src="https://github.com/user-attachments/assets/9fecec4d-8eba-4700-b2c2-e508941dff80" />


Testing the theory that the page might be a hidden command shell, a `cmd` parameter was supplied, and the page returned a Base64-encoded string, decoded with CyberChef:
<img width="1913" height="951" alt="image" src="https://github.com/user-attachments/assets/0065e34b-f518-4357-829a-6cfc80c04d67" />


This behavior comes from an unsanitized parameter: the `index.php` script takes the value of the `cmd` parameter and executes it through the shell, then Base64-encodes the output. Viewing the script's own source confirms this:
<img width="1916" height="921" alt="image" src="https://github.com/user-attachments/assets/c53508ff-84b0-4c4b-8d19-d7494527e789" />
<img width="1917" height="940" alt="image" src="https://github.com/user-attachments/assets/53c66f32-65fa-4ef0-8056-5511306a06d7" />


Running `?cmd=ls` and decoding the result confirmed command execution:
<img width="1918" height="942" alt="image" src="https://github.com/user-attachments/assets/5eeae4df-0b73-48d0-adf7-e3b1540ad334" />
<img width="539" height="101" alt="image" src="https://github.com/user-attachments/assets/263a72c9-e7d6-4769-b018-7ea5b7f010ba" />


## Gaining Access
### Reverse Shell
With confirmed command execution through the browser, a reverse shell was set up for easier interaction. An interactive Bash reverse shell failed, so an `nc` named-pipe (`mkfifo`) reverse shell was used instead, which worked. The shell was then stabilized:
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
<img width="1898" height="907" alt="image" src="https://github.com/user-attachments/assets/1d2db16d-db01-4176-80af-c582ad0f8b74" />


### Finding and Repairing the Hidden Credentials
Exploring the file system turned up a directory of interest:
<img width="362" height="37" alt="image" src="https://github.com/user-attachments/assets/e5c7fa81-ebf4-4691-98cc-0c0cdadbbd57" />
<img width="558" height="34" alt="image" src="https://github.com/user-attachments/assets/2d3f2291-c5f5-4388-9583-9256a748a7a7" />


- A passphrase was found: `AllmightForEver!!!` (likely an SSH-key or steganography passphrase).
<img width="1912" height="933" alt="image" src="https://github.com/user-attachments/assets/9ed1b07f-9546-4957-a046-80b103a5e2e1" />

In the `images` directory were two images. Since `index.php` lived in `assets` alongside `images`, each image could simply be requested with `wget`:
<img width="528" height="40" alt="image" src="https://github.com/user-attachments/assets/b06b9aa9-9e52-4798-bb1d-ca47289bb129" />
<img width="1885" height="405" alt="image" src="https://github.com/user-attachments/assets/9ce3e501-2729-48a0-80f4-6e2732eb415a" />

### How the Exploit Works: File Signature Repair and Steganography
`oneforall.jpg` would not display, a sign its bytes had been tampered with. Inspecting it in a hex editor showed the file signature was set to a PNG (`89 50 4E 47`) rather than a JPEG. Correcting the leading bytes to a valid JPEG signature (`FF D8 FF E0 00 10 4A 46 49 46 00 01`) repaired the file so it would open:
<img width="1911" height="909" alt="image" src="https://github.com/user-attachments/assets/14738396-3b0a-4638-9273-ad4a02a1de92" />
<img width="1910" height="912" alt="image" src="https://github.com/user-attachments/assets/30658c30-f803-4bf1-ba46-083969c9ed14" />

With the image fixed, `steghide` extracted hidden data using the passphrase found earlier, writing it to `creds.txt`:

<img width="406" height="88" alt="image" src="https://github.com/user-attachments/assets/fa35897a-124e-497a-97bc-e5c85335f2a1" />

<img width="992" height="92" alt="image" src="https://github.com/user-attachments/assets/cf5f672f-7ab7-4025-811e-638154ef3f4c" />

- This revealed the login for user `deku`: `One?For?All_!!one1/A`

The second image was noted for later:
<img width="755" height="148" alt="image" src="https://github.com/user-attachments/assets/8bc1d32f-0cdc-444f-8fc3-93d2dbc5363a" />

Logging in as `deku` and reading the user flag:

<img width="332" height="103" alt="image" src="https://github.com/user-attachments/assets/9a39848a-4da3-4986-924c-5665cef9ba8c" />

## Privilege Escalation
### How the Exploit Works: eval Command Injection in a Sudo Script
Checking the sudoers configuration showed that `deku` can run `/opt/NewComponent/feedback.sh` as root:
<img width="953" height="100" alt="image" src="https://github.com/user-attachments/assets/d971ae88-58b9-43fe-83fd-c32e563b74bf" />

Reading the script, it takes a feedback string, checks it against a blocklist of special characters (`` ` ``, `)`, `$(`, `|`, `;`, `?`, `!`, `\`), and, if none are present, passes the string to `eval` before appending it to a log file:
<img width="1891" height="354" alt="image" src="https://github.com/user-attachments/assets/637727a3-52a8-434a-8e04-2632df482608" />

The flaw is the `eval`: it executes the feedback string as a shell command. The blocklist is incomplete, it does not block the `>>` redirect or ordinary text, so a feedback string can append an arbitrary line to `/etc/sudoers`. Because the script runs as root, that write succeeds. Adding a rule that grants `deku` passwordless sudo makes it possible to switch to root:
```
deku NOPASSWD: ALL >> /etc/sudoers
```
(The `ALL=(ALL)` portion was omitted because its parentheses are on the blocklist, and only this machine needed to be targeted.)

### Exploiting It
Supplying the crafted feedback appended the sudoers rule through the root-run `eval`:
<img width="575" height="151" alt="image" src="https://github.com/user-attachments/assets/31c0bfbf-765f-40bd-b448-2810c3da036a" />

With the new rule in place, running `su` under sudo granted a root shell and the flag:
<img width="668" height="332" alt="image" src="https://github.com/user-attachments/assets/e5e4960d-8931-4e6a-afdb-2971c2990a31" />

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
