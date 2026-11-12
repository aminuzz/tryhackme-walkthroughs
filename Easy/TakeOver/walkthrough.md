# TakeOver -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** Web Enumeration / Subdomain Discovery

## Introduction
TakeOver is an easy-rated TryHackMe room focused entirely on subdomain enumeration. The target domain, `futurevera.thm`, hides its flag behind a subdomain that is not linked anywhere obvious. The room tests whether you can enumerate virtual hosts, recognize when a subdomain looks like a dead end, and pull additional information out of an SSL/TLS certificate to uncover the final hidden subdomain that holds the flag. It is a good introduction to how local DNS resolution works and how much information a certificate can leak.

### Background: What is /etc/hosts?
The `/etc/hosts` file is a plain-text operating system file that maps hostnames to IP addresses locally. The operating system checks this file before querying network DNS servers, which allows redirecting domains, blocking sites locally, or reaching a site before it is publicly deployed. It lives at `/etc/hosts` on Linux and `C:\Windows\System32\drivers\etc\hosts` on Windows.

On Windows:
<img width="1919" height="1034" alt="image" src="https://github.com/user-attachments/assets/bf1d1985-830b-4a20-9c2e-f9e6f86dd305" />

On Kali Linux:
<img width="1461" height="915" alt="image" src="https://github.com/user-attachments/assets/a5271053-5385-4113-9cd9-0da0458ed15d" />

The reason this matters: opening the provided URL directly returns a "site cannot be reached" error, because the machine has no way to resolve `futurevera.thm` to an IP:
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/6d65d01b-b531-4608-a1bb-6a2e14432d42" />

Accessing the site by its raw IP address, however, works without issue:
<img width="1916" height="1077" alt="image" src="https://github.com/user-attachments/assets/d81d3e94-53e7-421b-949a-73ceb7a83064" />

## Setup
### Editing /etc/hosts
Since the room explicitly instructs you to add the target IP to `/etc/hosts`, that was the first step. I used a Kali VM with Vim, though the same can be done on Windows by editing the file with Notepad. If you are working from your local machine rather than the AttackBox, you will also need [OpenVPN](https://tryhackme.com/access) connected to reach the target.

After adding the mapping, I confirmed the site was reachable by its URL rather than its IP:
![Gif10](https://github.com/user-attachments/assets/ffc45aab-9a46-4612-a713-682d33bbcecd)

## Enumeration
### Subdomain Discovery
The room specifically calls for uncovering hidden subdomains, so `gobuster` was the tool of choice.

An initial attempt in `dns` mode returned no matching hits:
![Gif11](https://github.com/user-attachments/assets/73c11bdf-3eec-4d65-a49f-43bf9194251a)

After exhausting the default lists in `/usr/share/wordlists`, I installed [SecLists](https://github.com/danielmiessler/SecLists), a much larger set of security wordlists that is not built into Kali by default.

Switching `gobuster` to `vhost` mode then surfaced two subdomains:
```
gobuster vhost -u "https://futurevera.thm" --domain futurevera.thm -w subdomains-top1million-110000.txt -t 64 -ad -k
```
![Gif12](https://github.com/user-attachments/assets/f8c6d71b-144b-4f70-aae9-6fdf462b5b03)

The distinction matters: `dns` mode brute-forces DNS records, while `vhost` mode brute-forces the `Host` header against the same IP, which is what finds name-based virtual hosts that have no DNS entries of their own. Before visiting either subdomain, I added them to `/etc/hosts`.

## Investigating the Subdomains
### blog.futurevera.thm
I first checked `blog.futurevera.thm`, including the page source and any embedded JavaScript:
<img width="1474" height="916" alt="image" src="https://github.com/user-attachments/assets/8035168f-79a1-48ab-b20a-848cb4458693" />
<img width="1476" height="916" alt="image" src="https://github.com/user-attachments/assets/913bc90e-b678-454b-9b34-1c55c12aaad5" />

Nothing of interest turned up here.

### support.futurevera.thm
Next was `support.futurevera.thm`, which at first glance appeared to be a placeholder stating the site was still under development:
<img width="1467" height="915" alt="image" src="https://github.com/user-attachments/assets/f00fb398-20f5-4676-b73f-bb0e2f822483" />

The breakthrough came from inspecting the site's SSL/TLS certificate, which leaked a secret subdomain name in its details:
<img width="1481" height="923" alt="image" src="https://github.com/user-attachments/assets/8258d0a6-fdde-4c36-9fc9-b32a14c1adbc" />

A certificate's Subject Alternative Name (SAN) field often lists every hostname it was issued for, which is exactly the kind of information disclosure that reveals subdomains no wordlist would find. I copied the leaked domain and added it to `/etc/hosts`.

## Flag Capture
With the secret subdomain resolving locally, opening it in the browser returned the flag. Note that the URL has to be switched from HTTPS to HTTP to see it:
![Gif15](https://github.com/user-attachments/assets/51331bdb-095f-44f7-871f-e5710fd84ae5)

## Lessons Learned
- **DNS mode and vhost mode are not interchangeable.** `gobuster dns` found nothing because the subdomains had no DNS records, while `vhost` mode brute-forcing the `Host` header found them immediately. Knowing which one to reach for saves a lot of wasted scans.
- **SSL/TLS certificates leak subdomains.** The final, hidden subdomain was never going to appear in a wordlist, but it was sitting in plain sight in the certificate's details. Certificate inspection belongs in any web enumeration routine.
- **An "under development" page is not a dead end.** The placeholder on `support.futurevera.thm` looked empty, but the certificate behind it held the key. Judging a target by its rendered page alone leaves information on the table.
- **Wordlist quality matters.** Swapping Kali's default lists for SecLists was the difference between no hits and a full subdomain list.

## Tools Used
- `gobuster`
- SecLists
- Vim (editing `/etc/hosts`)
