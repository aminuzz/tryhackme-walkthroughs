# Easy Peasy 🕶️
This challenge had me use tools like **Nmap** and **GoBuster** to locate a hidden directory on a target machine and answering some questions based on the results I found. From there we can execute our privileges through a vulnerable **cron** job to get the final flag.



# Task 1 **Nmap Enumeration**
The first couple of questions are about the ports/services that are running on the machine. To answer the following questions, I used **Nmap** to perform the initial enumeration of the target.



## Question 1
The first question states:
- How many ports are open?

To check for open ports on a machine, we can run a simple **Nmap** scan to scan for all open TCP ports on the target machine:
```Bash
nmap -sV -p1-65535 10.66.143.198
```
- ``-sV`` returns the service and the version that is running on that port
- ``-p1-65535`` scans all ports from 1 through 65535. By default, **Nmap** scans the top 1000 TCP ports but to get a throughough scan of the machine, scanning all possible ports is the appropriate choice of action.
![Gif16](https://github.com/user-attachments/assets/bf20b90c-5652-4dee-b968-9e57a5169d60)
- In total there were 3 ports that were open.
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
- Now we can see that there is subdirectory within `hidden` called `whatever`. This caught my attention so I opened the web page on my browser to investigate.
<img width="1475" height="922" alt="image" src="https://github.com/user-attachments/assets/f67febeb-f414-4a11-9761-7fc4619b5636" />

Nothing can be found on the web page so I checked the source code for the web page:
<img width="1470" height="921" alt="image" src="https://github.com/user-attachments/assets/9e7874ed-150b-4710-9f5b-03b31c187277" />
- 

