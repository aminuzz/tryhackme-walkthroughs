# Pickle Rick 🥒 -- TryHackMe Writeup

**Difficulty:** Easy
**Category:** Web Exploitation / Linux Privilege Escalation

## Introduction
Pickle Rick is a Rick and Morty themed, easy-rated TryHackMe room built around a vulnerable web server. The goal is to perform web enumeration, credential discovery, and command execution to recover three hidden ingredients Rick needs to turn himself back into a human. Beneath the theming, the room tests a fairly standard chain: finding leaked credentials through source code and hidden files, using them to access a restricted command execution panel, and then escalating out of that restricted environment through a sudo misconfiguration to read a root-owned file.

This is what the website looks like:
<img width="1469" height="912" alt="image" src="https://github.com/user-attachments/assets/0cccff62-6f94-4bcf-a70f-6e7a89807d81" />

## Enumeration
### Source Code Review
The first step was to examine the page's source code, which revealed a comment containing a username:
<img width="1475" height="917" alt="image" src="https://github.com/user-attachments/assets/1e3cbca9-9a8f-479f-a389-1092a44d2cc2" />

```
R1ckRul3s
```

At this point, a valid username was known, but not where it was used or what the matching password was.

### Directory Enumeration
The next step was to look for hidden pages, such as a login portal, using `gobuster`:
```
gobuster dir -u "http://10.67.165.164" -x html,js,txt,php,py,css -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -t 64
```
![Gif21](https://github.com/user-attachments/assets/e1fa72f7-2a62-4a43-9e87-a8308d6bd82f)

This surfaced several files of interest:
- `portal.php` (redirects to `login.php`)
- `clue.txt`
- `robots.txt`

### Investigating clue.txt and robots.txt
`clue.txt` hinted that the remaining ingredients would need to be found by searching the file system, but didn't reveal credentials directly:
<img width="1471" height="916" alt="image" src="https://github.com/user-attachments/assets/d3f45876-f02d-4ae3-bbf7-9aedc8dc36ee" />

`robots.txt`, however, contained a single suspicious string:
<img width="1467" height="913" alt="image" src="https://github.com/user-attachments/assets/06564a88-81d4-4328-90b3-f072bdf206d4" />

```
Wubbalubbadubdub
```

Given the Rick and Morty theming, this was a strong candidate for a password.

## Gaining Access
### Background
With a username recovered from the page source and a likely password recovered from `robots.txt`, and a login page already identified via `gobuster`, the natural next step was to try the two together.

### Logging In
```
Username: R1ckRul3s
Password: Wubbalubbadubdub
```
![Gif22](https://github.com/user-attachments/assets/57a2440a-169a-440e-a450-a6ef0f6335df)

The login succeeded, revealing a command execution panel, a restricted shell interface connected to the web server rather than a full terminal.

### Bypassing the Command Restriction
Listing the current directory revealed a suspicious file:
<img width="1476" height="919" alt="image" src="https://github.com/user-attachments/assets/b249a089-4857-46a9-8c8b-52dc297996c9" />

```
Sup3rS3cretPickl3Ingred.txt
```

Attempting to read it with `cat` failed, the web application was blocking that specific command, a common but shallow form of command filtering in these kinds of restricted panels. Since the filter blocked `cat` by name rather than blocking file-reading behavior generally, an equivalent command, `less`, worked instead:
![Gif23](https://github.com/user-attachments/assets/a0d6213e-bc07-421d-b15b-42c51d82548f)

This revealed the **first ingredient**:
<img width="839" height="73" alt="image" src="https://github.com/user-attachments/assets/6b4afb74-89cb-4702-9ef8-a47520c6a47c" />

## Privilege Escalation
### Background
With command execution established, the next step was to search the rest of the file system for the remaining ingredients, as `clue.txt` had hinted. Since the server was running Apache, the web root was confirmed as the standard `/var/www/html` location:
```bash
pwd
```
<img width="1472" height="921" alt="image" src="https://github.com/user-attachments/assets/d67d9cfb-ab74-4271-a1c1-e2e16d7b023d" />

### Finding the Second Ingredient
Exploring outward from the root directory, `/home` was the first stop:
<img width="1474" height="914" alt="image" src="https://github.com/user-attachments/assets/373e4522-b5e7-4555-a6b7-adffe6b21aef" />

![Gif24](https://github.com/user-attachments/assets/c8741427-c85d-4789-8034-038ad02805e5)

Inside `rick`'s home directory, the **second ingredient** was located and read with `less`:
```
less /home/rick/"second ingredients"
```
<img width="843" height="77" alt="image" src="https://github.com/user-attachments/assets/634738fe-109b-4be2-8588-2b99262e52a0" />

### Escalating via Sudo to Read the Third Ingredient
The third ingredient was located in `/root`, a directory the current user has no inherent permission to read. Rather than a file-upload or kernel exploit, the escalation path here was a `sudo` misconfiguration: the restricted shell's user had passwordless `sudo` rights to run `less`. Since `less` can read any file regardless of ownership when invoked this way, that permission alone was enough to read directly out of root's home directory:
```
sudo less /root/3rd.txt
```
![Gif25](https://github.com/user-attachments/assets/d9ea451d-c911-44fb-beaf-50fd01e6ef93)

## Flag Capture
All three ingredients were recovered over the course of the room:
<img width="838" height="77" alt="image" src="https://github.com/user-attachments/assets/505b14bb-3114-4b7c-984b-9c9e10d3b217" />

## Lessons Learned
- **Source code is enumeration too.** The very first piece of the puzzle, the username, was sitting in an HTML comment, a reminder that reviewing page source should be a routine first step, not an afterthought.
- **Command blocklists are easy to bypass.** The panel blocked `cat` specifically, but not functionally equivalent commands like `less`. Filtering by command name rather than by capability is a shallow defense that rarely holds up.
- **Passwordless sudo on a single binary is still a full escalation path.** `less` isn't normally thought of as a dangerous binary, but sudo access to it is enough to read any file on the system as root, the same category of risk as the more commonly cited GTFOBins entries like `vim` or `find`.
- **Restricted shells are a speed bump, not a wall.** Once any command execution is achieved, the real question becomes what that execution context is allowed to do, not whether it looks like a full terminal.

## Tools Used
- `gobuster`
