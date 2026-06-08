# Agent Sudo 👽
This TryHackMe room is a beginner-friendly challenge that covers a wider range of techniques than most entry-level rooms, making it a solid step up from basic enumeration and privesc. It begins with `nmap` reconnaissance to identify open ports and services, followed by manipulating the **User-Agent** header in an HTTP request to uncover a hidden agent codename. From there, `hydra` is used to brute-force FTP credentials, which surfaces a set of files containing hidden data. Extracting that data requires **steganography** tools to uncover concealed information buried inside the files, eventually yielding valid SSH credentials for an initial foothold. After capturing the user flag, privilege escalation revolves around a known `sudo` vulnerability (**CVE-2019-14287**) that allows a restricted user to run commands as root by manipulating the user ID — reinforcing a diverse set of skills including service enumeration, HTTP header manipulation, credential brute-forcing, steganography, and CVE-based privilege escalation along the way.



## Reconnaissance 🔍
As per usual, the first thing I did was run a `nmap` scan to enumerate the machine and find open ports/services running on the machine:
```
