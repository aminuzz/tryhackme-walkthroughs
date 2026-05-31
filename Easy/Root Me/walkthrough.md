# Root Me 💻
This TryHackMe room is a beginner-friendly challenge centered on web exploitation and privilege escalation. It begins with reconnaissance using tools like nmap to map out open ports and services, followed by directory enumeration to uncover a hidden file-upload page. From there, you bypass the upload filter to deliver a PHP reverse shell and gain an initial foothold on the machine. After capturing the user flag, the room walks you through privilege escalation by identifying a misconfigured SUID binary and abusing it to spawn a root shell, reinforcing core concepts like enumeration, filter evasion, and Linux privilege escalation along the way.


## Reconnaissance 🔍
Once connected to the TryHackMe network, my first order of business was to conduct a `nmap` scan to look for open ports on the machine to get an idea on what this machine is running:
```
nmap -sV -sC -p- -T4 10.64.143.151
```
