# Active-Directory-Lab
**Bilal GHLOUCI**

## Index 
1. [Introduction](#introduction)
2. [Setup VMs](#setup-vms)
   - [Software Requirements](#software-requirements)
   - [Lab Topology](#lab-topology)
   - [VM Configurations](#vm-configurations)
3. [Installation of Software](#installation-of-software)
   - [Splunk Server (Ubuntu)](#splunk-server-ubuntu)
   - [Splunk Forwarder and Sysmon](#splunk-forwarder-and-sysmon)
4. [Install and Configure Active Directory](#install-and-configure-active-directory)
5. [Brute Force Attack with Kali Linux](#brute-force-attack-with-kali-linux)
6. [Testing with Atomic Red Team](#testing-with-atomic-red-team)
7. [Conclusion](#conclusion)

---

## Introduction

In this project, I have set up a comprehensive Active Directory environment to simulate real-world IT administration and security monitoring. The core of this lab is a **Splunk SIEM** instance that ingests telemetry from both a Windows Server (Domain Controller) and a Windows 10 workstation. 

The primary goal is to observe the digital footprints left by attackers. We will perform a **Brute Force attack** using Kali Linux and execute various adversary emulation tests using **Atomic Red Team** to validate our detection capabilities.

---

## Setup VMs

### Software Requirements
- **Hypervisor**: VirtualBox
- **Operating Systems**:
    - Windows Server 2022 (Domain Controller)
    - Windows 10 (Target Workstation)
    - Ubuntu 22.04 (Splunk Server)
    - Kali Linux (Attacker Machine)

### Lab Topology

<img width="1001" height="751" alt="AD-project drawio" src="https://github.com/user-attachments/assets/c74650fa-8f52-4d35-90f2-4997c0163e98" />

The infrastructure follows this network plan:
- **Domain Controller (ADDC01)**: `192.168.10.7` | Managing the `MyLAB` domain.
- **Splunk Server**: `192.168.10.10` | Running on Ubuntu for log analysis.
- **Attacker Machine**: `192.168.10.250` | Kali Linux for offensive testing.
- **Windows 10 Machine**: `DHCP` | The target workstation within the AD domain.

### VM Configurations
Each machine is configured within a **NAT Network** in VirtualBox to allow internal communication and internet access for updates.
<img width="952" height="607" alt="ad2" src="https://github.com/user-attachments/assets/291f0d52-b99c-4daa-b00a-924e2fb3bcb7" />
<img width="812" height="431" alt="ad3" src="https://github.com/user-attachments/assets/0d585662-3486-40ab-9e54-f1a458139c7c" />

* **Splunk Server**: 8GB RAM, 4 CPUs, 100GB Disk.
* **Domain Controller**: 4GB RAM, 2 CPUs, 50GB Disk.
* **Windows 10**: 4GB RAM, 2 CPUs, 50GB Disk.
<img width="861" height="441" alt="ad1" src="https://github.com/user-attachments/assets/c5f7a27f-6206-4b9e-b4c4-81c65238f9de" />

---

## Installation of Software

### Splunk Server (Ubuntu)
The Splunk instance is hosted on Ubuntu Server. After the initial OS install, the system is updated and Splunk Enterprise is installed via the `.deb` package.
<img width="622" height="416" alt="ad5" src="https://github.com/user-attachments/assets/c8ede16e-d564-4d86-9d93-19eaaf7d60f5" />

<img width="780" height="503" alt="ad6" src="https://github.com/user-attachments/assets/c88d1684-0d29-49ff-a38a-75944302b3d5" />

1. **Static IP Configuration**: Set to `192.168.10.10` using Netplan.
<img width="595" height="242" alt="ad4" src="https://github.com/user-attachments/assets/b866a112-f013-4712-a6a2-f8baebbf2142" />
2. **Splunk Setup**: Installed in `/opt/splunk`.
<img width="480" height="33" alt="ad7" src="https://github.com/user-attachments/assets/395e837d-703f-424a-91e3-2039fe917d65" />

<img width="650" height="41" alt="ad8" src="https://github.com/user-attachments/assets/5fbc82e1-1302-4114-8434-f361d1682642" />

3. **Receiving Configuration**: Configured Splunk to listen on port `9997` to receive data from universal forwarders.
<img width="797" height="387" alt="ad10" src="https://github.com/user-attachments/assets/8d8d9df1-5706-4d1f-875c-9a081d87d24a" />

### Splunk Forwarder and Sysmon
To get visibility into the Windows machines, we deploy the **Splunk Universal Forwarder (UF)** and **Sysmon**.
<img width="536" height="452" alt="ad11" src="https://github.com/user-attachments/assets/16c07504-da09-4f82-8856-4df76241e8fc" />

- **Sysmon**: Installed using the `sysmonconfig.xml` (Olaf Hartong configuration) to capture high-fidelity process and network logs.
<img width="436" height="261" alt="ad12" src="https://github.com/user-attachments/assets/d3845f4f-404c-4360-a7ed-7ffc79421733" />

- **inputs.conf**: Configured on both Windows machines to push the following logs to the indexer:
    - Application, Security, and System logs.
    - Microsoft-Windows-Sysmon/Operational logs.
<img width="525" height="367" alt="ad13" src="https://github.com/user-attachments/assets/54387705-c4ab-4299-9497-e59e532c7f54" />
<img width="841" height="602" alt="ad14" src="https://github.com/user-attachments/assets/ba591c08-8724-4324-be98-7a8126dea905" />
<img width="592" height="270" alt="ad15" src="https://github.com/user-attachments/assets/18b5e33f-7ef0-4d03-8413-6bc7678be42c" />

---

## Install and Configure Active Directory

The Windows Server was promoted to a **Domain Controller** for the `adlab.local` (or `MyLAB`) forest.

1. **Role Installation**: Added Active Directory Domain Services (ADDS).
<img width="1010" height="452" alt="ad16" src="https://github.com/user-attachments/assets/b326e3f7-490e-4f4a-997f-2a3588208334" />

2. **Promotion**: Configured the new forest and set up the DNS.
<img width="795" height="635" alt="ad9" src="https://github.com/user-attachments/assets/32f5b098-7c1c-4cb9-98eb-68c1caa732ee" />

3. **User Creation**: Built Organizational Units (OUs) for "IT" and "HR" and added simulation users like "Jenny Smith".
<img width="698" height="388" alt="ad17" src="https://github.com/user-attachments/assets/31fdc68e-d24e-4ab8-95cc-f1c24690494c" />

4. **Domain Join**: The Windows 10 workstation was pointed to the DC's IP for DNS and successfully joined to the domain.
<img width="655" height="492" alt="Capture d’écran 2026-02-12 153137" src="https://github.com/user-attachments/assets/e69d2a83-5b33-4a91-9ba3-3abaad13725d" />


---

## Brute Force Attack with Kali Linux

With the logging pipeline active, we simulate a **Brute Force attack** from the Kali Linux machine (`192.168.10.250`).

- **Tool used**: Crowbar
- **Target**: RDP service on the Windows 10 machine.
<img width="550" height="407" alt="ad19" src="https://github.com/user-attachments/assets/280be63e-e7d7-468c-ae3e-60e1b2ee345a" />

- **Observation**: In Splunk, we monitor for **Event ID 4625** (Failed Logon) to visualize the attack pattern and identify the source IP.
<img width="880" height="588" alt="ad20" src="https://github.com/user-attachments/assets/acce9a3f-7f3e-4aae-a0b9-aaff818d6222" />

---

## Testing with Atomic Red Team

To go beyond simple brute force, we use **Atomic Red Team** on the Windows 10 machine. This allows us to execute small, highly portable detection tests mapped to the **MITRE ATT&CK** framework.
<img width="843" height="217" alt="ad21" src="https://github.com/user-attachments/assets/43c26f4b-aee7-4d26-bfb0-3bf200c4f2fa" />

**Simulated Techniques:**
- **T1059.001**: PowerShell execution patterns.
- **T1136.001**: Create Account: Local Account.
<img width="623" height="373" alt="ad22" src="https://github.com/user-attachments/assets/c09f7fcb-3791-4b73-86e7-a8e25d81c1e5" />

Each "Atom" executed provides immediate telemetry in Splunk, allowing us to verify if our Sysmon and Splunk configurations are capturing the malicious activity.
<img width="980" height="538" alt="ad23" src="https://github.com/user-attachments/assets/468ee3cd-c158-4b6f-aada-740eb253ae05" />


---

## Conclusion

This lab successfully demonstrates the "Full-Stack" of security operations: from infrastructure building and service configuration to offensive simulation and defensive analysis. By correlating Kali Linux actions with Splunk telemetry, we gain a deep understanding of how to defend an Active Directory environment.

---
**Bilal** | © 2026
