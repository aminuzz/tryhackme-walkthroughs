# Gaming Server -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** Web Enumeration / Linux Privilege Escalation

## Introduction
Gaming Server is an easy-rated TryHackMe room that moves from web enumeration to a full root compromise. It starts with directory fuzzing that exposes an SSH private key, which is passphrase-protected and cracked offline using a wordlist found on the site itself. After logging in with the recovered key, privilege escalation relies on the user's membership in the `lxd` group, which is abused to mount the host's filesystem inside a privileged container and read the root flag.

## Enumeration
### Nmap Scan
The first step was an in-depth `nmap` scan to identify open ports and service versions, running default scripts and scanning all ports:
<img width="936" height="315" alt="image" src="https://github.com/user-attachments/assets/6c19115c-17d0-4287-8b5e-a806bf55572d" />


### Web Enumeration
Next, `ffuf` was used to fuzz for hidden directories:
<img width="1094" height="690" alt="image" src="https://github.com/user-attachments/assets/bf19be64-181f-4963-99e5-546b2b3f83fd" />


Checking the `secret` directory revealed an SSH private key:
<img width="1910" height="915" alt="image" src="https://github.com/user-attachments/assets/c4247101-0696-4f7e-8a16-4be8f285ab7e" />


## Gaining Access
### Background
An SSH private key allows passwordless authentication, but this one was protected by a passphrase. A passphrase-protected key can still be attacked offline: `ssh2john` converts the key into a hash format that `john` can process, and `john` then brute-forces the passphrase against a wordlist. The wordlist used here was one found earlier in the site's `uploads` directory, a reminder that a target often provides the exact resources needed to attack it.

### Cracking the Key Passphrase
Using `ssh2john` to convert the key into a crackable format, then `john` with the wordlist from the `uploads` directory, recovered the passphrase:

<img width="746" height="197" alt="image" src="https://github.com/user-attachments/assets/f7e833ea-a8fc-46d3-ab67-3465825c1351" />
<img width="1913" height="919" alt="image" src="https://github.com/user-attachments/assets/a6ee20b8-36c0-46bd-80af-a54a51cd160d" />


The last piece was the correct username, which was identified after referencing a hint:
<img width="713" height="67" alt="image" src="https://github.com/user-attachments/assets/26966364-99eb-4ffd-b279-fdbb37a2ad2e" />


### Logging In
With the private key, the cracked passphrase, and the username `john`, SSH access was established:
<img width="685" height="428" alt="image" src="https://github.com/user-attachments/assets/4616b2e1-e368-4b52-aa2b-9f958b5a8230" />


The user flag was then read:
<img width="293" height="46" alt="image" src="https://github.com/user-attachments/assets/4c4704ba-eccb-4cca-b7c5-e27dd5f1a8d5" />


## Privilege Escalation
### How the Exploit Works: LXD Group Privilege Escalation
Checking the groups the user `john` belonged to showed membership in the `lxd` group:

<img width="852" height="37" alt="image" src="https://github.com/user-attachments/assets/f0f05196-def5-46ac-b5fd-e8b042bd453d" />

LXD is a container manager on Linux, essentially a lightweight virtual machine system built on Linux containers. A member of the `lxd` group can create and control containers, including configuring their privileges and mounting filesystems into them. That last capability is the vulnerability: by mounting the host's root filesystem into an unprivileged container that the attacker fully controls, the attacker can read and write any file on the host as root, since inside the container they are root and the mounted filesystem carries the host's real files. No kernel exploit is required; the group membership alone grants it.

### Exploiting the LXD Group
**Step 1** — Create the container image. A ready-made Alpine Linux image is commonly used because it is small:
<img width="1887" height="288" alt="image" src="https://github.com/user-attachments/assets/94954e43-8b2a-49a9-8b6b-e0f94d8cc1cc" />


**Step 2** — Transfer the image to the target machine (via HTTP or `scp`; `scp` was used here):
<img width="1882" height="130" alt="image" src="https://github.com/user-attachments/assets/9f2a90c4-b0d4-4971-9959-5b28a60a5388" />

**Step 3** — Import the image on the target machine:
<img width="806" height="36" alt="image" src="https://github.com/user-attachments/assets/eade4649-d07f-4406-9a57-738d932d155e" />


**Step 4** — Initialize the image from the `lxd` server:
<img width="590" height="44" alt="image" src="https://github.com/user-attachments/assets/d4b3c260-6017-417a-a9b5-eff2f503f365" />


**Step 5** — Mount the host's root directory inside the initialized container:
<img width="884" height="45" alt="image" src="https://github.com/user-attachments/assets/d7488f11-005e-4dad-849b-9b33e52a5428" />


**Step 6** — Start and enter the container. Because this is a minimal image, it has `/bin/sh` rather than `/bin/bash`:
<img width="357" height="38" alt="image" src="https://github.com/user-attachments/assets/c15adc2a-ecad-45b2-94ec-b73899b9245f" />


**Step 7** — Inside the elevated container, navigate into the mounted host root directory and read the flag:
<img width="1817" height="694" alt="image" src="https://github.com/user-attachments/assets/c0441a76-3f9d-42ed-b5ff-14ba744ff04a" />
<img width="1843" height="456" alt="image" src="https://github.com/user-attachments/assets/e627b3ed-2a0c-472a-b15b-926025701263" />


## Flag Capture
Both flags were recovered: the user flag after logging in as `john` with the cracked SSH key, and the root flag by abusing the `lxd` group membership to mount and read the host's root filesystem from inside a privileged container.

## Lessons Learned
- **Targets often hand you the tools to break them.** The wordlist that cracked the SSH key passphrase was sitting in the site's own `uploads` directory. Enumerate thoroughly before reaching for generic wordlists.
- **A passphrase-protected key is not a dead end.** `ssh2john` plus `john` turns an encrypted private key into an offline cracking problem, and a weak passphrase falls quickly.
- **Group membership is privilege.** The `lxd` group is as good as root. Checking a user's group memberships with `id` should be an early privilege-escalation step, since several groups (`lxd`, `docker`, `disk`) grant a direct path to root.
- **LXD privilege escalation is about mounting, not exploiting.** The technique works by mounting the host filesystem into a container the attacker controls, not by exploiting a software bug, which is why it is reliable and does not depend on a specific version.

## Tools Used
- `nmap`
- `ffuf`
- `ssh2john`
- `john`
- `ssh` / `scp`
- `lxd` / `lxc`
