Step 1: Environment Preparation & VPN Setup
 
<img width="940" height="629" alt="image" src="https://github.com/user-attachments/assets/7bcd8777-68eb-4956-b56a-01f49cd65f70" />


sudo apt update && sudo apt install openvpn -y

•	sudo: Runs the command with administrative (root) privileges.

•	apt update: Refreshes the local package index to ensure the system knows about the 
latest versions of available software.

•	&&: A logical operator that tells the terminal to run the second command only if the first one finishes successfully.

•	apt install openvpn: Downloads and installs the OpenVPN client, which is the industry standard for connecting to remote lab environments.

•	-y: Automatically answers "yes" to any prompts, allowing for an unattended installation.

Step 2: Establishing the VPN Tunnel
 
<img width="836" height="628" alt="image" src="https://github.com/user-attachments/assets/179f6dad-efca-404c-88ad-7fa4bf92a9d1" />


After installing the client, the next goal is to initiate an encrypted tunnel between your local machine and the TryHackMe network. This gives your Kali Linux instance a virtual IP address on the same internal network as the target machine, allowing you to perform scanning and exploitation.

•	cd ~/Downloads: Changes the working directory to the Downloads folder, where the VPN configuration file was saved after being downloaded from the TryHackMe portal.

•  sudo openvpn [filename].ovpn:

•	sudo: Administrative privileges are required to modify network interfaces and routing tables.

•	openvpn: The binary that processes the configuration file.

•	ap-south-1-zulailanatashazulhelmi-regular.ovpn: Your specific configuration file containing the necessary keys, certificates, and remote server addresses for the connection.

Output

•	Version Check: Confirms OpenVPN 2.7.1 is running with OpenSSL 3.6.1.

•	Peer-to-Peer Connection: The line UDPv4 link remote: [AF_INET]35.154.230.92:1194 
shows your machine communicating with the TryHackMe gateway on port 1194.

•	Authentication: The VERIFY OK and VERIFY EKU OK lines confirm that the server’s SSL certificates are valid and the handshake is proceeding securely.

Step 3: Initial Reconnaissance (Network Scanning)
 
<img width="940" height="352" alt="image" src="https://github.com/user-attachments/assets/bb04a75f-e139-42c2-b8c9-4f40d2e0fa2a" />


The goal of this stage is Service Discovery. By scanning the target IP address (10.48.130.21), we aim to identify which ports are open and what services (and versions) are running on those ports. This information is vital for identifying potential entry points or vulnerabilities.

nmap -sV -sC -Pn 10.48.130.21

•	nmap: The Network Mapper tool used for vulnerability discovery and network auditing.

•	-sV: Service Version detection. It probes open ports to determine what software is running and its specific version number.

•	-sC: Runs Default Scripts. This uses the Nmap Scripting Engine (NSE) to check for common misconfigurations or known vulnerabilities associated with the discovered services.

•	-Pn: Treat all hosts as online. This skips the initial host discovery (ping) phase, which is useful if the target machine has a firewall blocking ICMP echo requests.

•	10.48.130.21: The target IP address for the Lian Yu machine.


Output

•	Port 21/tcp (Open): The File Transfer Protocol (FTP) service is running.

•	Service Version: The target is using vsftpd 3.0.2.

•	OS Insight: The Service Info: OS: Unix confirms the target is a Linux/Unix-based 
machine.

Step 4: Web Application Enumeration

 
<img width="940" height="390" alt="image" src="https://github.com/user-attachments/assets/92444f54-e360-4d6c-bb0b-9638ae2643f0" />


After identifying that a web server is likely running on the target (usually on Port 80), the goal is to perform a visual inspection of the landing page. This step is crucial for gathering "Open Source Intelligence" (OSINT) from the page content, which might reveal potential usernames, themes, or hidden clues planted by the machine creator.

•	URL Accessed: http://10.48.143.196

•	Tool Used: Web Browser (Firefox on Kali Linux).

•	Observation: The page displays a themed landing page titled "ARROWVERSE" with a description of Oliver Queen and the island of Lian Yu.

Step 5: Web Directory Brute-Forcing (Fuzzing)
 
<img width="829" height="491" alt="image" src="https://github.com/user-attachments/assets/647b8238-6122-45d1-ba58-7a5085c837db" />


Since the main landing page provided no direct links to sensitive areas, the goal of this stage is Content Discovery. By using a brute-force approach against the web server, we aim to find hidden directories or files that are not intended to be seen by public users but might contain sensitive information or further entry points.

gobuster dir -u http://10.48.143.196 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

•	gobuster: A high-performance tool used to discover hidden objects (directories/files) on websites.

•	dir: Specifies that we are in "directory" enumeration mode.

•	-u http://10.48.143.196: The target URL.

•	-w [path_to_wordlist]: Specifies the wordlist to use. In this case, directory-list-2.3-medium.txt is a standard, robust list containing common directory names.

