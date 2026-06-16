# Lazy Admin
This TryHackMe room is a beginner-friendly Linux challenge that demonstrates how a chain of small misconfigurations can lead to full root compromise. It begins with `nmap` reconnaissance to identify open ports and services, followed by web directory enumeration to uncover a hidden **SweetRice CMS** installation. From there, further directory fuzzing surfaces a publicly accessible backup file containing a MySQL database dump — which leaks an admin username and a hashed password. After cracking the hash and logging into the CMS admin panel, a file upload vulnerability in SweetRice is exploited to deploy a PHP reverse shell and establish an initial foothold on the machine. Privilege escalation is achieved by abusing misconfigured `sudo` permissions on a Perl script that executes a writable shell script, which is modified to deliver a second reverse shell as root — reinforcing core concepts like web enumeration, credential recovery, hash cracking, file upload exploitation, and `sudo`-based privilege escalation along the way.







### Quick Disclaimer
You'll notice that the IP addresses are different in different commands and that's because I worked on the challenge in multiple sessions and in each session, the IP address changes.

















## Reconnaissance 🔍
The first step was to run a 'nmap' scan to enumerate the machine and look for open ports/services running on the machine:
```
nmap -sV -sC -T4 10.65.147.221
```
- `-sV` obtains the service version of the service
- `-sC` runs default `nmap` scripts found and obtains additional information about the service running on a particular port
- `-T4` uses aggresive timing to speed up the scanning process (real world scenario would be very noisy)

<img width="1913" height="513" alt="nmap_scan" src="https://github.com/user-attachments/assets/96e402ec-5cd4-45c9-bee8-e36c6890ed25" />
- port 22 is running `ssh` specifically **OpenSSH 7.2p2**
- port 80 is running `http` specifically **Apache 2.4.18**


Based on the `nmap` scan, the landing page of this particular website is the Apache2 default web page as shown below:
<img width="1920" height="1003" alt="default_webpage" src="https://github.com/user-attachments/assets/f39b1b55-bfc1-47cd-90a5-51e7f94e941b" />

Upon checking the HTML I knew that there had to be hidden web pages for this challenge so I used `ffuf` to enumerate the IP address for hidden pages:
```
ffuf -u http://10.65.147.221/FUZZ -w /usr/share/wordlists/dirb/big.txt -e .js,.py,.php,.txt,.html,.css -mc 200,301,302
```
