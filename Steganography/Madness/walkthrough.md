# Madness -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** Steganography / Web Exploitation / Linux Privilege Escalation

Note: The privilege-escalation exploit used in this walkthrough was not written by me. It is the public GNU Screen 4.5.0 local privilege escalation exploit ([Exploit-DB 41154](https://www.exploit-db.com/exploits/41154)), and the script is included in this repository.

## Introduction
Madness is an easy-rated TryHackMe room that leans heavily on file analysis and puzzle-solving before any exploitation. A deliberately corrupted image hides a directory once its file signature is repaired, a secret URL parameter is brute-forced to reveal an encoded password, a ROT cipher yields a username, and a steganographic image gives up the login password. Privilege escalation then abuses a SUID `screen 4.5.0` binary using a well-known public exploit.

## Enumeration
### Nmap Scan
The first step was an `nmap` scan to find open ports and services:

<img width="797" height="292" alt="image" src="https://github.com/user-attachments/assets/08e1c05d-2192-4a52-b5d3-01ca0573692f" />

### Web Enumeration
The main web page contained something worth following up on:

<img width="429" height="129" alt="image" src="https://github.com/user-attachments/assets/dc71fdcd-33c8-4bb3-974f-b45122f9c620" />

Which led to this page:

<img width="1913" height="911" alt="image" src="https://github.com/user-attachments/assets/956a1499-fc4e-4819-85c5-0799847a4ce4" />

No image displayed on the page, so I used `curl` to pull it down and save the output:

<img width="933" height="455" alt="image" src="https://github.com/user-attachments/assets/f283eb4a-b94d-4239-be0e-4356e617d4b3" />

Trying to open the saved file produced an error, with the image viewer unable to render it correctly:

<img width="1918" height="916" alt="image" src="https://github.com/user-attachments/assets/21fe2e1c-c3fb-46d7-b149-0611f1536723" />

## Repairing the Corrupted Image
### How the Exploit Works: File Signatures
Running `binwalk` found nothing embedded in the file:

<img width="726" height="113" alt="image" src="https://github.com/user-attachments/assets/8308fd09-05d9-4a65-92f6-fa0cca1d8ebf" />

Running `strings` on it suggested, based on the first line (`$3br`), that the file was intended to be a JPEG but would not load, meaning it was corrupted somehow:

<img width="429" height="215" alt="image" src="https://github.com/user-attachments/assets/281b3c72-9e21-4d99-b47f-09af98485b47" />

A file's type is determined by its **magic bytes**, the first few bytes at the start of the file (its signature). Inspecting the hexdump revealed the problem:

<img width="571" height="208" alt="image" src="https://github.com/user-attachments/assets/8bbad8bd-14aa-4c1c-8bcd-5c5d305d874f" />

- The first bytes were `89 50 4E 47`, the signature for a **PNG**, yet the image would not display as one.
- Toward the end, `1414` was repeated and the PNG **IHDR chunk** (`49 48 44 52`) was missing, confirming the file had been altered on purpose.

Referencing the [List of file signatures](https://en.wikipedia.org/wiki/List_of_file_signatures), the file was actually a JPEG disguised as a corrupted PNG. Editing the leading bytes to a valid JPEG signature with `hexeditor` repaired it:

<img width="1912" height="883" alt="image" src="https://github.com/user-attachments/assets/8916c4cc-4dfc-42c3-a24c-f6af0ed0bd81" />

With the signature fixed, the image displayed and revealed a hidden directory, `/th1s_1s_h1dd3n`:

<img width="1910" height="917" alt="image" src="https://github.com/user-attachments/assets/5b3b8ae2-6fa1-4562-aaa7-46d85435886e" />

## Finding the Credentials
Visiting the hidden directory:

<img width="1916" height="916" alt="image" src="https://github.com/user-attachments/assets/b5076735-a600-4c7f-b538-e746fb4f5140" />

Checking the page source revealed a hint:

<img width="601" height="309" alt="image" src="https://github.com/user-attachments/assets/bba39f39-8f86-4449-a102-2abf3fbef036" />

A `gobuster` scan of the directory came up empty:

<img width="662" height="334" alt="image" src="https://github.com/user-attachments/assets/337b07d0-f410-4afb-a3e9-e0d7d5e9aac0" />

Testing the theory that the page expected a value, I supplied a `secret` parameter in the URL, which produced a response:

<img width="1919" height="921" alt="image" src="https://github.com/user-attachments/assets/43e7a1e3-a3bc-4313-b443-a7e95dadc6fd" />

Since the value needed to be brute-forced, `ffuf` was used to fuzz the `secret` parameter:

<img width="660" height="50" alt="image" src="https://github.com/user-attachments/assets/9c7bf4b3-9a6a-49d5-bbef-a2a68683b0fa" />

One response stood out from the rest:

<img width="1909" height="799" alt="image" src="https://github.com/user-attachments/assets/c76d8e68-eecc-48a8-a359-e979854542d0" />

Submitting that value returned the correct number along with an encoded string:

<img width="1915" height="916" alt="image" src="https://github.com/user-attachments/assets/c1eae960-4f45-4a75-82db-a9f81f280128" />

- The encoded text turned out to be a **password**, not the username I initially assumed.

Using the recovered value surfaced a username, but the TryHackMe hint noted it was "ROTten," meaning it was obscured with a ROT cipher. Decoding it revealed the real username:

<img width="661" height="422" alt="image" src="https://github.com/user-attachments/assets/a2a1beaa-03da-4b1d-b182-c34e288c5feb" />
<img width="1918" height="946" alt="image" src="https://github.com/user-attachments/assets/276a33b8-f7a0-40c3-99b8-f696a33663ca" />

### Recovering the Login Password via Steganography
The password found earlier did not work for login. The actual login password was hidden with steganography inside the image on the main page, extracted with `steghide`:

<img width="428" height="209" alt="image" src="https://github.com/user-attachments/assets/500e773b-9d92-470c-b514-2cd6c0899954" />

## Gaining Access
Logging in with the recovered username and password and reading the user flag:

<img width="479" height="136" alt="image" src="https://github.com/user-attachments/assets/092a12c0-4d55-4bcf-bc30-75aa8ed3f568" />

## Privilege Escalation
### How the Exploit Works: SUID screen 4.5.0
Enumerating SUID binaries on the machine:

<img width="565" height="313" alt="image" src="https://github.com/user-attachments/assets/4b2f2acc-92ff-43db-89db-ccc45c7e475f" />

The SUID bit causes a binary to run with the permissions of its owner (here, root) rather than the user running it. Cross-referencing the results against GTFOBins turned up nothing useful, except that the machine was running `screen` version 4.5.0. That version has a known local privilege-escalation vulnerability (**CVE-2017-5618**) that abuses its SUID-root `screen` binary: by exploiting how it handles logfile creation, an attacker can write a root-owned file or load a malicious shared library and execute code as root.

I downloaded the public exploit for this version, transferred the script to the target, and ran it, which returned a root shell and the flag:

<img width="982" height="548" alt="image" src="https://github.com/user-attachments/assets/d570058f-6252-48bb-9062-a246bf003fca" />

## Flag Capture
Both flags were recovered: the user flag after solving the image, parameter, cipher, and steganography chain to log in, and the root flag after exploiting the SUID `screen 4.5.0` binary.

## Lessons Learned
- **Always inspect images at the byte level.** A file that will not open is suspicious. Checking the hexdump with `xxd` and comparing against known file signatures revealed a JPEG disguised as a broken PNG.
- **A missing or wrong magic byte is often deliberate.** The altered signature and missing IHDR chunk were intentional, and repairing them unlocked the hidden directory.
- **When a directory scan fails, think about parameters.** `gobuster` found nothing, but a `secret` URL parameter was the real mechanism, and `ffuf` brute-forced its value.
- **Know your encodings and ciphers.** The challenge layered ROT and other obscuring on top of the credentials, so recognizing and reversing them was essential.
- **Check for odd SUID binaries and their versions.** A SUID `screen 4.5.0` was the entire path to root. Enumerating SUID binaries and looking up version-specific exploits is a core privesc habit.

## Tools Used
- `nmap`
- `curl`
- `binwalk`
- `strings`
- `hexeditor` / `xxd`
- `ffuf`
- `gobuster`
- **CyberChef**
- `steghide`
- **GNU Screen 4.5.0 exploit (Exploit-DB 41154)**