Output

•	Identified Path: /island

•	Status: 301: This is a "Moved Permanently" redirect, which usually indicates that the directory exists and the server is redirecting the browser to include a trailing slash (/island/).

•	Significance: The word "island" matches the "Lian Yu" theme perfectly. This confirms we have found a valid area of interest for the next phase of the investigation.

Step 6: Analyzing the "Island" Directory
 
<img width="940" height="455" alt="image" src="https://github.com/user-attachments/assets/56c31630-39a3-41f9-b419-88fbbda010e8" />


After discovering the /island/ directory via brute-forcing, the goal is to inspect the content for actionable intelligence. In capture-the-flag (CTF) environments, these pages often contain "fluff" text that hides critical clues in plain sight or within the page's metadata.

•	URL Accessed: http://10.48.143.196/island/

•	Observation: The page displays a cryptic message: "You should find a way to Lian_Yu as we are planed. The Code Word is:"


Step 7: Source Code Analysis & Hidden Data Discovery
 
<img width="940" height="452" alt="image" src="https://github.com/user-attachments/assets/fa93be03-0b21-483c-b273-0b3291ecd998" />


When a web page appears to have missing information (like the "Code Word" in the previous step), the next logical phase of enumeration is Source Code Inspection. Developers or CTF creators often hide clues in HTML comments or use CSS to make text invisible to the end-user while remaining present in the underlying code.
The Action Breakdown

•	Method: Accessed the source code using the view-source: URI scheme in the browser.

•	URL: view-source:http://10.48.143.196/island/

Step 8: Targeted File Enumeration
 
<img width="940" height="604" alt="image" src="https://github.com/user-attachments/assets/afe7d54e-c767-47a3-9248-7ec7d343d410" />


After identifying a deeper directory (likely /2100/ which is often found by viewing the source or further fuzzing), the goal shifted to finding specific files within that path. By looking for non-standard file extensions, we can uncover configuration files, credentials, or "tickets" left behind by the system administrator.

gobuster dir -u http://10.48.143.196/island/2100/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .ticket

•	dir -u .../island/2100/: Sets the scope of the search to the specific sub-directory discovered in the previous phase.

•	-w ...: Uses the medium-sized directory list for comprehensive coverage.

•	-x .ticket: This is the crucial flag. It tells Gobuster to append the .ticket extension to every word in the wordlist. This is a targeted search for a specific file type rather than just a directory.

Output
The scan successfully identified a high-value file:

•	Identified File: green_arrow.ticket

•	Status: 200: The file is accessible and was successfully found on the server.

•	Size: 71: The small file size suggests it contains a short string, likely a password, a hash, or a further hint.

Step 9: Extracting the Encoded Token
 
<img width="940" height="304" alt="image" src="https://github.com/user-attachments/assets/ae153665-5731-41ea-99a3-c002e93ef93b" />


The goal was to view the contents of the green_arrow.ticket file found in the previous step to identify any usable credentials, passwords, or further instructions for moving deeper into the target system.

•	URL Accessed: http://10.48.143.196/island/2100/green_arrow.ticket

•	Content Found: * Text: "This is just a token to get into Queen's Gambit(Ship)"

•	Data: RTy8yhBQdscX

Step 10: Cryptographic Analysis & Decoding
 
<img width="940" height="395" alt="image" src="https://github.com/user-attachments/assets/70e782d5-5729-43a8-8ff3-a4d7b33b1e19" />


The goal of this phase was to convert the "token" found in the green_arrow.ticket file into a human-readable format. Since the string RTy8yhBQdscX did not appear to be a plaintext password, it was analyzed for common encoding schemes used to obfuscate data.

•	Tool Used: CyberChef (the "Cyber Swiss Army Knife").

•	Input: RTy8yhBQdscX

•	Recipe applied: From Base58

•	Output: !#th3h00d

Step 11: Gaining Access (FTP Exploitation)
 
<img width="940" height="670" alt="image" src="https://github.com/user-attachments/assets/b3a67708-ae4b-4e78-bd22-d60b24771a96" />


