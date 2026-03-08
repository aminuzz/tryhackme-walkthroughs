# Pickle Rick
This Rick and Morty themed CTF challenge requires you to exploit a web server and find three ingredients to help Rick make his potion and transform himself back into a human from a pickle.

This is what the website looks like:
<img width="1469" height="912" alt="image" src="https://github.com/user-attachments/assets/0cccff62-6f94-4bcf-a70f-6e7a89807d81" />

## Question 1
The first question states:
- What is the first ingredient that Rick needs?

I looked through the source code of the website and found something interesting:
<img width="1475" height="917" alt="image" src="https://github.com/user-attachments/assets/1e3cbca9-9a8f-479f-a389-1092a44d2cc2" />
- There's a note stating the username is ``R1ckRul3s`` however we do not know what this username can be used for and we do not have a password

My initial thought was to probe the website to find a hidden webpage that might contain a password and a webpage where I can use login credentials using **Gobuster**:

```
gobuster dir -u "http://10.67.165.164" -x html,js,txt,php,py,css -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -t 64
```

![Gif21](https://github.com/user-attachments/assets/e1fa72f7-2a62-4a43-9e87-a8308d6bd82f)

There were several pages of interest from the scan:
- ``portal.php`` (fowarded to ``login.php``)
- ``clue.txt``
- ``robots.txt``

I opened ``clue.txt`` directly through the browser to look for any information that would lead to the first ingredient:

<img width="1471" height="916" alt="image" src="https://github.com/user-attachments/assets/d3f45876-f02d-4ae3-bbf7-9aedc8dc36ee" />

- Note that the IP address is different because I got disconnected from the previous section

Based on the text it states to look around the file system for the other ingredient. My assumption was that this had to be for either the 2nd or 3rd hint so I moved on and looked through ``robots.txt``:

<img width="1467" height="913" alt="image" src="https://github.com/user-attachments/assets/06564a88-81d4-4328-90b3-f072bdf206d4" />

The text ``Wubbalubbadubdub`` didn't stand out to me in any sort of way and based on all of the other files I found, I assumed this was the password to some sort of login page. Luckily we found a login page from our initial scan so I used that and ``R1ckRul3s`` as the username to login:

![Gif22](https://github.com/user-attachments/assets/57a2440a-169a-440e-a450-a6ef0f6335df)

The web portal contained a command panel and it acted as a restricted terminal connected to the web server. The first thing I did was to list out the files that were in the directory I was in:
<img width="1476" height="919" alt="image" src="https://github.com/user-attachments/assets/b249a089-4857-46a9-8c8b-52dc297996c9" />

The first file caught my attention (``Sup3rS3cretPickl3Ingred.txt``) so I tried opening with ``cat`` but it didn't work because that command was disabled. I then used the ``less`` command to open the file which led to the answer for question 1:
![Gif23](https://github.com/user-attachments/assets/a0d6213e-bc07-421d-b15b-42c51d82548f)

<img width="839" height="73" alt="image" src="https://github.com/user-attachments/assets/6b4afb74-89cb-4702-9ef8-a47520c6a47c" />


## Question 2
Based on the clue that we got from ``clue.txt``, we should be able to find the other ingredients if we looked around the file system. Since this web server is an **Apache** web server, the root directory is ``/var/www/html`` so I assumed that was the directory we were in. I doubled checked by running ``pwd``:
<img width="1472" height="921" alt="image" src="https://github.com/user-attachments/assets/d67d9cfb-ab74-4271-a1c1-e2e16d7b023d" />

Now we know we're in the web server's root directory, I checked for the rest of the directories on the machine from ``/``:
<img width="1474" height="914" alt="image" src="https://github.com/user-attachments/assets/373e4522-b5e7-4555-a6b7-adffe6b21aef" />



