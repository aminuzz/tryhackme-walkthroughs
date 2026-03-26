# OhSINT 🕶️
This challenge had me work on my **OSINT** (Open Source Intelligence) skills. From one image, I had to answer a set of questions and find out about who was behind the image. The image is shown below:
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/13176b70-e8f0-44fb-9570-575a6e775f53" />


## Question 1
The first question is as follows:
- What is this user's avatar of?

From looking at the image itself, it's impossible to determine who was the behind the photo yet alone a avatar. This is where the magic of Kali Linux tools comes into play. Using `exiftool` we can extract some data about the image:

<img width="702" height="343" alt="image" src="https://github.com/user-attachments/assets/e954a9ab-4afc-4c22-92b3-ab6db3115304" />
`exiftool` is used to read, write, and edit metadata inside files which is the information about the file itself. This includes:
- Camera model
- The date the photo was taken
- GPS location
- The author (person) behind the image
- And much more
