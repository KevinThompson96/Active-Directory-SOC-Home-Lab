# Active-Directory-SOC-Home-Lab
## Overview
This project builds off my previous labs, combining my Active Direcotry corpprate simulation, adding a network firewall and SIEM, and performing an attack/security monitoring exercise. After configuring the lab architecture in VMware, my goal was to perform a remote desktop protocol (RDP) brute-force password attack against the client machine in my AD and detect the activity using a SIEM platform.

## Environment/Technology
-VMware Workstation Pro

-pfSense (firewall)

-Windows Server 2025 (Active Directory)

-Windows 10 (Client)

-Ubuntu (Wazuh SIEM)

-Kali Linux (Hydra)


## Lab Architecture

![Network Diagram](Screenshots/ADSOC-Network-Diagram.png)

## Attack Scenario: RDP Brute-Force
I configured the client machine to allow RDP services and setup a port forwarding rule on the firewall to allow and forward RDP traffic on port 3389, purely for the demonstration in this lab. 

![Port forwarding rule](Screenshots/Port-Forwarding-Rule.png)

After scanning the target WAN IP with nmap, a brute-force attack was launched from the Kali machine against the domain-joined Windows client.

![nmap scan](Screenshots/nmap-scan2.png)

![hydra attack](Screenshots/Hydra-Attack.png)

![hydra success](Screenshots/Hydra-Success.png)

Using xfreerdp3 to remote into the domain-joined client, I added a backdoor account to maintain persistence and assigned it to the administrators group to escalate my privileges

![Remote access](Screenshots/Remote-Access.png)

![Added Backdoor](Screenshots/Add-Backdoor.png)

![Assigned backdoor to admins](Screenshots/Backdoor-Admin.png)


**Outcome:**

-Multiple failed login attempts using the pre-configured wordlist "rockyou.txt" containing weak passwords

-Hydra successfully identified valid credentials

-Remote access to the target system was achieved via RDP

-A "backdoor" account with elevated privileges was added to maintain persistence 


## Detection and Monitoring with Wazuh

I installed Wazuh agents on both the domain controller and the client machine to forward event logs to the SIEM server. For this attack, I focused on two Windows event IDs related to authentication: 

-4625 (Failed login attempt)

-4624 (Successful login attempt)

The brute-force attack was identified by analyzing repeated failed login attempts followed by a successful login

![Wazuh failed logins](Screenshots/Failed-Login-4625.png)

![Wazuh successful login](Screenshots/Successful-Login-4624.png)

![Wazuh user added and admin group changed](Screenshots/User-Admin-Added.png)

**Observations:**

-A high volume of failed login attempts in a short timeframe

-Consistent targeting of a single user account (password stuffing)

-User added to system and administrator group modified


**Group Policy Account Lockout and Firewall Configuration**

To protect against password stuffing, I added a group policy object (gpo) in my domain through active directory, setting the threshold to 5 invalid logon attempts before the account it locked out (for 10 minutes), and the counter resets every 10 minutes. Down below is a user account in the domain (mbailey) that has been locked out after 5 repeated attempts. To unlock the account immediately as an administrator, I could go to active directory users and groups on the domain controller, find the account in my domain, and unlock it manually. 

![GPO](Screenshots/GPO.png)

![Lockout](Screenshots/Lockout.png)


