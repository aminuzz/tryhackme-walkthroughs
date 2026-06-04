# Root Me 💻
This TryHackMe room is a beginner-friendly challenge centered on web exploitation and privilege escalation. It begins with reconnaissance using tools like nmap to map out open ports and services, followed by directory enumeration to uncover a hidden file-upload page. From there, you bypass the upload filter to deliver a PHP reverse shell and gain an initial foothold on the machine. After capturing the user flag, the room walks you through privilege escalation by identifying a misconfigured SUID binary and abusing it to spawn a root shell, reinforcing core concepts like enumeration, filter evasion, and Linux privilege escalation along the way.


## Reconnaissance 🔍
Once connected to the TryHackMe network, my first order of business was to conduct a `nmap` scan to look for open ports on the machine to get an idea on what this machine is running:
```
nmap -sV -sC -p- -T4 10.64.143.151
```
<img width="935" height="627" alt="nmap_scan" src="https://github.com/user-attachments/assets/2702547d-36e3-46eb-a50e-044c2f209fe6" />

- Port 22 is open running `ssh` specifically version 8.2p1
- Port 80 is open running `http` specifically Apache 2.4.41
- The title of the website is **HackIT - Home**

The next step was to visit the website to see what I was working with:
<img width="960" height="1002" alt="image" src="https://github.com/user-attachments/assets/fa3ccf9a-be18-4e49-a7c6-726417f024f5" />

The main landing page for the website was bland and didn't have any clues on where to go next so using `ffuf`, I enumerated the website to find hidden webpages:
```
