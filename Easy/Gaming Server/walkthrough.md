# Gaming Server -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** Web Enumeration / Linux Privilege Escalation

## Introduction
Gaming Server is an easy-rated TryHackMe room that moves from web enumeration to a full root compromise. It starts with directory fuzzing that exposes an SSH private key, which is passphrase-protected and cracked offline using a wordlist found on the site itself. After logging in with the recovered key, privilege escalation relies on the user's membership in the `lxd` group, which is abused to mount the host's filesystem inside a privileged container and read the root flag.

## Enumeration
### Nmap Scan
The first step was an in-depth `nmap` scan to identify open ports and service versions, running default scripts and scanning all ports:

<!-- SCREENSHOT: nmap scan output -->

### Web Enumeration
Next, `ffuf` was used to fuzz for hidden directories:

<!-- SCREENSHOT: ffuf directory fuzzing results -->

Checking the `secret` directory revealed an SSH private key:

<!-- SCREENSHOT: secret directory containing the SSH private key -->

## Gaining Access
### Background
An SSH private key allows passwordless authentication, but this one was protected by a passphrase. A passphrase-protected key can still be attacked offline: `ssh2john` converts the key into a hash format that `john` can process, and `john` then brute-forces the passphrase against a wordlist. The wordlist used here was one found earlier in the site's `uploads` directory, a reminder that a target often provides the exact resources needed to attack it.

### Cracking the Key Passphrase
Using `ssh2john` to convert the key into a crackable format, then `john` with the wordlist from the `uploads` directory, recovered the passphrase:

<!-- SCREENSHOT: ssh2john and john cracking the passphrase -->

<!-- SCREENSHOT: wordlist found in the uploads directory -->

The last piece was the correct username, which was identified after referencing a hint:

<!-- SCREENSHOT: identifying the username -->

### Logging In
With the private key, the cracked passphrase, and the username `john`, SSH access was established:

<!-- SCREENSHOT: SSH login with the private key as john -->

The user flag was then read:

<!-- SCREENSHOT: reading user.txt -->

## Privilege Escalation
### How the Exploit Works: LXD Group Privilege Escalation
Checking the groups the user `john` belonged to showed membership in the `lxd` group:

<!-- SCREENSHOT: id / groups output showing lxd membership -->

LXD is a container manager on Linux, essentially a lightweight virtual machine system built on Linux containers. A member of the `lxd` group can create and control containers, including configuring their privileges and mounting filesystems into them. That last capability is the vulnerability: by mounting the host's root filesystem into an unprivileged container that the attacker fully controls, the attacker can read and write any file on the host as root, since inside the container they are root and the mounted filesystem carries the host's real files. No kernel exploit is required; the group membership alone grants it.

### Exploiting the LXD Group
**Step 1** — Create the container image. A ready-made Alpine Linux image is commonly used because it is small:

<!-- SCREENSHOT: building/obtaining the Alpine image -->

**Step 2** — Transfer the image to the target machine (via HTTP or `scp`; `scp` was used here):

<!-- SCREENSHOT: transferring the image with scp -->

**Step 3** — Import the image on the target machine:

<!-- SCREENSHOT: importing the image -->

**Step 4** — Initialize the image from the `lxd` server:

<!-- SCREENSHOT: initializing the image -->

**Step 5** — Mount the host's root directory inside the initialized container:

<!-- SCREENSHOT: mounting the host root directory into the container -->

**Step 6** — Start and enter the container. Because this is a minimal image, it has `/bin/sh` rather than `/bin/bash`:

<!-- SCREENSHOT: starting and entering the container -->

**Step 7** — Inside the elevated container, navigate into the mounted host root directory and read the flag:

<!-- SCREENSHOT: navigating the mounted host filesystem -->

<!-- SCREENSHOT: reading the root flag -->

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
