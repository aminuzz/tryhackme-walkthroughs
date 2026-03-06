# Easy Peasy 🕶️
This challenge had me use tools like **Nmap** and **GoBuster** to locate a hidden directory on a target machine and answering some questions based on the results I found. From there we can execute our privileges through a vulnerable **cron** job to get the final flag.



# Task 1 **Nmap Enumeration**
The first couple of questions are about the ports/services that are running on the machine. To answer the following questions, I used **Nmap** to perform the initial enumeration of the target.



## Question 1
The first question states:
- How many ports are open?

To check for open ports on a machine, we can run **Nmap** to scan for all open TCP ports on the target machine:
```Bash
nmap -sV -p1-65535 10.66.143.198
```
- ``-sV`` returns the service and the version that is running on that port
- ``-p1-65535`` scans all ports from 1 through 65535. By default, **Nmap** scans the top 1000 TCP ports but to get a thorough scan of the machine, scanning all possible ports is the appropriate choice of action.
![Gif16](https://github.com/user-attachments/assets/bf20b90c-5652-4dee-b968-9e57a5169d60)

- In total there were **3** ports that were open.
<img width="836" height="68" alt="image" src="https://github.com/user-attachments/assets/3d3ce26a-298c-4e58-a3c4-8caae2fa6107" />


## Question 2
The second question asks:
- What is the version of nginx?
<img width="1464" height="901" alt="image" src="https://github.com/user-attachments/assets/11816e88-d96a-4297-b8d6-85578fa13301" />

Since I ran the **Nmap** scan with the ``-sV`` flag, it showed open ports along with the specific service and service version running on the port. In this case, on port 80 a **nginx** web server was running version **1.16.1**.

<img width="845" height="71" alt="image" src="https://github.com/user-attachments/assets/706e0689-56a9-4757-ae7c-d24d58acc198" />

## Question 3
The third question states:
- What is running on the highest port?
<img width="1475" height="914" alt="image" src="https://github.com/user-attachments/assets/9ed07283-70b3-4efa-a660-27439eb497d8" />

We can look at the **Nmap** scan and we can see that a **Apache web server** is running on port **65524**, which is the highest port that is open.

<img width="836" height="64" alt="image" src="https://github.com/user-attachments/assets/18ffadb9-334e-4067-a7c8-ab913f1e8040" />


# Task 2 **Gobuster Enumeration**
After our first enumeration with **Nmap**, the next couple of questions had me using **Gobuster** to enumerate the **nginx & Apache** web servers that were running on the target machine.


## Question 1
The first question of Task 2 states:
- Using Gobuster, find flag1

My thought process was to start enumerating the **nginx** server since that was the first thing I saw when I accessed it through the web browser:

<img width="1476" height="915" alt="image" src="https://github.com/user-attachments/assets/4db5ca87-c5c1-40d3-aadd-82f4ccbaec6a" />

Next I ran **Gobuster** in directory mode to look for hidden subdirectories in the web server:

```Bash
 gobuster dir -u "http://10.67.190.178" -w /usr/share/wordlists/dirb/big.txt -t 64     
 ```
![Gif17](https://github.com/user-attachments/assets/7655bb6c-5117-4ae8-b2fb-4648ad31962d)

- Gobuster located two hidden pages, ``hidden`` and ``robots.txt``

``robots.txt`` is a file that's placed within the root directory of a website that instructs search engine crawlers which pages or sections of a website to crawl (look through) or ignore:

<img width="1472" height="910" alt="image" src="https://github.com/user-attachments/assets/14e2bca0-a1b4-4e9b-89f1-a7e3ba7f7e50" />

What this means:
- ``User-Agent: *`` applies to all web crawlers
- ``Disallow: /`` tells crawlers they are not allowed to access anything on the site
- ``Robots Not Allowed`` just a comment/message

Now let's look at ``http://10.67.190.178/hidden/``:

<img width="1476" height="916" alt="image" src="https://github.com/user-attachments/assets/b6bceac0-085f-4644-a7a3-d353616b37b4" />

Now let's look at the HTML source code for this webpage:

<img width="1452" height="910" alt="image" src="https://github.com/user-attachments/assets/25f875f9-393b-480a-877a-9061575cd946" />

After looking through the HTML I decided to further enumerate the website following this url ``http://10.67.190.178/hidden/``:

![Gif18](https://github.com/user-attachments/assets/931047bf-6508-4570-bef2-a86103aec9ff)

- Now we can see that there is subdirectory within `/hidden` called `whatever`. This caught my attention so I opened the web page on my browser to investigate.

<img width="1475" height="922" alt="image" src="https://github.com/user-attachments/assets/f67febeb-f414-4a11-9761-7fc4619b5636" />

Nothing can be found on the web page so I checked the source code for the web page:

<img width="1470" height="921" alt="image" src="https://github.com/user-attachments/assets/9e7874ed-150b-4710-9f5b-03b31c187277" />

- There is a hidden paragraph element within the HTML that contains a Base64 string

We can decode the string using a tool like CyberChef to decode the text:

<img width="1919" height="938" alt="image" src="https://github.com/user-attachments/assets/1a2fd39b-8a62-40a0-b915-33a36a5b5d7e" />

- The resulting text is the answer to question 1

<img width="841" height="66" alt="image" src="https://github.com/user-attachments/assets/4f421237-a5d4-49c0-bc5e-5f7a6a0f661d" />


## Question 2
The second question states:
- Further enumerate the machine, what is flag 2?
Based on the earlier scan there was also a **Apache** web server running on port 65524 on the target machine which became my next target for investigation.

I ran the following **Gobuster** command:
```Bash
gobuster dir -u "http://10.65.188.233:65524" -w /usr/share/wordlists/dirb/common.txt -t 64
```
![Gif19](https://github.com/user-attachments/assets/6427909b-0542-4691-a330-8bacd42de968)
The result was that Gobuster found ``index.html`` and ``robots.txt``. After further investigation through ``robots.txt`` I noticed a MD5 hash in the **User-Agent** section. I then used [hashes.com](https://hashes.com/en/decrypt/hash) to decrypt the string to get the second flag:
<img width="1373" height="915" alt="image" src="https://github.com/user-attachments/assets/29de7362-b26c-438c-b933-3f4a27bc7f61" />
<img width="843" height="71" alt="image" src="https://github.com/user-attachments/assets/fc332d23-3a58-4108-b0b7-1c80e752418a" />


## Question 3
The third question states:
- Crack the hash with easypeasy.txt. What is the flag 3?

After looking through the source code on the Apache root web server, there is a string that contains the answer to question 3:
<img width="1479" height="919" alt="image" src="https://github.com/user-attachments/assets/b39fb2e3-6df7-4ec5-bfcf-d9e60a2414c2" />


## Question 4
The fourth question states:
- What is the hidden directory?

<img width="1479" height="918" alt="image" src="https://github.com/user-attachments/assets/21b4f6eb-5e7c-4c57-96bc-6f4f55b59572" />





In the highlighted section I noticed a hidden paragraph element with the following text:
- ``its encoded with ba....:ObsJmP173N2X6dOrAgEAL0Vu``

After some trial and error using CyberChef I was able to decode the text from **base62** to regular text. This revealed a hidden directory called ``/n0th1ng3ls3m4tt3r``.

<img width="842" height="77" alt="image" src="https://github.com/user-attachments/assets/42fa7b13-0ce8-4b5f-bccf-6b28bc938ca7" />

## Question 5
The fifth question states:
- Using the wordlist that was provided to you in this task crack the hash and what is the password?

Given we found the hidden directory I visited the subdirectory and I found the hidden hash buried in the source code of the web page:
<img width="1475" height="918" alt="image" src="https://github.com/user-attachments/assets/26774697-72a2-449d-9f54-01c234ed170b" />
- The challenge gives us a hint stating that this hash is a ``GOST`` hash

We can download the wordlist that the challenge provides us with along with saving the hash in a file. We can then use **JohnTheRipper** to crack the hash:
<img width="967" height="254" alt="image" src="https://github.com/user-attachments/assets/80d9b6ea-1a4e-4ea5-8682-be880690ff5f" />

After a successful run the password we get is:
``mypasswordforthatjob``
<img width="837" height="82" alt="image" src="https://github.com/user-attachments/assets/2018b18b-8d1d-4c60-b200-5895b70780a8" />


## Question 6
The sixth question states:
- What is the password to login to the machine via SSH?

At first I was confused on how I was supposed to obtain the SSH login password. Based on the previous question, the hash of the password was located in the source code of the subdirectory ``/n0th1ng3ls3m4tt3r``. I assumed for this question I had to take the same approach so my next focus of attention was to look for images that may contain data that could lead to the password.
<img width="1463" height="924" alt="image" src="https://github.com/user-attachments/assets/8598b40d-5166-42a9-9364-aa90db66f35e" />

- The previous question had the hash of the password in the background image so instead I focused on the image with the binary numbers. I saved this image to my PC for further investigation.
<img width="1471" height="905" alt="image" src="https://github.com/user-attachments/assets/6a30fe4c-fb0d-4bb0-b592-fd58f5870590" />

Attackers can hide information within an image using a process called **steganography**. My assumption was that the hint to the password had to be embedded within the image, so I used a built-in Kali tool called **steghide** to extract the information out of the image:
![Gif20](https://github.com/user-attachments/assets/0ec37ccd-e636-4c96-9459-f5e75050d0c8)
- During the extraction process I had to provide a password which was the password we cracked from the previous question
- The result was outputted to a file called ``secrettext.txt``
- We found a username called ``boring`` and a password that's in binary format

To find the password we can simply use CyberChef to convert the binary back into plaintext:
<img width="950" height="936" alt="image" src="https://github.com/user-attachments/assets/e7e4ceda-feba-4e4e-96ab-18f9c0cef1ed" />

<img width="834" height="77" alt="image" src="https://github.com/user-attachments/assets/04fb9969-24e3-4c2f-9877-f208a0f8de68" />


## Question 7
The seventh question states:
- What is the user flag?

After finding the SSH login credentials from the previous question, I used the credentials to login to the machine and luckily the flag was in the home directory. The text was encoded with the **ROT13** cipher so using CyberChef I was able to obtain the flag:
<img width="807" height="432" alt="image" src="https://github.com/user-attachments/assets/132bd068-cc99-4a2b-86fb-7965d2fb844a" />
<img width="958" height="945" alt="image" src="https://github.com/user-attachments/assets/ab1aa9d2-8d0f-4d48-8094-37113f52a2b6" />

<img width="844" height="77" alt="image" src="https://github.com/user-attachments/assets/03fa2171-f84d-40f4-b8ce-eddf3cf9a60a" />

## Question 8
The final question states:
- What is the root flag?

The description of the challenge states to **escalate our privileges through a vulnerable cronjob**. A cronjob is a task that is automated by the OS to execute at a specific time. These tasks include clearing folders, downloading media through a script, etc. These jobs can either be system wide (where it affects all users) or user specific. 

After looking through the system wide cronjobs (in ``/etc/crontab``) one cronjob stood out to me:
<img width="958" height="945" alt="image" src="https://github.com/user-attachments/assets/f8647505-139e-4482-9667-1ecca717f2af" />

<img width="543" height="73" alt="image" src="https://github.com/user-attachments/assets/b1be9043-1fae-4f8f-9135-4d73546512a9" />

Anything within this cronjob will run as root, so I inserted a reverse shell into ``.mysecretcronjob.sh`` and used ``netcat`` on my machine to get an interactive shell with root privileges:
<img width="936" height="706" alt="image" src="https://github.com/user-attachments/assets/6d905006-886c-4aea-8c82-048f55b9dbde" />

Once my listener received the connection on port 1000, I verified I was the root user and found the flag in ``.root.txt``:
<img width="623" height="483" alt="image" src="https://github.com/user-attachments/assets/60262ee5-6c4e-4670-ab68-b03dab774776" />
<img width="848" height="88" alt="image" src="https://github.com/user-attachments/assets/2a19b76f-58c7-4f6d-849f-e9390b896a45" />

# Lessons learned 📚
- **Thorough enumeration is critical**. Running a full port scan with ``nmap -p1-65535`` helped identify services running on non-standard ports, such as the Apache server running on port 65524.
- **Web enumeration often reveals hidden content**. Using Gobuster to brute-force directories uncovered hidden paths like ``/hidden`` and ``/whatever``, which led to discovering encoded data and additional clues.
- **robots.txt** can reveal useful information during reconnaissance. Even though it is meant for search engine crawlers, it can expose hidden paths or hints that attackers can investigate further.
- **Source code inspection is important**. Viewing the HTML source revealed hidden elements and encoded strings that were not visible on the page itself.
- **Understanding common encoding schemes is useful in CTFs**. Techniques such as Base64 decoding, Base62 decoding, ROT13, and binary-to-text conversion were necessary to uncover multiple flags throughout the challenge.
- **Password cracking tools are essential in penetration testing**. Using John the Ripper with a provided wordlist allowed the GOST hash to be cracked successfully.
- **Steganography can hide sensitive data in images**. Tools like steghide can extract hidden data embedded within images when the correct passphrase is supplied.
- **Cronjobs can lead to privilege escalation**. Misconfigured cronjobs that run scripts as root can be exploited by modifying the script to execute malicious commands such as a reverse shell.
- **Privilege escalation often comes from misconfigurations rather than vulnerabilities**. In this case, writable cronjob scripts allowed execution of commands with root privileges.

# Tools used 🧰
- Nmap
- Gobuster
- CyberChef
- John the Ripper
- Steghide
- Netcat

