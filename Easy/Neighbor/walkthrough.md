# Neighbor -- TryHackMe Writeup

**Difficulty:** Easy
**Category:** Web Exploitation / Broken Access Control

## Introduction
Neighbor is an easy-rated TryHackMe room built around a single, very common web vulnerability: an **Insecure Direct Object Reference (IDOR)**. The target is a web application called **Authentication Anywhere**, which lets users log in from anywhere they like. Rather than relying on an exploit chain, the room tests whether you can recover leaked credentials from the page source, log in as a low-privileged user, and then recognize that the application trusts a user-supplied URL parameter to decide whose profile to display, allowing you to "peep your neighbor's profile" and reach the admin account that holds the flag.

<img width="956" height="944" alt="image" src="https://github.com/user-attachments/assets/2a2e63eb-435c-4311-92af-3bf50305f804" />

### Objective
The objective is to recover the flag from the "neighbor's" login page.

## Enumeration
### Source Code Review
A note at the bottom of the login page stood out as a hint worth following up on:
<img width="217" height="33" alt="image" src="https://github.com/user-attachments/assets/22c3671b-f546-4d26-9705-f8e0e9729899" />

Pressing `Ctrl + U` opens the page source, which is a reliable first place to look for hardcoded secrets, comments, or endpoints that aren't visible in the rendered UI. Inspecting it revealed hardcoded credentials for a **guest** user:
<img width="1918" height="915" alt="Screenshot 2026-03-25 202011" src="https://github.com/user-attachments/assets/29cdd8e1-a9b3-469a-a1ee-14fb8cd6f242" />

## Gaining Access
### Logging In as Guest
Using the recovered credentials, logging in as `guest` led to the following page:
<img width="1916" height="916" alt="image" src="https://github.com/user-attachments/assets/3d5cb07e-520f-4a97-816c-82c64091161e" />

The key detail is in the URL: the page that's displayed is controlled by a `user` parameter, which was set to `guest`:
<img width="494" height="35" alt="Screenshot 2026-03-25 202725" src="https://github.com/user-attachments/assets/9b952ce6-56cf-473b-8f02-f42eaca2c889" />

## Exploitation
### Background
The application's own warning, *"Try not to peep your neighbor's profile"*, combined with a `user` parameter visibly controlling which profile is shown, pointed directly at an **IDOR** vulnerability. IDOR is a broken access control flaw where an application uses user-supplied input to reference an object directly, without verifying that the requesting user is actually authorized to access it. It commonly shows up through:
- URL parameter tampering
- Hidden field manipulation
- Object enumeration
- File path manipulation

In this case, the vulnerability is exploited through **URL parameter tampering**.

### Exploiting the IDOR
Changing the value of the `user` parameter in the URL granted access to other users' profiles, with the welcome text updating to reflect whichever username was supplied:
![Gif27](https://github.com/user-attachments/assets/7fe92564-27a7-40eb-aa8a-0142e17fe4c3)

Since sensitive data like a flag would almost certainly live on a higher-privileged account, the logical target was the `admin` user, a profile a normal user should never be able to view:
![Gif28](https://github.com/user-attachments/assets/4414194f-9e9f-4375-b13d-b4c63c2bf6f8)

## Flag Capture
Setting `user=admin` exposed the admin profile and, with it, the flag, completing the room.

## Lessons Learned
- **Always inspect the source code.** Viewing the page source (`Ctrl + U`) exposed the guest credentials that bootstrapped the entire attack, which is a reminder that hardcoded secrets and developer comments are a routine first find.
- **Never trust user-controlled input.** The application relied on a `?user=` parameter to decide what to show without validating whether the logged-in user was allowed to see it, which is the root cause of the entire vulnerability.
- **IDOR doesn't need tooling.** No `gobuster`, no Burp, nothing but changing a single URL parameter and observing the response. Manual enumeration is a mindset as much as a toolset.
- **Privilege levels are a map.** Flags and sensitive data tend to live on higher-privileged accounts, so once a parameter-based access control flaw is confirmed, the `admin` account is the obvious thing to point it at.

### Key Takeaway
Broken access control is one of the most common and dangerous vulnerability classes. A single parameter change can completely bypass authorization when the application fails to enforce proper access checks on the server side.
