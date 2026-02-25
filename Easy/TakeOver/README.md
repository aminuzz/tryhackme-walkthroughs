# TakeOver 🫅
This challenge focuses on subdomain enumeration against a target website in order to identify subdomains to uncover the flag.

In this case, the challenge tells us to add the IP of the machine to ``/etc/hosts`` on our attack machine.


### What is ``/etc/hosts``?
The ``/etc/hosts`` file is a plain text operating system file that is used to map hostnames to IP addresses on our local machine. The computer will look through this file first before going through network DNS servers, allowing users to redirect domains, block websites locally, or test a website before deploying it. ``/etc/hosts`` is the location for Linux machines but for Windows it is located in ``C:\\Windows\System32\drivers\etc\hosts``. 

For example on **Windows**:
<img width="1919" height="1034" alt="image" src="https://github.com/user-attachments/assets/bf1d1985-830b-4a20-9c2e-f9e6f86dd305" />


On a **Kali Linux** machine:
<img width="1461" height="915" alt="image" src="https://github.com/user-attachments/assets/a5271053-5385-4113-9cd9-0da0458ed15d" />

Another way to demonstrate this is if we try to open the URL that the challenge provided us with. it'll give us an error stating that the site cannot be reached:
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/6d65d01b-b531-4608-a1bb-6a2e14432d42" />


But when accessing the website by the IP address, we have no issues:
<img width="1916" height="1077" alt="image" src="https://github.com/user-attachments/assets/d81d3e94-53e7-421b-949a-73ceb7a83064" />

# Step 1 (Editing ``/etc/hosts``) 📓
Since this challenge had already stated to add the target IP address into ``/etc/hosts``, that was the first thing I did.
- In my case I was using a **Kali Linux** virtual machine and **Vim** as my text editor but the same process can be done on a Windows machine by simply opening the file in **File Explorer** and using **Notepad** to edit the file.
- It is also important to note that if you're doing these challenges on your local machine, you must set up **OpenVPN** in order to access these machines. You can find the link for the **OpenVPN** setup [here](https://tryhackme.com/access).

Once we are connected to the network, we can start editing ``/etc/hosts``. 


I first edited ``/etc/hosts`` and verified it by accessing the website by its URL name:
![Gif10](https://github.com/user-attachments/assets/ffc45aab-9a46-4612-a713-682d33bbcecd)
- Now our machine can successfully acess the website via its URL

# Step 2 (Enumeration) 🔎
Now we can start the enumeration process:
- The challenge had specifically mentioned we had to uncover hidden **subdomains** so I used **Gobuster** to help me with this process.

I initially tried to use **Gobuster** in ``dns`` mode but I got no matching hits:
![Gif11](https://github.com/user-attachments/assets/73c11bdf-3eec-4d65-a49f-43bf9194251a)


After some research and going through all of the default lists in ``/usr/share/wordlists``, I found and installed a Github repo that contained an even larger set of security lists that aren't built into Kali. 
- You can find the repo by clicking this link [SecLists](https://github.com/danielmiessler/SecLists?tab=readme-ov-file)


Then after some trial and error I ran **Gobuster** in ``vhost`` mode and found 2 subdomains:
![Gif12](https://github.com/user-attachments/assets/f8c6d71b-144b-4f70-aae9-6fdf462b5b03)
- But before we can visit these subdomains we need to make sure we update ``/etc/hosts`` so we can access them without any issues.


# Step 3 (Looking through subdomains)
First I checked out ``blog.futurevera.thm``:


Then I checked out ``support.futurevera.thm``:


