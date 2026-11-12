# Pickle Rick -- TryHackMe Writeup

**Difficulty:** Easy
**Category:** Web Exploitation / Linux Privilege Escalation

## Introduction
Pickle Rick is an easy-rated, Rick and Morty themed TryHackMe room built around a vulnerable web server. The objective is to recover three hidden ingredients Rick needs to turn himself back into a human, and doing so walks through a complete beginner attack chain: recovering leaked credentials from the page source and hidden files, using them to reach a restricted command execution panel, and finally escalating out of that restricted environment through a sudo misconfiguration to read a root-owned file.

This is what the website looks like:
<img width="1469" height="912" alt="image" src="https://github.com/user-attachments/assets/0cccff62-6f94-4bcf-a70f-6e7a89807d81" />

## Enumeration
### Source Code Review
The first step was to review the page's source code, which is a reliable early place to find hardcoded secrets, developer comments, or endpoints that aren't visible in the rendered page. A comment in the source exposed a username:
<img width="1475" height="917" alt="image" src="https://github.com/user-attachments/assets/1e3cbca9-9a8f-479f-a389-1092a44d2cc2" />

```
R1ckRul3s
```

At this stage a valid username was known, but not where it was used or what password went with it.

### Directory Enumeration
To find a login portal or other hidden content, the next step was directory enumeration with `gobuster`:
```
gobuster dir -u "http://10.67.165.164" -x html,js,txt,php,py,css -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -t 64
```
![Gif21](https://github.com/user-attachments/assets/e1fa72f7-2a62-4a43-9e87-a8308d6bd82f)

The scan surfaced several files of interest:
- `portal.php` (redirects to `login.php`)
- `clue.txt`
- `robots.txt`

### Investigating clue.txt and robots.txt
`clue.txt` hinted that the remaining ingredients would have to be found by searching the file system, but it offered no credentials directly:
<img width="1471" height="916" alt="image" src="https://github.com/user-attachments/assets/d3f45876-f02d-4ae3-bbf7-9aedc8dc36ee" />

`robots.txt` held a single, out-of-place string:
<img width="1467" height="913" alt="image" src="https://github.com/user-attachments/assets/06564a88-81d4-4328-90b3-f072bdf206d4" />

```
Wubbalubbadubdub
```

Given the theme of the room, this was a strong candidate for a password.

## Gaining Access
### Background
Two pieces were now in hand: a username pulled from the page source, and a likely password recovered from `robots.txt`. With a login page already identified during directory enumeration, the natural next step was to pair the two and authenticate.

### Logging In
```
Username: R1ckRul3s
Password: Wubbalubbadubdub
```
![Gif22](https://github.com/user-attachments/assets/57a2440a-169a-440e-a450-a6ef0f6335df)

The login succeeded and led to a command execution panel, a restricted shell interface wired to the web server rather than a full terminal.

### Bypassing the Command Restriction
Listing the current directory revealed a suspicious file:
<img width="1476" height="919" alt="image" src="https://github.com/user-attachments/assets/b249a089-4857-46a9-8c8b-52dc297996c9" />

```
Sup3rS3cretPickl3Ingred.txt
```

Reading it with `cat` failed because the web application was filtering that specific command. The filter blocked `cat` by name rather than blocking file-reading behavior in general, so a functionally equivalent command, `less`, worked without issue:
![Gif23](https://github.com/user-attachments/assets/a0d6213e-bc07-421d-b15b-42c51d82548f)

This exposed the first ingredient:
<img width="839" height="73" alt="image" src="https://github.com/user-attachments/assets/6b4afb74-89cb-4702-9ef8-a47520c6a47c" />

## Privilege Escalation
### Background
With command execution in place, the remaining ingredients had to be found by searching the file system, as `clue.txt` suggested. Since the server was running Apache, the web root was confirmed as the standard `/var/www/html`:
```
pwd
```
<img width="1472" height="921" alt="image" src="https://github.com/user-attachments/assets/d67d9cfb-ab74-4271-a1c1-e2e16d7b023d" />

### Finding the Second Ingredient
Exploring outward from the root directory, `/home` was the first place to check:
<img width="1474" height="914" alt="image" src="https://github.com/user-attachments/assets/373e4522-b5e7-4555-a6b7-adffe6b21aef" />

![Gif24](https://github.com/user-attachments/assets/c8741427-c85d-4789-8034-038ad02805e5)

Inside Rick's home directory, the second ingredient was located and read with `less`:
```
less /home/rick/"second ingredients"
```
<img width="843" height="77" alt="image" src="https://github.com/user-attachments/assets/634738fe-109b-4be2-8588-2b99262e52a0" />

### Escalating via Sudo to Read the Third Ingredient
The third ingredient was located in `/root`, a directory the current user has no inherent permission to read. The escalation path was a sudo misconfiguration: the restricted shell's user had passwordless sudo rights to run `less`. Because `less` can read any file regardless of ownership when invoked this way, that single permission was enough to read directly out of root's home directory:
```
sudo less /root/3rd.txt
```
![Gif25](https://github.com/user-attachments/assets/d9ea451d-c911-44fb-beaf-50fd01e6ef93)

## Flag Capture
All three ingredients were recovered over the course of the room:
<img width="838" height="77" alt="image" src="https://github.com/user-attachments/assets/505b14bb-3114-4b7c-984b-9c9e10d3b217" />

## Lessons Learned
- **Source code is enumeration too.** The first piece of the puzzle, the username, was sitting in an HTML comment. Reviewing page source should be a routine first step rather than an afterthought.
- **Command blocklists are easy to bypass.** The panel blocked `cat` specifically but not equivalent commands like `less`. Filtering by command name rather than by capability is a shallow defense that rarely holds.
- **Passwordless sudo on a single binary is still a full escalation path.** `less` is not normally thought of as dangerous, but sudo access to it is enough to read any file on the system as root, the same category of risk as more commonly cited GTFOBins entries like `vim` or `find`.
- **Restricted shells are a speed bump, not a wall.** Once any command execution is achieved, the real question is what that execution context is allowed to do, not whether it looks like a full terminal.

## Tools Used
- `gobuster`
