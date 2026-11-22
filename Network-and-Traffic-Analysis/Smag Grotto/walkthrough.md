# Smag Grotto -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** Network Traffic Analysis / Linux Privilege Escalation

## Introduction
Smag Grotto is an easy-rated TryHackMe room that moves from packet analysis to a full root compromise. A hidden mail directory leaks a packet capture containing plaintext login credentials, which open a hidden virtual host that runs a command box, turned into a reverse shell. Privilege escalation then abuses a root cron job that trusts a writable file to plant an SSH key, followed by a passwordless `sudo apt-get` misconfiguration to reach root.

## Enumeration
### Nmap Scan
A quick but comprehensive `nmap` scan to find all open ports and services:

<img width="855" height="319" alt="image" src="https://github.com/user-attachments/assets/a403776f-1cef-4eb9-b115-b26251c041ac" />

- Port 22 is open for SSH.
- Port 80 is open for HTTP, running Apache 2.4.18.

### Web Enumeration
The website itself (and its page source) gave nothing away, so the next step was directory enumeration with `ffuf`:

<img width="1915" height="919" alt="image" src="https://github.com/user-attachments/assets/2e158b7c-397f-46da-a732-72c9adbaf671" />

<img width="1087" height="671" alt="image" src="https://github.com/user-attachments/assets/4af29c83-39f7-4d1d-b925-f176d29b290d" />

- The `mail` directory stood out, returning a 301 redirect to another page.

### Investigating the Mail Directory
The `mail` directory contained a set of email conversations:

<img width="1914" height="919" alt="image" src="https://github.com/user-attachments/assets/01b26e63-0431-4a13-b365-2d31fa85b285" />

- One conversation linked to a `pcap` file. The emails discussed migrating services to a larger `10.10.0.0/8` network (a /8, where 8 bits identify the network and the remaining 24 bits identify over 16 million possible hosts), and noted that a direct download failed, so the file should be fetched with `wget`.

## Gaining Access
### Analyzing the PCAP
Clicking the link downloaded the packet capture, which was opened in Wireshark:

<img width="1912" height="918" alt="image" src="https://github.com/user-attachments/assets/3689a3c5-6cd7-417f-9abe-5b385ff0b738" />

<img width="1911" height="882" alt="image" src="https://github.com/user-attachments/assets/aeeedf57-6044-46be-bc4e-d9d1610736b2" />

- The capture shows two hosts, `192.168.33.10` (client) and `192.168.33.69` (server).
- After a completed TCP handshake, the client sends an HTTP `POST` request to `/login.php`. Inspecting that packet exposes the credentials submitted to the login form:

<img width="1911" height="887" alt="image" src="https://github.com/user-attachments/assets/f938beca-bdaa-4836-b9e8-88f3b68c7e66" />

- Username: `helpdesk`
- Password: `cH4nG3M3_n0w`

The server replied with a `200`, confirming the login succeeded.

### Discovering the Virtual Host
`/login.php` does not exist under the box's IP directly. Looking more closely at the captured request revealed the `Host` header pointed to a subdomain, `development.smag.thm`:

<img width="939" height="384" alt="image" src="https://github.com/user-attachments/assets/d9b5ed06-ee17-4d94-8813-01a7087662ab" />

Trying to reach it by `ping` or in the browser failed, because DNS could not resolve it:

<img width="483" height="64" alt="image" src="https://github.com/user-attachments/assets/a90ce819-fd74-4630-b023-fdec76995a1c" />

This is name-based virtual hosting: one IP serves multiple sites, selected by the `Host` header. Adding the IP and subdomain to `/etc/hosts` lets the local machine resolve it:

<img width="1308" height="213" alt="image" src="https://github.com/user-attachments/assets/f7e2c8af-27d0-4e23-bcc1-0b55626c13de" />

### Accessing the Development Site
With the host resolving, the development site exposed two pages of interest, `admin.php` and `login.php`. Using the `helpdesk` credentials recovered from the pcap, I logged in through `login.php`:

<img width="1919" height="923" alt="image" src="https://github.com/user-attachments/assets/79ffdec8-09f6-40b7-969b-69f8934e60ee" />

<img width="1913" height="885" alt="image" src="https://github.com/user-attachments/assets/52c8b13d-c3b1-4f66-ba90-fc40510c3fb0" />

### How the Exploit Works: Command Box to Reverse Shell
The page after login presented an "Enter a command" box. Simple Linux commands like `whoami`, `ls`, and `id` returned nothing, which suggested the input was not being run directly as a shell command. Since the application is PHP, the box was likely executing PHP rather than raw shell input, so a PHP reverse shell was the right payload: it runs on the server, opens an outbound connection back to a waiting `nc` listener, and drops into a shell.

Submitting a PHP reverse shell worked and returned a shell:

<img width="1914" height="919" alt="image" src="https://github.com/user-attachments/assets/6baa131f-a686-4fc2-b8b9-7c108b83458c" />

<img width="528" height="132" alt="image" src="https://github.com/user-attachments/assets/e2727370-29f0-4089-a238-fbe9e904b7e3" />

The shell was then upgraded to a proper interactive TTY:

<img width="559" height="245" alt="image" src="https://github.com/user-attachments/assets/72aaa041-1ab9-4d96-ad0e-b421040fe87b" />

Looking around, `user.txt` was present but not readable by the current web user, meaning the next step was to pivot to the `jake` user:

<img width="405" height="71" alt="image" src="https://github.com/user-attachments/assets/ebb6965d-8e97-4d84-b8c3-ead1172d49e8" />

## Privilege Escalation to jake
### How the Exploit Works: Cron Job SSH Key Injection
To enumerate further, I hosted `linpeas.sh` with a Python web server (`python -m http.server 80`, which serves files from the current directory), downloaded it to the target, and ran it:

<img width="711" height="271" alt="image" src="https://github.com/user-attachments/assets/9f5c12b5-ec19-4e80-b68f-41ed26426133" />

<img width="635" height="308" alt="image" src="https://github.com/user-attachments/assets/c3475a40-29fb-429d-846a-760a2199bcbb" />

`linpeas` surfaced a root cron job that takes a *backup* public key and writes it into jake's `authorized_keys` file:

<img width="898" height="139" alt="image" src="https://github.com/user-attachments/assets/46ff7f8e-e374-4c05-9f92-7495d1373cd2" />

This is the escalation vector. SSH key authentication works by having the server store a user's public key in `authorized_keys`; anyone holding the matching private key can then log in as that user without a password. Because this cron job copies a backup key file into jake's `authorized_keys` on a schedule, and that backup file is writable by the current user, I can replace its contents with my own public key. On the next run, the cron job plants my key as jake's, letting me SSH in as jake.

Confirming the backup file was writable:

<img width="658" height="41" alt="image" src="https://github.com/user-attachments/assets/bc410644-e229-4f77-8a7b-2736cc442aca" />

Generating a new key pair:

<img width="992" height="365" alt="image" src="https://github.com/user-attachments/assets/ce17d5e6-dcf8-4cda-a5e9-e46d15b97455" />

Writing my public key into the backup file:

<img width="1907" height="81" alt="image" src="https://github.com/user-attachments/assets/af54fba4-0710-4089-8c11-4448fee4ee98" />

After the cron job ran, SSH as jake with the private key succeeded:

<img width="701" height="298" alt="image" src="https://github.com/user-attachments/assets/52b9c628-9bc7-4038-b82c-81f0d05b19d9" />

And `user.txt` was now readable:

<img width="616" height="296" alt="image" src="https://github.com/user-attachments/assets/719c0838-5719-4b1d-8318-6dbd823017e5" />

## Privilege Escalation to root
### How the Exploit Works: Passwordless sudo on apt-get
Checking the sudoers configuration showed that any user can run `apt-get` as root without a password:

<img width="957" height="103" alt="image" src="https://github.com/user-attachments/assets/b4d2e156-1887-486e-b23d-60735969b1e1" />

That is a direct path to root. `apt-get` has a feature that runs commands during package operations (for example, through its `APT::Update::Pre-Invoke` hooks or its pager), and because it can be run as root via sudo, that command executes as root. [GTFOBins](https://gtfobins.org/gtfobins/apt-get/#shell) documents the exact technique:

<img width="1915" height="943" alt="image" src="https://github.com/user-attachments/assets/1605d85e-9230-42ce-91e0-728e61918d6c" />

Running the documented command spawned a root shell and gave up the flag:

<img width="1100" height="180" alt="image" src="https://github.com/user-attachments/assets/4167bd35-73b0-4b47-b6b3-c59777787072" />

## Flag Capture
Both flags were recovered: the user flag after escalating to jake via the cron-job SSH-key injection, and the root flag after abusing the passwordless `sudo apt-get` misconfiguration to spawn a root shell.

## Lessons Learned
- **Packet captures are credential goldmines.** The entire foothold came from a pcap left in a mail directory. A single plaintext `POST` to `/login.php` handed over working credentials, a reminder of why HTTP (versus HTTPS) is dangerous.
- **Read the full request, not just the body.** The credentials were in the POST body, but the `Host` header is what revealed the hidden virtual host that those credentials actually unlocked.
- **Virtual hosts hide entire sites.** A site that will not resolve is not necessarily down; it may be a name-based vhost on the same IP. Adding it to `/etc/hosts` made the development site reachable.
- **Test what a command box actually executes.** Raw shell commands failed because the box ran PHP, not shell input. Matching the payload to the execution context (a PHP reverse shell) was the key.
- **Writable files touched by root cron jobs are escalation vectors.** The cron job itself was fine; the problem was that it trusted a file a lower-privileged user could edit. Anything a root process reads or executes on a schedule is worth checking for weak permissions.
- **Passwordless sudo on a package manager is root.** Tools like `apt-get`, `apt`, and many others can run arbitrary commands, so `sudo NOPASSWD` on them is equivalent to full root access. Always check `sudo -l` and cross-reference GTFOBins.

## Tools Used
- `nmap`
- `ffuf`
- Wireshark
- `wget`
- `nc`
- `linpeas`
- `ssh-keygen` / `ssh`
- [GTFOBins](https://gtfobins.org/)