The goal was to leverage the credentials discovered through web enumeration (vigilante : !#th3h00d) to gain access to the FTP (File Transfer Protocol) service. Once inside, the objective is to enumerate the file system for sensitive documents, images, or hidden data that could lead to a system shell.

•	Command: ftp 10.48.143.196

•	Authentication: Successfully logged in as user vigilante.

•	ls: Listed the files in the current remote directory.

•	get Leave_me_alone.png: Downloaded an image file to the local Kali machine.

•	get Queen's_Gambit.png: Downloaded a second image file related to the "Ship" clue 
found earlier.

The FTP directory contains three interesting files:

1.	Leave_me_alone.png
2.	Queen's_Gambit.png
3.	aa.jpg

The presence of image files in a CTF often indicates Steganography. Steganography is the practice of hiding secret data (like SSH keys or passwords) inside non-secret files (like images).

Step 12: Advanced Post-Login Enumeration
 
<img width="940" height="801" alt="image" src="https://github.com/user-attachments/assets/dd732a08-043e-43e0-aec0-4c8ded4a0474" />


Once authenticated to the FTP server as vigilante, the goal shifted to a deeper inspection of the file system. By using more aggressive listing flags, a penetration tester can find hidden configuration files or "dotfiles" that are often ignored but frequently contain sensitive system information

•	ls -al:

•	-a: Shows all files, including hidden files (those starting with a dot .).

•	-l: Provides a long-form listing showing permissions, owners, and file sizes.

•	get .other_user: Downloads a hidden file discovered during the enumeration.

Step 13: Analyzing the Lore Dump for Credentials
 
<img width="940" height="743" alt="image" src="https://github.com/user-attachments/assets/7eb57501-778b-4dd3-899d-284c5ce60543" />


The goal was to read the contents of the exfiltrated hidden file .other_user. In themed CTFs, these large blocks of text usually serve to provide context for the next user account you need to compromise. We are looking for usernames, passwords, or hints about the next service to attack.

•	Command: cat .other_user

•	Content: A detailed biography of the character Slade Wilson (Deathstroke), covering his military history, his relationship with Adeline Kane, and his time with Wintergreen.

Step 14: Low-Level Hexadecimal and Metadata Analysis

 <img width="940" height="394" alt="image" src="https://github.com/user-attachments/assets/9de4b687-694c-481e-ba10-19979bb35b80" />

 
<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/8e218c3d-364b-4152-93f9-e933c32ef48e" />


The primary goal was to perform a Manual Forensic Analysis of the file Leave_me_alone.png to detect hidden data that automated tools like steghide failed to identify. This involves inspecting the file for appended data (EOF injection), hidden strings in the ASCII panel, or non-standard chunk headers.

•	Tool Utilization: The binary data was inspected using Hexed.it, a web-based hexadecimal editor.

•	Magic Byte Verification: Manual inspection confirmed the presence of standard PNG magic bytes (89 50 4E 47 0D 0A 1A 0A) at the start of the file, verifying it is a legitimate image format.


Step 15: Archive Extraction and Intelligence Gathering
 
<img width="940" height="556" alt="image" src="https://github.com/user-attachments/assets/c1fa330a-c3b9-4f66-8916-befea4ce9a1d" />


The primary goal was to extract and analyze the contents of an identified archive file, ss.zip, to find secondary credentials or system configurations necessary for horizontal privilege escalation to the slade account.

•	Archive Decompression: The command unzip ss.zip was executed, revealing two distinct text files: passwd.txt and shado.

•	Reading shado via the cat command revealed a single string: M3tahuman.

•	This string is highly significant as it follows the "Deathstroke" theme and serves as a primary candidate for an SSH password or a steganographic passphrase.

Step 16 : Lateral Movement (SSH Intrusion)

<img width="940" height="640" alt="image" src="https://github.com/user-attachments/assets/5bc54c94-8bbe-4311-8563-3cee980a8b84" />


With a high-probability credential pair (slade : M3tahuman), the next objective was to gain a remote terminal (shell) on the target machine via SSH (Secure Shell).

ssh slade@10.48.143.196

•	Authentication: The system prompted for a password, and the harvested string M3tahuman was entered.

•	The Result: The connection was established successfully. The target machine displayed a custom ASCII banner: "WELCOME 2 LIAN_YU".

Step 12: Privilege Escalation and Root Compromise

<img width="940" height="494" alt="image" src="https://github.com/user-attachments/assets/04223aef-21c3-44e0-9f82-8770a2b7b0c8" />

After establishing a stable foothold as the user slade, the final objective was to identify and exploit a misconfiguration to gain root (superuser) privileges and retrieve the final system flag.


- sudo -l was executed to list the allowed commands for the current user.

- The output revealed that slade may run /usr/bin/pkexec as root without a password:
 (root) NOPASSWD: /usr/bin/pkexec

- The pkexec binary was leveraged to spawn a root shell by executing:
sudo pkexec /bin/bash

- The command whoami was run, returning root, confirming successful vertical privilege escalation.

- Listed hidden files using ls -al to locate root.txt.

- Executed cat root.txt to retrieve the final proof of compromise.

Root Flag: THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}

<img width="940" height="103" alt="image" src="https://github.com/user-attachments/assets/16b3a19f-a2b4-4881-8b7b-460d2f63071e" />

<img width="940" height="306" alt="image" src="https://github.com/user-attachments/assets/ca99d82d-abf1-467e-831c-b1c9c2b6b8a4" />

<img width="940" height="159" alt="image" src="https://github.com/user-attachments/assets/f1454964-7f88-4467-ad6b-20c1dc74ea2d" />





