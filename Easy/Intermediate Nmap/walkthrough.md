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

<!-- SCREENSHOT: nmap command -->

The output revealed the key detail:

<!-- SCREENSHOT: nmap scan output -->

<!-- SCREENSHOT: nmap scan output (continued) -->

- Port 31337 exposed a username and password for an Ubuntu machine.
- Two SSH ports were open, 22 and 2222, so the next task was to determine which one the credentials were actually for.

Credentials recovered:
- `ubuntu:<password>`

## Gaining Access
Scanning all ports (`-p-`) was what surfaced the credentials on port 31337, a port a default scan would never have checked. With two SSH services available, the only question was which port accepted the login.

I first tried SSH on port 2222, assuming the non-standard port was the intended one, but that failed:

<!-- SCREENSHOT: failed SSH login on port 2222 -->

Trying the standard port 22 worked, and a shell was granted:

<!-- SCREENSHOT: successful SSH login on port 22 -->

## Flag Capture
Looking through the home directory, a `user` directory contained the flag:

<!-- SCREENSHOT: reading the flag -->

## Lessons Learned
- **Scan every port.** The credentials lived on port 31337, far outside the default top-1000 range. Running `-p-` is the difference between finding the foothold and missing it entirely.
- **Read the full scan output.** The answer was handed over in the scan results; the work was noticing it, not exploiting anything.
- **Do not assume a non-standard port is the intended one.** The credentials worked on the standard SSH port 22, not the non-standard 2222. Test rather than assume when more than one candidate service is open.

## Tools Used
- `nmap`
- `ssh`
