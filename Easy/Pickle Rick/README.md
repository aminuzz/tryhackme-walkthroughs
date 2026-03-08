# Pickle Rick 🥒

This Rick and Morty themed CTF challenge requires exploiting a vulnerable web server to locate **three hidden ingredients** needed for Rick to transform himself back into a human.

The goal of the challenge is to perform **web enumeration, credential discovery, and command execution** on the target machine in order to retrieve the three ingredients.

This is what the website looks like:

<img width="1469" height="912" alt="image" src="https://github.com/user-attachments/assets/0cccff62-6f94-4bcf-a70f-6e7a89807d81" />

---

# Question 1
### What is the first ingredient that Rick needs?

I began by examining the **page source code** of the website and found an interesting note:

<img width="1475" height="917" alt="image" src="https://github.com/user-attachments/assets/1e3cbca9-9a8f-479f-a389-1092a44d2cc2" />

A comment in the source code revealed the username:
``R1ckRul3s``

At this point, we know a username exists, but we still do not know **where it is used or what the password is**.

My next step was to perform **directory enumeration** to identify hidden pages that might contain useful information such as login portals or credentials. I used **Gobuster** to scan the web server:

```
gobuster dir -u "http://10.67.165.164" -x html,js,txt,php,py,css -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -t 64
```
![Gif21](https://github.com/user-attachments/assets/e1fa72f7-2a62-4a43-9e87-a8308d6bd82f)

This scan discovered several interesting files:
- ``portal.php`` (redirected to ``login.php``)
- ``clue.txt``
- ``robots.txt``


### Investigating clue.txt
I first opened clue.txt in the browser to see if it contained any useful hints.

<img width="1471" height="916" alt="image" src="https://github.com/user-attachments/assets/d3f45876-f02d-4ae3-bbf7-9aedc8dc36ee" />

The file suggested searching through the file system to locate the remaining ingredients. Since this hint did not immediately reveal credentials, I moved on to examine ``robots.txt``.


### Investigating robots.txt
<img width="1467" height="913" alt="image" src="https://github.com/user-attachments/assets/06564a88-81d4-4328-90b3-f072bdf206d4" />

The file contained the string:
```
Wubbalubbadubdub
```
Given the context of the challenge, this appeared to be a potential password.

Since Gobuster had already revealed a login page (``portal.php`` → ``login.php``), I attempted to log in using:
```
Username: R1ckRul3s
Password: Wubbalubbadubdub
```


![Gif22](https://github.com/user-attachments/assets/57a2440a-169a-440e-a450-a6ef0f6335df)

### Accessing the command portal
After successfully logging in, I was presented with a command panel that allowed execution of commands on the server.

The interface essentially functioned as a restricted shell connected to the web server.

The first step was to list the files in the current directory:
<img width="1476" height="919" alt="image" src="https://github.com/user-attachments/assets/b249a089-4857-46a9-8c8b-52dc297996c9" />

One file immediately stood out:
```
Sup3rS3cretPickl3Ingred.txt
```
Attempting to open it using cat did not work because the command was disabled by the web application.

To bypass this restriction, I used the less command instead:

![Gif23](https://github.com/user-attachments/assets/a0d6213e-bc07-421d-b15b-42c51d82548f)

This revealed the **first ingredient**:
<img width="839" height="73" alt="image" src="https://github.com/user-attachments/assets/6b4afb74-89cb-4702-9ef8-a47520c6a47c" />


# Question 2
The hint in `clue.txt` mentioned that the remaining ingredients could be found by **searching the file system**.

Since the server was running **Apache**, the web root directory is typically:
```
/var/www/html
```


To confirm my current location, I used:

```bash
pwd
```
<img width="1472" height="921" alt="image" src="https://github.com/user-attachments/assets/d67d9cfb-ab74-4271-a1c1-e2e16d7b023d" />

After confirming the directory, I began exploring other directories on the system starting from the root (/):

<img width="1474" height="914" alt="image" src="https://github.com/user-attachments/assets/373e4522-b5e7-4555-a6b7-adffe6b21aef" />

The first directory I investigated was ``/home``:

![Gif24](https://github.com/user-attachments/assets/c8741427-c85d-4789-8034-038ad02805e5)

After navigating through the directory structure, I eventually located the **second ingredient**.
<img width="843" height="77" alt="image" src="https://github.com/user-attachments/assets/634738fe-109b-4be2-8588-2b99262e52a0" />


# Question 3
The **third ingredient** was located in the root directory.
![Gif25](https://github.com/user-attachments/assets/d9ea451d-c911-44fb-beaf-50fd01e6ef93)

<img width="838" height="77" alt="image" src="https://github.com/user-attachments/assets/505b14bb-3114-4b7c-984b-9c9e10d3b217" />


# My thoughts 🧠
This was a fun and beginner-friendly challenge that demonstrated several fundamental **web exploitation and enumeration techniques**.

Key concepts used in this challenge included:
- Source code analysis to discover hidden credentials
- Directory enumeration using Gobuster
- Investigating ``robots.txt`` and hidden files
- Exploiting a web-based command execution panel
- Basic Linux file system enumeration


Overall, this challenge provided a good introduction to web application enumeration and command execution vulnerabilities.
