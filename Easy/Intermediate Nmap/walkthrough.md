# Intermediate Nmap -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** Enumeration / Nmap

## Introduction
Intermediate Nmap is a short, easy-rated TryHackMe room focused on reinforcing `nmap` enumeration habits. The goal is to run a thorough scan, read the output carefully to spot credentials exposed on a non-standard port, and then use those credentials to log in over SSH and retrieve the flag. It is a good reminder that the answer is often already in the scan output if you read every port rather than just the common ones.

## Enumeration
### Nmap Scan
The first step was a thorough scan covering service versions, default scripts, aggressive timing, and all ports:
```
nmap -sV -sC -T4 -p- <target>
```
- `-sV` grabs the service version running on each port (for example, identifying the exact Apache version behind port 80).
- `-sC` runs `nmap`'s default scripts, which dig deeper into each service to enumerate things like SMB shares, HTTP titles, banners, and SSL certificates.
- `-T4` speeds up the scan.
- `-p-` scans all 65535 ports, rather than only the top 1000, so nothing on a non-standard port is missed.

<img width="493" height="47" alt="image" src="https://github.com/user-attachments/assets/7c52d6ee-b0c1-4f58-a307-74819cadc824" />


The output revealed the key detail:
<img width="1914" height="766" alt="image" src="https://github.com/user-attachments/assets/cec4b38d-f9d3-4c49-b28e-ad1bbf0fdae5" />
<img width="1493" height="571" alt="image" src="https://github.com/user-attachments/assets/662d37c0-2371-436c-bb4d-ebd3c3fef1c9" />
- Port 31337 exposed a username and password for an Ubuntu machine.
- Two SSH ports were open, 22 and 2222, so the next task was to determine which one the credentials were actually for.

Credentials recovered:
- `ubuntu:Dafdas!!/str0ng`

## Gaining Access
Scanning all ports (`-p-`) was what surfaced the credentials on port 31337, a port a default scan would never have checked. With two SSH services available, the only question was which port accepted the login.

I first tried SSH on port 2222, assuming the non-standard port was the intended one, but that failed:
<img width="745" height="188" alt="image" src="https://github.com/user-attachments/assets/efd6753b-c9f5-4c5c-97c1-622165bede13" />


Trying the standard port 22 worked, and a shell was granted:
<img width="683" height="487" alt="image" src="https://github.com/user-attachments/assets/aca7a4cf-ef06-46ba-9e20-c1ff80edac43" />


## Flag Capture
Looking through the home directory, a `user` directory contained the flag:
<img width="580" height="409" alt="image" src="https://github.com/user-attachments/assets/b69330ad-1642-425a-991d-296bda3b17ec" />


## Lessons Learned
- **Scan every port.** The credentials lived on port 31337, far outside the default top-1000 range. Running `-p-` is the difference between finding the foothold and missing it entirely.
- **Read the full scan output.** The answer was handed over in the scan results; the work was noticing it, not exploiting anything.
- **Do not assume a non-standard port is the intended one.** The credentials worked on the standard SSH port 22, not the non-standard 2222. Test rather than assume when more than one candidate service is open.

## Tools Used
- `nmap`
- `ssh`
