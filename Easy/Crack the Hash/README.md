# Crack the Hash 🔒
This challenge goes over basic level hash cracking. There are two levels, each with a couple of hashes that we need to crack. The first level contains some hashes without any salts while the second level 
ups the difficulty by adding salts to the hashes. 

## Quick Note
Because I've already worked on the challenge I made sure to delete the contents of John the Ripper's pot file to clear all of the cracked passwords from this challenge. For some of the questions, the files I save the hash will be different. 

# Level 1 
These are the questions and answers to the Level 1 questions:

### Question 1
The hash we have to crack is:
```text
48bb6e862e54f2a795ffc4e541caed4d
```

We can use tools like `hashid` or `hash-identifier` to identify the type of hash. For this example, I used the latter:
<img width="1475" height="917" alt="image" src="https://github.com/user-attachments/assets/3589641c-aa42-4c27-a7d9-c7238deaf69f" />

- The results indicate that the hash is most likely either an **MD5** hash or a **Windows Domain Cached Credentials** hash

We can save this hash to a file, in my case I saved it to a file called ``hashes.txt``, and use **John the Ripper** to crack it:

```Bash
john --format=Raw-MD5 --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

```

- ``--format=Raw-MD5`` specifies the format that **John** will use to crack the hash. This option is not required since John can detect the hash type in most cases.
- ``--wordlist=/usr/share/wordlists/rockyou.txt`` specifies the wordlist **John** will use to compute the candidate hashes and compare them to the target hash to find a match.
- ``hashes.txt`` contains our hash.

The cracked hash evaluates to:
<img width="800" height="162" alt="image" src="https://github.com/user-attachments/assets/cfb87da4-8379-46a7-9744-9916933f4ca9" />
<img width="1244" height="48" alt="image" src="https://github.com/user-attachments/assets/386de309-814f-494d-a758-1f2e47e2910a" />

### Question 2
The next hash we have to crack is:
```text
CBFDAC6008F9CAB4083784CBD1874F76618D2A97
```

We can again use ``hash-identifier`` to determine the type of hash:
<img width="1462" height="815" alt="image" src="https://github.com/user-attachments/assets/e2f85803-903d-4ece-a31c-4c61744c6c14" />
- The results indicate that this hash is most likely a **SHA-1** hash or a **MySQL SHA-1** hash

We can save this hash to ``hash.txt`` and use **John the Ripper** to crack it:

 ```Bash
john --format=Raw-SHA1 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

The cracked hash evaluates to:
<img width="975" height="192" alt="image" src="https://github.com/user-attachments/assets/8411b2b3-2960-4a2e-904c-7752efd35809" />
<img width="1247" height="50" alt="image" src="https://github.com/user-attachments/assets/703f94ee-7157-4bcc-a10e-897639a63fb1" />


### Question 3
The next hash we have to crack is:
```text
1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032
```

Using ``hash-identifier`` the hash is most likely a **SHA-256** hash:
<img width="832" height="548" alt="image" src="https://github.com/user-attachments/assets/f3a8a50f-cdd4-4b0e-be24-13c365ead28c" />


Using this hash format with **John the Ripper**, the cracked hash evaluates to **letmein**:

<img width="665" height="52" alt="image" src="https://github.com/user-attachments/assets/04ffab81-3683-4a42-8466-62ac69f48462" />

### Question 4
The next hash we have to crack is:
```text
$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom
```

Using ``hashid`` I found that this format is likely a **Blowfish (BSD)** hash:
<img width="688" height="227" alt="image" src="https://github.com/user-attachments/assets/042387ac-2eb5-4299-bfab-e4c101a49977" />

These hashes are difficult to crack because they take a long time to crack. **John the Ripper** is **CPU-based** so it's optimized for cracking passwords on computers without powerful GPUs. Luckily my personal computer has a decently powerful GPU, so I used **hashcat** for this question:
<img width="1001" height="310" alt="image" src="https://github.com/user-attachments/assets/7f768b93-8cad-4e5b-a2e8-2740d67661d4" />
The syntax is explained as follows:
- ``--show`` tells **hashcat** to show the value of the cracked password
- ``-m 3200`` indicates the mode **hashcat** is using. Similar to the ``--format`` option for **John the Ripper**, it specifies what type of hash to crack. All of the hashes that are supported by **hashcat** are listed on [here](https://hashcat.net/wiki/doku.php?id=example_hashes).
- ``-a 3`` indicates the attack mode **hashcat** is going to use. This attack mode is called a **mask attack** in which we can specify a password pattern that **hashcat** will use to crack the password.
- ``?l?l?l?l`` is the mask that we specified. This mask covers all combinations of 4 lowercase character words.

The hash evaluates to the word **bleh**:

<img width="662" height="52" alt="image" src="https://github.com/user-attachments/assets/73a988d2-90c3-4c1b-a9e2-7e0cd72b8190" />


### Question 5
The final hash is:
```
279412f945939ba78ce0758d3fd83daa
```

I had no luck with trying to crack this password on my virtual machine and my personal computer due to a technical issue so instead I used a website called [crackstation.com](https://crackstation.net/) to do the job for me:

