# Startup -- TryHackMe Writeup

**Difficulty:** Easy  
**Category:** FTP / Web Exploitation / Packet Analysis / Linux Privilege Escalation

## Introduction
Startup is an easy-rated TryHackMe room built around a misconfigured FTP service. Anonymous FTP access is allowed and the uploadable share is also served by the web server, which turns a simple file upload into remote code execution through a PHP reverse shell. From a `www-data` foothold, a packet capture left on the box leaks a reusable password, and the final step abuses a root-scheduled script that sources a writable file to escalate to root.

## Enumeration
### Nmap Scan
A full-port service scan identified three open ports:

<img width="1568" height="783" alt="image" src="https://github.com/user-attachments/assets/d7fb5c63-406a-4f5c-a32d-8808ab86d95c" />

- Port 21 (FTP, vsftpd 3.0.3) with anonymous login allowed and a world-writable `ftp` directory.
- Port 22 (SSH, OpenSSH 7.2p2).
- Port 80 (HTTP, Apache 2.4.18), serving a page titled "Maintenance."

The two standout details are the anonymous FTP login and the writable share flagged by Nmap (`[NSE: writeable]`), which immediately suggest an upload path.

### Web Enumeration
Directory fuzzing with `ffuf` surfaced a `/files` directory:

<img width="1568" height="605" alt="image" src="https://github.com/user-attachments/assets/2c9bd745-ee1b-4c36-a20a-50f34836d208" />

Browsing to it returned an open directory listing containing an `ftp/` folder, `important.jpg`, and `notice.txt`:

<img width="1489" height="812" alt="image" src="https://github.com/user-attachments/assets/30ab31a6-2c7c-424a-9fd1-7a468fd3551d" />

This is the key observation for the entire foothold: the `/files/ftp` directory on the web server is the same share exposed over anonymous FTP. Anything written into the FTP share becomes reachable, and executable, over HTTP.

`notice.txt` contained an in-character note hinting at a username and setting up the room's theme:

<img width="1512" height="791" alt="image" src="https://github.com/user-attachments/assets/50fff3d7-370d-4fd4-a0a8-5c53913a9974" />

The `ftp/` listing and `recipe.txt` answered one of the room's questions, naming the "secret spicy soup" ingredient as love:

<img width="1565" height="784" alt="image" src="https://github.com/user-attachments/assets/3f75b72e-4f3c-490e-8999-fc44393e8361" />

Running `binwalk` against `important.jpg` showed an embedded PNG and a Zlib-compressed stream, but extraction led nowhere useful, making the image a dead end rather than part of the attack path:

<img width="1568" height="649" alt="image" src="https://github.com/user-attachments/assets/5584fdde-8aaf-46e7-9d77-8a3857e37a14" />

