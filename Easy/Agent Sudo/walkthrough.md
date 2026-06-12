# Agent Sudo 👽
This TryHackMe room is a beginner-friendly challenge that covers a wider range of techniques than most entry-level rooms, making it a solid step up from basic enumeration and privesc. It begins with `nmap` reconnaissance to identify open ports and services, followed by manipulating the **User-Agent** header in an HTTP request to uncover a hidden agent codename. From there, `hydra` is used to brute-force FTP credentials, which surfaces a set of files containing hidden data. Extracting that data requires **steganography** tools to uncover concealed information buried inside the files, eventually yielding valid SSH credentials for an initial foothold. After capturing the user flag, privilege escalation revolves around a known `sudo` vulnerability (**CVE-2019-14287**) that allows a restricted user to run commands as root by manipulating the user ID — reinforcing a diverse set of skills including service enumeration, HTTP header manipulation, credential brute-forcing, steganography, and CVE-based privilege escalation along the way.

### Quick Disclaimer
You'll notice that the IP addresses are different in different commands and that's because I worked on the challenge in multiple sessions and in each session, the IP address changes.


## Reconnaissance 🔍
As per usual, the first thing I did was run a `nmap` scan to enumerate the machine and find open ports/services running on the machine:
```
nmap -sV -sC -T4 10.64.142.8
```
- `sV` obtains the service version of the service
- `-sC` runs default `nmap` scripts on services found and obtains additional information; for example on port 80 the script pulled the `http-server-header` from the `Server:` field in the HTTP response header and the `http-title` from the `<title>` tag in the HTML response body
- `-T4` sends packets aggressively 
<img width="957" height="579" alt="Screenshot_2026-06-08_13-43-00" src="https://github.com/user-attachments/assets/4e7f71c8-dfa9-45f3-9dc1-0fb4302b120f" />
- port 21 is running the `ftp` service specifically version `vsftpd 3.0.3`
- port 22 is running `ssh` specifically version `7.6p1`
- port 80 is running an Apache web server specifically **Apache 2.4.29**
- `nmap` also picked up the title of the webpage being **Announcement**

Unlike previous challenges, this challenge does not allow for `anonymous` login on port 21 so the next logical step was to check the website:
<img width="960" height="1001" alt="website_ss" src="https://github.com/user-attachments/assets/f183af42-a605-4e77-84ff-69de57778fa5" />
- The website states to use our own **codename** as the user agent to access the website
- The **user agent** header specifies what browser we are using to make a network request

Using a tool like **Burpsuite** where you can manually edit HTTP headers, I edited the `user-agent` header to access the website.


### Burpsuite 
Upon opening **Burpsuite**, I was greeted with this screen:
<img width="1920" height="1048" alt="burpsuite" src="https://github.com/user-attachments/assets/d76ac4b7-83c9-4afa-a85c-d8e6c81d65fa" />
- This is the **Target** screen where it opens a custom Chromium based web browser so it can build a site map where it shows an organized overview of the website

Once I visited the site on the Burpsuite Browser, the following information showed up on Burpsuite:
<img width="1920" height="1044" alt="burpsuite_1" src="https://github.com/user-attachments/assets/3138751d-270b-43cc-a90e-4ea4b5a504ef" />

- On the left hand side the IP address of the website is stated along with what pages we visited; in this case we've only visited the **root page** which is the top level directory/entry point of the website
- Towards the bottom of the screen you can see the body of the HTTP Request along with the values for each header and the HTTP Response

The next step is to edit the HTTP request and change the **User-Agent** header by using the **Repeater** module:
- Before modifying the HTTP request I first sent the original HTTP request to the Repeater module by right clicking on the request and clicking **Send to Repeater**

Once the request is loaded into the Repeater module I chose a random name and modified the **User-Agent** header to reflect that and sent the request:
<img width="1920" height="1045" alt="repeater_1" src="https://github.com/user-attachments/assets/37da63bf-9236-4d0a-9d45-53486cb790da" />


- The reason why **C** worked as the User-Agent is by nature of this challenge (I had to look this up but my best guess is that since we already have **Agent R**, I would assume the codename here would be letters specifically capital letters)
- Keep in mind changing the User-Agent header to **R** gives us the following response which further supports my claim

<img width="1920" height="1044" alt="repeater" src="https://github.com/user-attachments/assets/db8f1252-812b-4ac2-9dbf-71cd9116ae74" />



Changing the User-Agent sent the request to the `agent_C_attention.php` which I had to check out:
<img width="353" height="162" alt="hidden_page" src="https://github.com/user-attachments/assets/bc230e91-8e78-4a8a-8f05-54de71c2bd14" />
<img width="1920" height="927" alt="agent_c" src="https://github.com/user-attachments/assets/728c4017-ba34-4eac-95b2-00fa45e6d9bb" />
- **Agent R** tells **chris** about the deal that's going on along with mentioning something about **agent J** but the most important thing I took away from this message was that **chris's** password was weak

## Brute Forcing 🔓
With this in mind, **chirs** could be a potential username on the target machine so I attempted to brute  force his `ssh` password using `hydra`:
```
hydra -l chris -P /usr/share/wordlists/rockyou.txt ssh://10.66.161.225
```
<img width="951" height="548" alt="hydra_attempt" src="https://github.com/user-attachments/assets/4c0cf72c-69e6-42ee-88cb-1abc3aab1f27" />


After some time I realized this wasn't working so instead I attempted to brute force the `ftp` password using the same method:
```
hydra -l chris -P /usr/share/wordlists/rockyou.txt ftp://10.66.161.225
```
<img width="939" height="560" alt="hydra_attempt2" src="https://github.com/user-attachments/assets/9eeac753-1881-454e-9c1b-c57cca387e9b" />

After a successful brute force attempt, I logged into the `ftp` server using the provided credentials:
```
Username: chris
Password: crystal
ftp chris@10.66.161.225
```
<img width="920" height="282" alt="ftp_login" src="https://github.com/user-attachments/assets/89a24037-4bfb-4eb8-a623-0b9067b85d06" />

Once inside the `ftp` server I listed out the files and found 3 files on the server:
<img width="954" height="223" alt="ftp_files" src="https://github.com/user-attachments/assets/cfca7529-e32e-4fca-a9f4-570254c2f707" />

After copying them over to my computer, the first file I took a look at was `To_agentJ.txt`:
<img width="1920" height="228" alt="agentJ" src="https://github.com/user-attachments/assets/af79f9ad-a572-4c73-a92b-63669e4af00b" />
- Agent C is telling Agent J that their login credentials is stored inside of a image within their directory
- So I assumed one of the 2 files I found in the `ftp` server contains the login password

Using a tool like `stegseek`, I cracked the passphrase of the image and obtained the hidden message:
<img width="1295" height="515" alt="stegseek_crack" src="https://github.com/user-attachments/assets/3d3ab138-30de-4cbe-a8d5-499fbfd80eb4" />


Now I had a username and a password:
```
Username: james
Password: hackerrules!
```

Upon logging in with those credentials I was able to get the user flag:
<img width="963" height="861" alt="user_flag" src="https://github.com/user-attachments/assets/f6797e7e-bc28-4153-89fd-92aec9413682" />
