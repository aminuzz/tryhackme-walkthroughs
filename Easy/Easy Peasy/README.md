# Easy Peasy 🕶️
This challenge had me use tools like **Nmap** and **GoBuster** to locate a hidden directory on a target machine and answering some questions based on the results I found. From there we can execute our privileges through a vulnerable **cron** job to get the final flag.



# Step 1 **Nmap Enumeration**
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

