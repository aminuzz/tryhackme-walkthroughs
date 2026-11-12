# OhSINT -- TryHackMe Writeup

**Difficulty:** Easy
**Category:** OSINT (Open Source Intelligence)

## Introduction
OhSINT is an easy-rated TryHackMe room that tests OSINT skills rather than any technical exploitation. Starting from a single image, the goal is to answer a series of questions about the person behind it: their username, location, Wi-Fi network, email, travel, and even a password. Each answer is found by extracting metadata and pivoting across publicly available sources, a reminder of how much of a person's digital footprint can be reconstructed from one small starting point.

The image provided is shown below:
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/13176b70-e8f0-44fb-9570-575a6e775f53" />

## Question 1: What is this user's avatar of?
The image alone gives nothing away about who created it, so the first step was to extract the file's metadata with `exiftool`:
<img width="702" height="343" alt="image" src="https://github.com/user-attachments/assets/e954a9ab-4afc-4c22-92b3-ab6db3115304" />

`exiftool` reads, writes, and edits the metadata embedded inside a file, the information about the file itself, which can include:
- Camera model
- Date the photo was taken
- GPS location
- The author of the file

The metadata revealed the owner's name: OWoodflint. A basic Google search for that username turned up a matching X (Twitter) profile:
<img width="1007" height="775" alt="image" src="https://github.com/user-attachments/assets/af72b524-1733-4ba7-8a98-8699a5b0222f" />

The profile picture is a cat, which is the answer:
<img width="1582" height="144" alt="image" src="https://github.com/user-attachments/assets/39ffab27-04eb-4b10-be14-ee19d05713ef" />

## Question 2: What city is this person based in?
The same username also has a GitHub account. Opening the `people_finder` repository revealed the answer in its first line:
<img width="1011" height="492" alt="image" src="https://github.com/user-attachments/assets/5f76785d-7ad9-4440-9ffd-9f3e382cca90" />

The city is London:
<img width="1561" height="113" alt="image" src="https://github.com/user-attachments/assets/d69cecd0-662d-4a9a-88dd-963dc2015af1" />

## Question 3: What is the SSID of the WAP he connected to?
Returning to the user's X account:
<img width="1007" height="775" alt="image" src="https://github.com/user-attachments/assets/af72b524-1733-4ba7-8a98-8699a5b0222f" />

The first post contains the BSSID of the user's Wi-Fi network, the unique MAC address of the router. Searching that BSSID on WiGLE.net returns the network's SSID, the human-readable network name:
<img width="1007" height="500" alt="image" src="https://github.com/user-attachments/assets/4d2e256a-2c5f-4a50-8182-13d38c752dd6" />

WiGLE.net is a large crowdsourced database of wireless networks (Wi-Fi, cellular, Bluetooth) mapped worldwide. In OSINT work it is commonly used to geolocate networks, find rogue access points, and perform reconnaissance.

The SSID is Unilever Wifi:
<img width="1569" height="114" alt="image" src="https://github.com/user-attachments/assets/8c51287c-e340-41e3-8a6f-8e6adaa0cfa4" />

## Question 4: What is his personal email address?
Back on the user's GitHub account, the email address is exposed:
<img width="1909" height="934" alt="image (2)" src="https://github.com/user-attachments/assets/7fceaca9-5643-46f4-a3e1-f0412d090201" />

The email is OWoodflint@gmail.com.

## Question 5: What site did you find this on?
This refers to the previous question, so the answer is GitHub.

## Question 6: Where has he gone on holiday?
A quick Google search surfaced a blog post made by the user:
<img width="1917" height="874" alt="image (3)" src="https://github.com/user-attachments/assets/75b68c00-09d1-45c6-a59e-e8553c0b11a0" />

In the post, he mentions being on vacation in New York.

## Question 7: What is his password?
For questions like this, a password is often hardcoded somewhere in the page, so the blog post's source code was the obvious place to check:
<img width="1914" height="891" alt="image (4)" src="https://github.com/user-attachments/assets/b0c1f5d4-ab1a-4d45-bc2e-8a38fbeb3d18" />

The assumption was correct. The password was left in the source: pennYDr0pper.!

## Lessons Learned
- **Metadata is a starting pistol.** A single username pulled from the image's EXIF data with `exiftool` was the pivot point for the entire investigation.
- **Cross-referencing accelerates everything.** The same username tied together X, GitHub, a blog, and WiGLE. Reusing one identifier across platforms is how a full profile gets built so quickly.
- **Small data points are pivots.** A username, a BSSID, a blog post, each individually minor, chained into location, network, email, and credentials.
- **SSID and BSSID are not the same thing.** Knowing that the post leaked the BSSID (the router's MAC) and that WiGLE maps it back to the SSID was the key distinction in Question 3.
- **OSINT is critical thinking, not hacking.** The room rewards patience and connecting dots across sources far more than any single tool, and it is a stark reminder of how easily a careless digital footprint can be traced.

## Tools Used
- `exiftool`
- Google
- GitHub
- WiGLE.net
- Browser source inspection
