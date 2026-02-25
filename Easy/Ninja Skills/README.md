# Ninja Skills 🥷
This room focuses on developing practical Linux enumeration skills by analyzing file ownership, permissions, hashes, and content. The objective is to efficiently navigate the file system, identify key file attributes (such as owner, group, permissions, and executable status), analyze file contents for specific data (IP addresses, line counts, hashes), and apply common Linux command-line tools to extract relevant information. 
The challenge emphasizes speed, accuracy, and a strong understanding of core Linux concepts used in real-world system administration and cybersecurity environments.

# Initial Enumeration 🔍
The challenge had us investigating the following files:
- 8V2L
- bny0
- c4ZX
- D8B3
- FHl1
- oiMO
- PFbD
- rmfX
- SRSq
- uqyw
- v2Vb
- X1Uy

Tryhackme provides a web-based terminal for this room however they also provide a **ssh** login. I ended up using the latter 
to login:
![Desktop 2026 02 22 - 17 23 03 01](https://github.com/user-attachments/assets/28607469-2a0e-4ced-9898-5088eb057fdb)

 
Since the challenge didn't provide us with the absolute paths of the files I ran the **find** command to list the
absolute path for each file provided:
```Bash
find / -type f \( -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiMO -o
-name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy \) 2>/dev/null
```
- **2>/dev/null** is important here so we can have no error messages outputted onto our terminal

![Gif2](https://github.com/user-attachments/assets/a8220999-5c2c-4f84-8a8e-eae9ccf630ea)

As we can see the command has located all of the files except for **bny0**:
<img width="1469" height="880" alt="image" src="https://github.com/user-attachments/assets/51dc4a46-7309-493e-aa2f-28fdccd9cd25" />

Now with this in mind, we can move onto the questions.

# Question 1 (Identifying File Group Ownership) 📁
The first question states:
- Which of the above files are owned by 
the best-group group(enter the answer separated by spaces in alphabetical order)?

Using the previous **find** command with the **-exec** option, we can execute **ls -l** on each of the files to view
file permissions along with the file ownership of each file.
```Bash
find / -type f \( -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiMO -o
-name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy \) 2>/dev/null -exec ls -l {} \;
```
![Gif3](https://github.com/user-attachments/assets/5aa7166b-5b30-46ac-b45a-0581af054fc8)
<img width="838" height="70" alt="image" src="https://github.com/user-attachments/assets/57dfb902-8758-4c9c-b59a-62bfb4616375" />




# Question 2 (Looking for IP addresses) 🔍
The second question states:
- Which of these files contains an IP address?
We can utilize the **find** command again to locate the paths for the files and save them to a file like so:
![Gif4](https://github.com/user-attachments/assets/bf0fb6aa-7f8d-4cc2-888d-4811a5ae5266)


Then we can use **grep** with a valid regex expression to return the path of the file that contains a IP address:
![Gif5](https://github.com/user-attachments/assets/70a6ad27-7268-4c9c-bd4e-dc7a06ba6e60)
- The ``-l`` flag allows us to return the path of the file that contains the pattern

<img width="839" height="70" alt="image" src="https://github.com/user-attachments/assets/0c21f2d3-2cb5-4fe7-9321-64cf6bd6d2d7" />


# Question 3 (Hash Analysis)
The third question states:
- Which file has the SHA1 hash of ``9d54da7584015647ba052173b84d45e8007eba94``?

Again we can use the **find** command along with the ``-exec`` flag to execute the **sha1** command to calculate the SHA1 hash of each of the files:
![Gif6](https://github.com/user-attachments/assets/19dec700-8a03-451c-a5ff-ab4be60c95ab)


<img width="847" height="79" alt="image" src="https://github.com/user-attachments/assets/6e2d7af6-ea62-4731-a0f6-14cca467f3f0" />





# Question 4 (Line Count Investigation)
The fourth question states:
- Which file contains 230 lines?
We can use the **find** command with the ``-exec`` flag to run ``wc -l`` to count the amount of words in each file:
![Gif7](https://github.com/user-attachments/assets/c6c81324-bfc2-490c-8964-abaa752b3c4d)
- In this case, all of the files have 209 words so the one file that isn't present, ``bny0``, must have 230 words. 
<img width="838" height="75" alt="image" src="https://github.com/user-attachments/assets/11b8647e-27b6-4c06-b27d-0902991e756f" />


# Question 5 (ID Number Check)
The fifth question states:
- Which file's owner has an ID of 502?
We can use the **find** commmand along with the ``-exec`` flag to run ``ls -n``. This will list the files along with the numeric and group IDs for each file.
![Gif8](https://github.com/user-attachments/assets/519d2b94-1959-42b9-b47f-9c982a851a58)
<img width="835" height="76" alt="image" src="https://github.com/user-attachments/assets/807f2a88-0989-4705-92cb-2d7ddafd2957" />

# Question 6 (Execute Permissions
The last question states:
- Which file is executable by everyone?
We can run the same command we used in Question 1 to look for file permissions for all of the files:

# Lessons Learned
