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
