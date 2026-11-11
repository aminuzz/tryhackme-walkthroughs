# Ninja Skills 🥷 -- TryHackMe Writeup

**Difficulty:** Easy
**Category:** Linux Fundamentals / Enumeration

## Introduction
Ninja Skills is an easy-rated TryHackMe room focused on practical **Linux enumeration**. Instead of exploiting a service, the room drops you onto a machine with a set of scattered files and asks a series of questions about them: who owns them, what permissions they carry, which ones contain specific data, and how they compare by hash or line count. It's a drill in navigating the file system quickly and accurately with core command-line tools, exactly the kind of muscle memory that matters in real system administration and security work.

## Initial Enumeration
The room centers on the following files:
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

TryHackMe provides a web-based terminal for this room, but it also offers SSH access. I used SSH for a more realistic workflow:
![Desktop 2026 02 22 - 17 23 03 01](https://github.com/user-attachments/assets/28607469-2a0e-4ced-9898-5088eb057fdb)

Since the room doesn't provide the absolute paths of the files, the first step was to locate them with `find`:
```bash
find / -type f \( -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiMO -o -name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy \) 2>/dev/null
```

### Why `2>/dev/null`?
Searching the entire file system as a normal user produces a flood of "permission denied" errors on directories you can't read. Redirecting `stderr` to `/dev/null` discards those errors, leaving only the clean list of results.

The command located every file except `bny0`, a detail that becomes important for a later question.

![Gif2](https://github.com/user-attachments/assets/a8220999-5c2c-4f84-8a8e-eae9ccf630ea)

<img width="1469" height="880" alt="image" src="https://github.com/user-attachments/assets/51dc4a46-7309-493e-aa2f-28fdccd9cd25" />

With the paths known, the individual questions can be answered.

## Question 1 — Identifying File Group Ownership
> Which of the above files are owned by the best-group group (answer separated by spaces, in alphabetical order)?

Extending the same `find` command with `-exec` runs `ls -l` on each located file, exposing its permissions and ownership:
```bash
find / -type f \( -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiMO -o -name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy \) 2>/dev/null -exec ls -l {} \;
```
![Gif3](https://github.com/user-attachments/assets/5aa7166b-5b30-46ac-b45a-0581af054fc8)
<img width="838" height="70" alt="image" src="https://github.com/user-attachments/assets/57dfb902-8758-4c9c-b59a-62bfb4616375" />

## Question 2 — Looking for IP Addresses
> Which of these files contains an IP address?

First, I saved all discovered file paths to a file for easier processing:
![Gif4](https://github.com/user-attachments/assets/bf0fb6aa-7f8d-4cc2-888d-4811a5ae5266)

Then `grep` with an IPv4 regex returned the path of the file containing a match:
![Gif5](https://github.com/user-attachments/assets/70a6ad27-7268-4c9c-bd4e-dc7a06ba6e60)

### Why `-l`?
The `-l` flag makes `grep` print only the name (or path) of the file containing a match, rather than the matching line itself, which is exactly what's needed to identify *which* file holds the IP.
<img width="839" height="70" alt="image" src="https://github.com/user-attachments/assets/0c21f2d3-2cb5-4fe7-9321-64cf6bd6d2d7" />

## Question 3 — Hash Analysis
> Which file has the SHA1 hash of `9d54da7584015647ba052173b84d45e8007eba94`?

Computing the SHA1 hash of each file allows a direct comparison against the target value:
![Gif6](https://github.com/user-attachments/assets/19dec700-8a03-451c-a5ff-ab4be60c95ab)
<img width="847" height="79" alt="image" src="https://github.com/user-attachments/assets/6e2d7af6-ea62-4731-a0f6-14cca467f3f0" />

## Question 4 — Line Count Investigation
> Which file contains 230 lines?

Counting lines across the files showed that every *located* file contained 209 lines. Since `bny0` was the one file the initial `find` couldn't reach, it was the logical candidate for the odd count of 230:
![Gif7](https://github.com/user-attachments/assets/c6c81324-bfc2-490c-8964-abaa752b3c4d)
<img width="838" height="75" alt="image" src="https://github.com/user-attachments/assets/11b8647e-27b6-4c06-b27d-0902991e756f" />

## Question 5 — ID Number Check
> Which file's owner has an ID of 502?

Listing with numeric IDs reveals the owning UID directly:
![Gif8](https://github.com/user-attachments/assets/519d2b94-1959-42b9-b47f-9c982a851a58)
<img width="835" height="76" alt="image" src="https://github.com/user-attachments/assets/807f2a88-0989-4705-92cb-2d7ddafd2957" />

### Why `ls -n`?
The `-n` flag displays numeric UID and GID instead of resolving them to usernames and group names, which is what lets you match against the ID of 502 directly.

## Question 6 — Execute Permissions
> Which file is executable by everyone?

Reviewing permissions with `ls -l`, the relevant part of a permission string like `-rwxr-xr-x` is the final three characters:
![Gif9](https://github.com/user-attachments/assets/d5002105-27c3-4e56-b15d-78acb162e3fc)

In Linux permissions, the last three characters represent permissions for **others**, users who are neither the owner nor members of the file's group. An `x` in that section means the file is executable by everyone on the system.
<img width="836" height="69" alt="image" src="https://github.com/user-attachments/assets/78489de2-4f66-40b4-92bc-5aed8919b515" />

## Lessons Learned
- **`find` with logical operators scales.** One command located a dozen scattered files system-wide, and chaining `-exec` onto it turned location into inspection in a single pass.
- **Absence is a signal.** The one file `find` couldn't reach (`bny0`) turned out to be the answer to the line-count question. A gap in results is information, not just noise.
- **Know your flags.** `grep -l`, `ls -n`, and `2>/dev/null` each changed the output into exactly the shape the question needed. Small flags save large amounts of manual filtering.
- **Permission strings are read in thirds.** Owner, group, others, reading the last triad is how you answer "executable by everyone" at a glance.

## Tools Used
- `find`
- `grep`
- `sha1sum`
- `ls` (`-l`, `-n`)
- `wc`
