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




# Question 2 (Looking for IP addresses)
The second question states:
- Which of these files contains an IP address?
We can utilize the **find** command again to locate the paths for the files and save them to a file like so:



![Gif4](https://github.com/user-attachments/assets/bf0fb6aa-7f8d-4cc2-888d-4811a5ae5266)


# Hash Analysis

# Line Count Investigation

# Permission and Executable Checks

# Key Concepts Reinforced

# Lessons Learned
