# Simple CTF
This TryHackMe room focuses on basic web exploitation and privilege escalation techniques in a beginner-friendly environment. The challenge starts with reconnaissance using tools like `nmap` to identify open ports and services running on the target machine. From there, you investigate the CMS in use, determine which known CVE it is vulnerable to, and exploit the vulnerability to gain initial access. After establishing a foothold, the room guides you through simple enumeration and privilege escalation steps, including leveraging `vim` to obtain elevated privileges, while answering questions along the way to reinforce the concepts being practiced.


## Question 1 🔍
How many services are running under port 1000?

I ran a simple `nmap` scan to scan all ports under port 1000:
<img width="1541" height="910" alt="Screenshot_2026-05-21_16-59-41" src="https://github.com/user-attachments/assets/d6a41861-fbc9-4671-b177-30de28e85d49" />
- This `nmap` scan was a deep scan which picked up the following:
  - `ftp` running on port 21 specifically vsftpd 3.0.3
  - `http` running on port 80 specifically Apache httpd 2.4.18
  - `ssh` wasn't found under port 22 meaning that it most likely is running on port 2222 (the most common alternative)
 
The answer for Question 1 is 2 services being `ftp` and `http`.


## Question 2
What is running on the higher port?

Based on our second scan I found that `ssh` is running on an alternative port being port 2222 which is confirmed in the screenshot below:
<img width="1364" height="915" alt="Screenshot_2026-05-21_17-04-53" src="https://github.com/user-attachments/assets/075f8af5-0004-494e-8170-8d8e7691dfc6" />


## Question 3 🔓
What's the CVE you're using against the application?

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

