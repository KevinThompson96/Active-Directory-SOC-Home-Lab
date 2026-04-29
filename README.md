# Active-Directory-SOC-Home-Lab
## Overview
This project builds off my previous labs, combining my Active Direcotry corpprate simulation, adding a network firewall and SIEM, and performing an attack/security monitoring exercise. After configuring the lab architecture in VMware, my goal was to perform a remote desktop protocol (RDP) brute-force password attack against the client machine in my AD and detect the activity using a SIEM platform.

## Environment/Technology
-VMware Workstation Pro

-pfSense (firewall and NAT)

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

![nmap scan](Screenshots/nmap-scan.png)

![hydra attack](Screenshots/Hydra-Attack.png)

![hydra success](Screenshots/Hydra-Success.png)

**Outcome:**

-Multiple failed login attempts using the pre-configured wordlist "rockyou.txt" containing weak passwords

-Hydra successfully identified valid credentials

-Remote access to the target system was achieved via RDP

## Detection and Monitoring with Wazuh

I installed Wazuh agents on both the domain controller and the client machine to forward event logs to the SIEM server. For this attack, I focused on two Windows event IDs related to authentication: 

-4625 (Failed login attempt)

-4624 (Successful login attempt)

The brute-force attack was identified by analyzing repeated failed login attempts followed by a successful login

Screenshot 5

**Creating a Custum Wazuh Rule:**

After the attack, I created a custom rule to alert for excessive failed login attempts

Screenshot 6

Screenshot 7

**Group Policy Account Lockout and Firewall Configuration**


