# Agent Sudo 👽
This TryHackMe room is a beginner-friendly challenge that covers a wider range of techniques than most entry-level rooms, making it a solid step up from basic enumeration and privesc. It begins with `nmap` reconnaissance to identify open ports and services, followed by manipulating the **User-Agent** header in an HTTP request to uncover a hidden agent codename. From there, `hydra` is used to brute-force FTP credentials, which surfaces a set of files containing hidden data. Extracting that data requires **steganography** tools to uncover concealed information buried inside the files, eventually yielding valid SSH credentials for an initial foothold. After capturing the user flag, privilege escalation revolves around a known `sudo` vulnerability (**CVE-2019-14287**) that allows a restricted user to run commands as root by manipulating the user ID — reinforcing a diverse set of skills including service enumeration, HTTP header manipulation, credential brute-forcing, steganography, and CVE-based privilege escalation along the way.



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
- 
