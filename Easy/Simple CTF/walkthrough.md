# Simple CTF
This TryHackMe room focuses on basic web exploitation and privilege escalation techniques in a beginner-friendly environment. The challenge starts with reconnaissance using tools like `nmap` to identify open ports and services running on the target machine. From there, you investigate the CMS in use, determine which known CVE it is vulnerable to, and exploit the vulnerability to gain initial access. After establishing a foothold, the room guides you through simple enumeration and privilege escalation steps, including leveraging `vim` to obtain elevated privileges, while answering questions along the way to reinforce the concepts being practiced.


## Question 1 🔍
- How many services are running under port 1000?

I ran a simple `nmap` scan to scan all ports under port 1000:
<img width="1541" height="910" alt="Screenshot_2026-05-21_16-59-41" src="https://github.com/user-attachments/assets/d6a41861-fbc9-4671-b177-30de28e85d49" />
- This `nmap` scan was a deep scan which picked up the following:
  - `ftp` running on port 21 specifically vsftpd 3.0.3
  - `http` running on port 80 specifically Apache httpd 2.4.18
  - `ssh` wasn't found under port 22 meaning that it most likely is running on port 2222 (the most common alternative)
 
The answer for Question 1 is 2 services being `ftp` and `http`.


## Question 2
- What is running on the higher port?

Based on our second scan I found that `ssh` is running on an alternative port being port 2222 which is confirmed in the screenshot below:
<img width="1364" height="915" alt="Screenshot_2026-05-21_17-04-53" src="https://github.com/user-attachments/assets/075f8af5-0004-494e-8170-8d8e7691dfc6" />


## Question 3,4,5 🔓
- What's the CVE you're using against the application?
- To what kind of vulnerability is the application vulnerable?
- What's the password?

Before determining what CVE this application is vulnerable to, I checked out the webpage that this machine is hosting by using `ffuf` to scan for hidden directories
since based on the `nmap` scan, only the Apache default webpage can be seen:

<img width="1912" height="765" alt="ffuf_scan" src="https://github.com/user-attachments/assets/7c2151bc-6478-4801-b580-1e1334c7a6f5" />

- `index.html` is the default Apache web page
- `robots.txt` is a file that manages what webpages bots/web crawlers can access. Looking into it I noticed `Disallow: /openemr-5_0_1_3` (something of interest since `openemr` is an open source electronic health and management application which could contain a vulnerability)
- `simple` takes us directly to the main page of the **CMS Made Simple** webpage (our attack vector)


Looking through the page, nothing stood out to me until I scrolled all the way down and found a version number that this CMS software is using:
- **CMS Made Simple** is a open source PHP based content management system that uses MySQL to make website development and management easy by allowing for customization through themes, modules, etc.It allows a user to make changes to the website, manage images and files, control layouts all through a admin page.

<img width="1240" height="317" alt="Screenshot 2026-05-21 at 17-34-12 Home - Pentest it" src="https://github.com/user-attachments/assets/e5cb87a2-eb08-4f46-88b9-a149b30ab24f" />


I then referenced **Exploit-DB** to see if there was an exploit for this particular version of **CMS Made Simple**:
<img width="1909" height="236" alt="searchsploit_scan" src="https://github.com/user-attachments/assets/1b61d9a2-95bc-433c-9519-e36c9825439b" />
- And voila there was an exploit for this particular version of the software specifically a **SQL Injection**

I copied the exploit to my machine but before I ran the exploit I did a quick google search on what it does:
<img width="542" height="338" alt="Screenshot 2026-05-21 at 17-55-01 cms made simple sql injection exploit 2 2 8 - Google Search" src="https://github.com/user-attachments/assets/687d0eb0-7585-4910-b6d4-963aca6765f9" />

Based on the description, this is a **time based SQL injection attack** meaning the exploit utilizies injecting SQL syntax
along with a conditional time delay at the end of the query. By measuring the server's response time this allows a attacker to infer the character's of the administrator credentials or password reset hashes.

With this in mind, the next step was to run the exploit:
<img width="756" height="199" alt="exploit_success" src="https://github.com/user-attachments/assets/a53b8297-8f4a-471f-8da1-a55a4e9a7a07" />

This sums up all of the answers for questions 3 through 5:
- CVE-2019-9053
- sqli
- secret

### Common issues
Some issues that I came across were the following:

- The exploit contains Python2 code so it's up to you if you want to run the exploit with Python2 and make the necessary changes. In my case, I found it easier making the changes to reflect Python3 syntax since that is what I am accustomed to.


- Also when choosing a wordlist for the exploit to use, it's important to make sure the wordlist that you are using is `utf-8` encoded otherwise you'll run into an error where the exploit won't be able to decode the password.
<img width="938" height="535" alt="Screenshot_2026-05-30_18-14-54" src="https://github.com/user-attachments/assets/e96a3156-4121-4d7e-8012-08f25bdd378a" />


## Questions 6,7,8 👨‍💻
- Where can you login with the details obtained?
- What's the user flag?
- Is there any other user in the home directory? What's its name?

From the output of the exploit we have a username and a password:
- `mitch`
- `secret`

After logging in, I found the user flag:
<img width="551" height="200" alt="Screenshot_2026-05-30_18-25-48" src="https://github.com/user-attachments/assets/bfec6ffd-052d-4364-b0a8-79dbf9772571" />

After moving out of the `mitch` user's home directory, I found another home directory with the name `sunbath`:
<img width="883" height="382" alt="Screenshot_2026-05-30_18-28-30" src="https://github.com/user-attachments/assets/30244561-8136-48a9-8344-5133f4c3273e" />



## Questions 9 & 10 🏁
- What can you leverage to spawn a privileged shell?
- What's the root flag?

Before getting into elevating our privileges, I ran the following command to stabilize my shell:
<img width="782" height="90" alt="Screenshot_2026-05-30_18-34-39" src="https://github.com/user-attachments/assets/3e80918f-9e6b-4977-af42-20bd6aa2861e" />

With the shell cleaned up, now I had to find a way to elevate my privileges. Since we know the password of the user `mitch`, I checked the sudoers file to see what they could run with sudo privileges:
<img width="862" height="94" alt="Screenshot_2026-05-30_18-37-16" src="https://github.com/user-attachments/assets/914d9526-335b-4603-814a-893d2069e4e2" />

The user `mitch` can run the `vim` binary as the `root` user without a password meaning that every time `vim` is used it's operating with elevated privileges. `vim` itself is a text editor however it has a feature where it allows the user to run shell commands. 





Here's the exploit in action:
- Create a file with `vim` and give it a name
<img width="609" height="46" alt="Screenshot_2026-05-30_18-51-41" src="https://github.com/user-attachments/assets/267755d3-3602-49fa-abc6-32804aaa102f" />

- Once inside `vim`, make sure you're in **normal** mode and type the following. This will spawn a root shell since we are running `vim` with root privileges
<img width="935" height="798" alt="Screenshot_2026-05-30_18-52-15" src="https://github.com/user-attachments/assets/e052c4c7-42f2-48fb-b232-b73ea855712c" />


After gaining access to the root shell, I found the root flag:
<img width="544" height="145" alt="root" src="https://github.com/user-attachments/assets/4fec6c1c-b99c-410a-b7b8-669c2a727d0b" />


## Lessons Learned 📘
- Don't assume default ports. When ssh didn't appear on port 22, checking common alternatives (2222) confirmed it was simply relocated. Always scan the full range rather than trusting defaults.
- Enumeration is the foundation. The default Apache page hid the real attack vector. Tools like ffuf and a quick read of robots.txt exposed the CMS that everything else depended on.
- Version numbers are gold. Identifying the exact CMS Made Simple version was what made matching it to CVE-2019-9053 possible. A footer version string is worth scrolling for.
- Understand the exploit before running it. Knowing this was a time-based blind SQL injection (inferring characters from server response delays) explained both how it worked and why it could be slow.
- Mind your tooling and inputs. The exploit shipped as Python2, so porting it to Python3 was necessary. Likewise, the wordlist had to be utf-8 encoded — an overlooked encoding issue caused a decode error and a failed run.
- Stabilize your shell early. Upgrading to a proper interactive TTY made enumeration and privilege escalation far smoother and prevented common shell-handling headaches.
- Check sudo -l first for privesc. A single misconfigured sudo entry (vim as root, no password) was the entire path to root. Legitimate binaries with elevated rights are a classic, high-impact misconfiguration (see GTFOBins).

## Tools Used 🧰
- `nmap`
- `ffuf`
- `searchsploit`
- `vim`