## Gaining Access
### How the Exploit Works: Anonymous FTP Write to the Web Root
The vulnerability is a configuration flaw, not a software bug. The vsftpd server accepts anonymous logins, and the `ftp` directory is world-writable. Because Apache serves that same directory at `http://<target>/files/ftp/`, a file uploaded over FTP can be requested in a browser, and the web server will execute it. Uploading a PHP reverse shell (pentestmonkey's `php-reverse-shell.php`, with the listener IP and port set) and then requesting it causes Apache to run the script, connecting back to a waiting `nc` listener.

### Uploading and Triggering the Shell
The reverse shell was uploaded through anonymous FTP with `put`:

<img width="1568" height="146" alt="image" src="https://github.com/user-attachments/assets/35f521b1-4492-4324-8b29-74a65320921b" />

With a `nc` listener running on the configured port, requesting the uploaded script at `/files/ftp/php-reverse-shell.php` returned a shell as `www-data`. The shell was then stabilized:
```
python -c 'import pty; pty.spawn("/bin/bash")'
```

## Pivoting
### Finding the Packet Capture
Enumerating the file system showed an `incidents` directory owned by `www-data`, which stood out because most of the system is root-owned:

<img width="1568" height="755" alt="image" src="https://github.com/user-attachments/assets/991a6211-9248-417c-be1d-3af5b462fa3b" />

Inside was `suspicious.pcapng`, a packet capture. To analyze it, it was served from the target with a simple HTTP server and pulled down to the attacking machine:

<img width="1568" height="193" alt="image" src="https://github.com/user-attachments/assets/2f420806-6bbf-42ab-9243-e5cc5560f715" />

<img width="1568" height="262" alt="image" src="https://github.com/user-attachments/assets/3aa28679-3d3d-4e81-b1a2-d13df9387a10" />

### Analyzing the Capture
Opening the capture in Wireshark and applying an `http` filter showed a prior operator's activity against the box, including the request to `/files/ftp/shell.php` and the resulting reverse shell traffic:
<img width="1488" height="812" alt="image" src="https://github.com/user-attachments/assets/2871c040-6484-4aa4-8d4d-8006c07b4e8d" />

<img width="709" height="87" alt="image" src="https://github.com/user-attachments/assets/86e27709-089a-4476-93a0-851bd0ee4d4f" />

Following the TCP stream of that session replayed the attacker's commands in plaintext. Among them was a `sudo` attempt where the password was typed directly into the shell, leaking a credential:

<img width="373" height="94" alt="image" src="https://github.com/user-attachments/assets/54005bfa-cf85-469d-a3c2-332deef1073f" />

The recovered password was `c4ntg3t3n0ughsp1c3`. It does not work for `www-data`, but `/etc/passwd` confirmed a `lennie` account, and the password reused successfully for SSH as that user:

<img width="973" height="559" alt="image" src="https://github.com/user-attachments/assets/82892e1f-8175-4b3b-a5bc-64c04e614c04" />

Logging in as `lennie` over SSH gave the user flag:
```
THM{03ce3d619b80ccbfb3b7fc81e46c0e79}
```

## Privilege Escalation
### How the Exploit Works: Writable Script Sourced by a Root Job
`lennie`'s home directory held a `scripts` folder:

<img width="384" height="58" alt="image" src="https://github.com/user-attachments/assets/aec81c1e-34bd-484e-b4bc-c1dadf131142" />

The folder contained `planner.sh` (owned by root) and `startup_list.txt`, whose timestamp kept updating, a sign that `planner.sh` runs on a schedule as root:

<img width="795" height="453" alt="image" src="https://github.com/user-attachments/assets/1a097d92-7e19-41a2-b94c-0a8bda0e7b7f" />

Reading `planner.sh` showed it writes a variable to `startup_list.txt` and then executes `/etc/print.sh`:

<img width="745" height="200" alt="image" src="https://github.com/user-attachments/assets/6012d717-31aa-45d3-8ce5-25a8f9c190f0" />

The flaw is that `/etc/print.sh` is writable by `lennie`, while `planner.sh` that calls it runs as root. Overwriting `print.sh` with attacker-controlled commands means those commands execute with root privileges the next time the scheduled job fires.

### Exploiting It
`/etc/print.sh` was replaced with a bash reverse shell pointing back at a listener:

<img width="575" height="76" alt="image" src="https://github.com/user-attachments/assets/3e6942f3-3029-4249-b9fa-8a3e25d2d9d2" />

When the root-run `planner.sh` next executed `print.sh`, the listener received a shell running as root, giving access to the root flag.

## Flag Capture
Both flags were recovered: the user flag after reusing the pcap-leaked password to SSH in as `lennie`, and the root flag after overwriting the writable `/etc/print.sh` that a root-scheduled script executes.

## Lessons Learned
- **Writable anonymous FTP that overlaps the web root is remote code execution.** Neither the anonymous login nor the writable share is fatal alone, but because the share is also served by Apache, an upload becomes an executable endpoint.
- **Artifacts left on a box are credentials waiting to be read.** A packet capture sitting in a `www-data`-accessible directory leaked a plaintext password that unlocked the next account.
- **Plaintext protocols leak everything.** The captured session exposed typed commands and a password in the clear, which is exactly why credential reuse over unencrypted channels is dangerous.
- **A writable file in a privileged job is a full escalation path.** The real weakness was not `planner.sh` itself but the writable `/etc/print.sh` it sourced as root. Privileged automation should never execute files a lower-privileged user can modify.

## Tools Used
- `nmap`
- `ffuf`
- `ftp`
- `binwalk`
- `nc`
- `python3 -m http.server`
- `wget`
- **Wireshark**
- pentestmonkey `php-reverse-shell.php`
