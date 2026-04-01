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

Upon further inspection, we can see that the owner of the file goes by the name of **OWoodflint**. Doing a basic Google search, you can find a X profile with that particular username:
<img width="1007" height="775" alt="image" src="https://github.com/user-attachments/assets/af72b524-1733-4ba7-8a98-8699a5b0222f" />

- The profile image of the user is a **cat**, which is the answer for question 1

<img width="1582" height="144" alt="image" src="https://github.com/user-attachments/assets/39ffab27-04eb-4b10-be14-ee19d05713ef" />

## Question 2
The second question is as follows:
- What city is this person is based in?

Going back to Google (our best friend when it comes to OSINT), we can see that this person also has a **Github** account under the same username. Clicking on the `people_finder` repository we get the following:

<img width="1011" height="492" alt="image" src="https://github.com/user-attachments/assets/5f76785d-7ad9-4440-9ffd-9f3e382cca90" />

- The first sentence contains our answer to this question, **London**.
<img width="1561" height="113" alt="image" src="https://github.com/user-attachments/assets/d69cecd0-662d-4a9a-88dd-963dc2015af1" />


## Question 3
The third question states:
- What is the SSID of the WAP he connected to?

Let's go back to the user's X account:
<img width="1007" height="775" alt="image" src="https://github.com/user-attachments/assets/af72b524-1733-4ba7-8a98-8699a5b0222f" />
- The first post contains the **BSSID** of the user's Wi-Fi network which is the unique MAC address of the Wi-Fi router that the user has for their network.

Now with a simple search on **Wiggle.net** we can search up the user's **SSID**, which is the human readable name of the Wi-Fi network:
<img width="1007" height="500" alt="image" src="https://github.com/user-attachments/assets/4d2e256a-2c5f-4a50-8182-13d38c752dd6" />
- **Wiggle.net** is a website which stores a massive crowdsourced database of wireless networks (Wi-Fi, cellular, Bluetooth) that can be found worldwide.
- Especially within the **OSINT** field, it's a tool that security professionals use to analyze networks, find rogue access points, and do reconnaissance. 

The answer for this question is **Unilever Wifi**:
<img width="1569" height="114" alt="image" src="https://github.com/user-attachments/assets/8c51287c-e340-41e3-8a6f-8e6adaa0cfa4" />


## Question 4
The fourth question states:
- What is his personal email address?

Let's go back to the user's Github account:
<img width="1909" height="934" alt="image (2)" src="https://github.com/user-attachments/assets/7fceaca9-5643-46f4-a3e1-f0412d090201" />

The user's email account is **OWoodflint@gmail.com**

## Question 5 
The fifth question states:
- What site did you find this on?

This is referring to the previous question so the answer would be **Github**.


## Question 6
The sixth question states:
- Where has he hone to holiday?

Doing a quick Google search I was able to find a blog post that he made:
<img width="1917" height="874" alt="image (3)" src="https://github.com/user-attachments/assets/75b68c00-09d1-45c6-a59e-e8553c0b11a0" />

In the post he says that he's in **New York** on vacation which is the answer to this question!


## Question 7
The final question states:
- What is his password?

For questions like these I immediately assume the password is hardcoded somewhere so I decided to check the source code on the blog post page:
<img width="1914" height="891" alt="image (4)" src="https://github.com/user-attachments/assets/b0c1f5d4-ab1a-4d45-bc2e-8a38fbeb3d18" />

Luckily my assumption was correct! The answer for this question is ****pennYDr0pper.!****.



# Lessons learned 📚

* Utilized tools like **exiftool** to find an identity behind an image and used Google to perform searches based on that identity
* Cross-referencing information from different websites significantly speeds up the investigation process when building a profile on someone
* Small pieces of data (like usernames, metadata, or posts) can act as pivots to uncover much larger amounts of information
* Publicly available information can reveal sensitive details if users are not careful about what they share online
* Understanding the difference between identifiers like **SSID** and **BSSID** is important when working with wireless OSINT
* Platforms like GitHub, blogs, and social media often unintentionally expose personal details such as emails or locations
* Viewing page source and inspecting code can reveal hidden information such as credentials or comments left by developers
* OSINT is less about hacking and more about thinking critically, being patient, and connecting the dots between different data points
* Always verify information from multiple sources to ensure accuracy and avoid false assumptions
* This challenge reinforced the importance of operational security (OPSEC) and how easily someone’s digital footprint can be traced
