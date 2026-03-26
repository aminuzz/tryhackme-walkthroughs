# Neighbor
This challenge has us investigating **Authentication Anywhere**, a web application which allows users to login in from anywhere
they would like:
<img width="956" height="944" alt="image" src="https://github.com/user-attachments/assets/2a2e63eb-435c-4311-92af-3bf50305f804" />


### Objective 📌
The objective for this challenge is to find the flag from our "neighbor's" login page.


## Initial Steps 🪜
I noticed on the bottom of the login page there was some text stating the following:
<img width="217" height="33" alt="image" src="https://github.com/user-attachments/assets/22c3671b-f546-4d26-9705-f8e0e9729899" />
- Clicking `Ctrl + U` allows us to view the source code for the web page. There could be hard coded secrets that could lead us to the flag


After inspecting the source code, I found valuable information:
<img width="1918" height="915" alt="Screenshot 2026-03-25 202011" src="https://github.com/user-attachments/assets/29cdd8e1-a9b3-469a-a1ee-14fb8cd6f242" />
- We now have login credentials for a **guest** user. Let's use these credentials to log in.


## Guest User Login ➜]
After logging into the guest profile, we are greeted with the following page:
<img width="1916" height="916" alt="image" src="https://github.com/user-attachments/assets/3d5cb07e-520f-4a97-816c-82c64091161e" />

In the screenshot above you can see that we are on the **guest** user page and in the url it confirms that by having the `user` parameter being equal to `guest`:

<img width="494" height="35" alt="Screenshot 2026-03-25 202725" src="https://github.com/user-attachments/assets/9b952ce6-56cf-473b-8f02-f42eaca2c889" />


## Grabbing the flag ✅
Based on the nature of this challenge and the website telling us **"Try not to peep your neighbor's profile"**, I immediately assumed
that this website had a **IDOR** (Insecure indirect object reference) vulnerability. This vulnerability is a **broken access control vulnerability**
where an application uses user-supplied input to directly access an object without validating if the user can access the requested object. Some common usage scenarios are the following:
- **URL Parameter Tampering**
- **Hidden Field Manipulation**
- **Object Enumeration**
- **File Path Manipulation**

In this case, we are dealing with an **IDOR** vulnerability that takes advantage of **URL Parameter Tampering**. In the video below as I change the value of the `user` parameter, we gain access to other user pages:
![Gif27](https://github.com/user-attachments/assets/7fe92564-27a7-40eb-aa8a-0142e17fe4c3)
- As you can see, changing the value of the `user` parameter updates the welcome text to include the name of the user that we specified.

The most likely user that might have the flag should be the `admin` user because in any application, a normal user should not be able to view the admin page of the application. Let's take a look:
![Gif28](https://github.com/user-attachments/assets/4414194f-9e9f-4375-b13d-b4c63c2bf6f8)

And there we have it, the flag for the challenge.


## Lessons Learned 🧩

- **Always inspect the source code**  
  Viewing the page source (`Ctrl + U`) can reveal hidden information such as hardcoded credentials, comments, or endpoints that are not visible in the UI.

- **Never trust user-controlled input**  
  Applications that rely on user-supplied parameters (like `?user=guest`) without proper validation are highly vulnerable to manipulation.

- **Understanding IDOR vulnerabilities is critical**  
  IDOR (Insecure Direct Object Reference) occurs when an application exposes internal objects (such as user accounts) without enforcing proper access control checks.

- **URL parameter tampering is a powerful technique**  
  Simply modifying parameters in the URL can sometimes grant access to unauthorized resources, especially in poorly secured applications.

- **Think like an attacker**  
  The hint “Try not to peep your neighbor's profile” was a direct clue. Learning to recognize these hints and apply logical assumptions (like checking the `admin` user) is key in CTFs.

- **Privilege levels matter**  
  Sensitive information (like flags or admin data) is often stored in higher-privileged accounts. Testing access to those accounts can lead to success.

- **Enumeration is not just tools, it's mindset**  
  Even without tools like `gobuster` or `burp`, manual enumeration (changing parameters, observing behavior) can uncover vulnerabilities.

---

## 🧠 Key Takeaway
This challenge reinforces that **broken access control is one of the most common and dangerous vulnerabilities**. Even a simple parameter change can completely bypass authentication if proper checks are not implemented.



