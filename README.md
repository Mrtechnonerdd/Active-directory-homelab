# Active-directory-homelab
Active directory homelab for creating, adding and reseting passwords for users
# My First Active Directory & Help Desk Home Lab

## What is this project?
I built this lab to get hands-on experience with Active Directory before landing my first IT Help Desk job. Instead of just reading about how domain networks work, I wanted to actually build one from scratch, see how computers talk to each other, and practice the exact tasks I'll be doing on the job every day.

### My Setup
* **The Software:** VirtualBox (Free tool to run virtual machines)
* **The Server VM:** Windows Server 2022
* **The Client VM:** Windows 10 Pro
* **The Network:** A private, isolated network (`intnet`) so these machines can talk to each other without touching my home internet.

---

## Step 1: Setting up the Virtual Machines & Private Network
First, I downloaded the official ISO files from Microsoft and created two virtual machines in VirtualBox. 
* To make sure they could see each other, I went into the settings for both VMs and changed their network adapters from "NAT" to "Internal Network" and named it `intnet`. 
* This basically plugs both machines into the same invisible network switch, completely safe and separated from my home Wi-Fi.
<img width="990" height="509" alt="Screenshot 2026-11-07 180242" src="https://github.com/user-attachments/assets/87a81e96-d57b-4159-bdd9-5b4134767199" />

---

## Step 2: Fixing the IP Addresses & Troubleshooting the Firewall
Since the machines were disconnected from the internet, they didn't have IP addresses. I had to type them in manually so they had a way to communicate.
* I gave the Server a static IP of `192.168.10.10` and set its DNS to `127.0.0.1` (so it looks at itself).
* I gave the Windows 10 Client an IP of `192.168.10.20` and pointed its Preferred DNS straight to the Server's IP (`192.168.10.10`).

### My First IT Support Win (Fixing a Broken Ping)
When I opened the Command Prompt on the Windows 10 machine and tried to type `ping 192.168.10.10` to see if it could talk to the server, it kept saying **"Request timed out" (100% loss)**. 
* I didn't panic. I checked my settings and realized the physical network was fine, but Windows Server blocks pings by default.
* I went onto the Server, opened the Control Panel, and turned off the Windows Defender Firewall for a second. 
* I re-ran the ping test on the Windows 10 machine and it was successful! Data was officially flowing.
*  
<img width="920" height="511" alt="Screenshot 2026-11-07 180119" src="https://github.com/user-attachments/assets/79c6ff25-8576-48ab-a8e0-ee207c3b871b" />

<img width="990" height="509" alt="Screenshot 2026-11-07 180242" src="https://github.com/user-attachments/assets/99165a81-e412-4a5b-af77-648d6f4ceda9" />


---

## Step 3: Installing Active Directory & Adding a Department Folder
Next, I opened Server Manager, clicked "Add Roles and Features," and checked the box for **Active Directory Domain Services**. Once the software downloaded, I promoted the server to a Domain Controller and named my new domain `helpdesk.local`. The server rebooted, and I was officially running Active Directory.

<img width="918" height="715" alt="Screenshot 2026-11-07 180618" src="https://github.com/user-attachments/assets/4b84562a-bb5e-45fc-bcfe-263c10e02035" />

<img width="1015" height="652" alt="Screenshot 2026-11-07 181247" src="https://github.com/user-attachments/assets/58172310-e387-435b-88f7-06612dabfa9c" />



### Creating a New Employee Account
I opened the main help desk tool: **Active Directory Users and Computers (ADUC)**. 
* To keep things clean, I didn't want to just dump users everywhere. I right-clicked my domain, clicked New, and made an **Organizational Unit (OU)** called `Sales`. This acts like a folder for that specific department.
* Inside the Sales folder, I right-clicked and created a new user for an employee named **John Doe** (username: `jdoe`). 
* I set a temporary password and checked the box for **"User must change password at next logon."** This is standard practice so the user has a private password the first time they log in.


<img width="1014" height="764" alt="Screenshot 2026-11-07 182837" src="https://github.com/user-attachments/assets/15707652-e86b-4bae-bd1d-6ee0d3c98652" />

---

## Step 4: Setting up a Security Policy & Joining the Domain
By default, you can guess a password a million times on a new server. To make my lab realistic, I wanted to lock out users if they guessed wrong 5 times.
* I opened **Group Policy Management**, edited the Default Domain Policy, navigated to Account Lockout Policy, and changed the threshold to **5 attempts**.

* 
* I opened Command Prompt on the server and typed `gpupdate /force` to make the new security rule kick in immediately.
* <img width="997" height="657" alt="Screenshot 2026-11-07 190324" src="https://github.com/user-attachments/assets/7fc480f0-f6a7-48d1-a889-428d48c8feb2" />


### Connecting the Windows 10 PC to the Domain

<img width="1016" height="773" alt="Screenshot 2026-11-07 183208" src="https://github.com/user-attachments/assets/3a1b536f-d489-442a-8b79-da04c9e012c7" />

I went over to the Windows 10 VM, opened the advanced system settings, and changed it from a "Workgroup" to the `helpdesk.local` domain. It asked for permission, I typed in the server's administrator password, and it popped up saying **"Welcome to the helpdesk.local domain."** After a quick restart, the PC was officially joined to the company.

---

## Everyday Help Desk Scenarios I Practiced in This Lab

### 1. Handling an Account Lockout
To test my new security policy, I went to the Windows 10 login screen, selected John Doe, and intentionally typed the wrong password 5 times. On the 6th try, Windows blocked me and threw the official **"Account is currently locked out"** error. 
* To fix it, I jumped back to the Server, right-clicked John Doe's account in ADUC, went to Properties -> Account tab, checked the **"Unlock account"** box, and hit Apply. John was instantly back online.

### 2. Password Resets
I practiced resetting passwords by right-clicking John's name, picking a new temporary password, and making sure the "change password at next logon" box was checked so he could reset it securely on his end.

### 3. Permissions & Security Groups (Least Privilege)
I practiced adding John Doe to the **Domain Admins** group under his account properties to see how easy it is to give someone full administrative rights. However, I also learned about the **Principle of Least Privilege**, meaning standard users shouldn't have admin rights because it's a massive security risk. After testing it out, I safely removed him from the admin group to keep the network secure.
