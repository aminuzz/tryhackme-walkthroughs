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
to login for a more realistic workflow:
![Desktop 2026 02 22 - 17 23 03 01](https://github.com/user-attachments/assets/28607469-2a0e-4ced-9898-5088eb057fdb)

 
Since the challenge didn't provide us with the absolute paths of the files I ran the **find** command to list the
absolute path for each file provided:
```Bash
find / -type f \( -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiMO -o
-name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy \) 2>/dev/null
```
### Why ``2>/dev/null``?
This suppresses permission-denied errors while searching the entire filesystem, keeping the output clean and easy to read.
The command successfully located all of the files except ``bny0`` which becomes important for another question later down the
line.

![Gif2](https://github.com/user-attachments/assets/a8220999-5c2c-4f84-8a8e-eae9ccf630ea)


<img width="1469" height="880" alt="image" src="https://github.com/user-attachments/assets/51dc4a46-7309-493e-aa2f-28fdccd9cd25" />

Now with this in mind, we can move onto the questions.

# Question 1 (Identifying File Group Ownership) 📁
The first question states:
- Which of the above files are owned by 
the best-group group(enter the answer separated by spaces in alphabetical order)?

Using the previous **find** command with the **-exec** option, we can execute ``ls -l`` on each of the files to view
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
First, I saved all discovered file paths to a file like so:
![Gif4](https://github.com/user-attachments/assets/bf0fb6aa-7f8d-4cc2-888d-4811a5ae5266)


Then we can use **grep** with a valid regex expression to return the path of the file that contains a IP address:
![Gif5](https://github.com/user-attachments/assets/70a6ad27-7268-4c9c-bd4e-dc7a06ba6e60)

### Why ``-l``?
The ``-l`` flag only returns the file name (or path in this case) containing a match instead of printing the matching
text. 
This quickly identifies the file containing an IP address.
<img width="839" height="70" alt="image" src="https://github.com/user-attachments/assets/0c21f2d3-2cb5-4fe7-9321-64cf6bd6d2d7" />


# Question 3 (Hash Analysis)
The third question states:
- Which file has the SHA1 hash of ``9d54da7584015647ba052173b84d45e8007eba94``?

To calculate the SHA1 hash of each file I ran the following command:
![Gif6](https://github.com/user-attachments/assets/19dec700-8a03-451c-a5ff-ab4be60c95ab)
- This computes and displays the hash for each file, allowing for a direct comparison with the value in question.

<img width="847" height="79" alt="image" src="https://github.com/user-attachments/assets/6e2d7af6-ea62-4731-a0f6-14cca467f3f0" />





# Question 4 (Line Count Investigation)
The fourth question states:
- Which file contains 230 lines?
To count lines I ran the following:
![Gif7](https://github.com/user-attachments/assets/c6c81324-bfc2-490c-8964-abaa752b3c4d)
- In this case, all visible files contained 209 lines. Since ``bny0`` was not located in my initial search, I assumed that it was the only file that contained 230 lines.
<img width="838" height="75" alt="image" src="https://github.com/user-attachments/assets/11b8647e-27b6-4c06-b27d-0902991e756f" />


# Question 5 (ID Number Check)
The fifth question states:
- Which file's owner has an ID of 502?
Using:
![Gif8](https://github.com/user-attachments/assets/519d2b94-1959-42b9-b47f-9c982a851a58)
<img width="835" height="76" alt="image" src="https://github.com/user-attachments/assets/807f2a88-0989-4705-92cb-2d7ddafd2957" />
- This allowed me to identify files owned by UID 502.

### Why ``ls -n``?
``-n`` displays numeric UID and GID instead of usernames and group names. 


# Question 6 (Execute Permissions) 
The last question states:
- Which file is executable by everyone?

We reviewed permissions using:
```Bash
ls -l
```
In Linux permissions:
```Bash
-rwxr-x-r-x
```


![Gif9](https://github.com/user-attachments/assets/d5002105-27c3-4e56-b15d-78acb162e3fc)
- In Linux file permissions, the final three characters represent the permissions assigned to others meaning users who are neither the file owner nor members of the file’s group. If the character ``x`` appears in this section, it indicates that the file is executable by all other users on the system.
<img width="836" height="69" alt="image" src="https://github.com/user-attachments/assets/78489de2-4f66-40b4-92bc-5aed8919b515" />

# Lessons Learned
- Efficient use of find with logical operators
- Practical application of -exec
- Suppressing errors with 2>/dev/null
- Regex pattern matching for IPv4 detection
- Hash verification using sha1sum
- Understanding Linux permission structure (owner, group, others)
- Using numeric UID/GID for deeper ownership analysis
