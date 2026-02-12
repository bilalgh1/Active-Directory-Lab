# Active-Directory-Lab
**[Ton Nom/Pseudo]**

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

![Network Diagram](https://i.imgur.com/kP12S0h.png)

The infrastructure follows this network plan:
- **Domain Controller (ADDC01)**: `192.168.10.7` | Managing the `MyLAB` domain.
- **Splunk Server**: `192.168.10.10` | Running on Ubuntu for log analysis.
- **Attacker Machine**: `192.168.10.250` | Kali Linux for offensive testing.
- **Windows 10 Machine**: `DHCP` | The target workstation within the AD domain.

### VM Configurations
Each machine is configured within a **NAT Network** in VirtualBox to allow internal communication and internet access for updates.

* **Splunk Server**: 8GB RAM, 4 CPUs, 100GB Disk.
* **Domain Controller**: 4GB RAM, 2 CPUs, 50GB Disk.
* **Windows 10**: 4GB RAM, 2 CPUs, 50GB Disk.

---

## Installation of Software

### Splunk Server (Ubuntu)
The Splunk instance is hosted on Ubuntu Server. After the initial OS install, the system is updated and Splunk Enterprise is installed via the `.deb` package.

1. **Static IP Configuration**: Set to `192.168.10.10` using Netplan.
2. **Splunk Setup**: Installed in `/opt/splunk`.
3. **Receiving Configuration**: Configured Splunk to listen on port `9997` to receive data from universal forwarders.

### Splunk Forwarder and Sysmon
To get visibility into the Windows machines, we deploy the **Splunk Universal Forwarder (UF)** and **Sysmon**.

- **Sysmon**: Installed using the `sysmonconfig.xml` (Olaf Hartong configuration) to capture high-fidelity process and network logs.
- **inputs.conf**: Configured on both Windows machines to push the following logs to the indexer:
    - Application, Security, and System logs.
    - Microsoft-Windows-Sysmon/Operational logs.

---

## Install and Configure Active Directory

The Windows Server was promoted to a **Domain Controller** for the `adlab.local` (or `MyLAB`) forest.

1. **Role Installation**: Added Active Directory Domain Services (ADDS).
2. **Promotion**: Configured the new forest and set up the DNS.
3. **User Creation**: Built Organizational Units (OUs) for "IT" and "HR" and added simulation users like "Adam Smith".
4. **Domain Join**: The Windows 10 workstation was pointed to the DC's IP for DNS and successfully joined to the domain.

---

## Brute Force Attack with Kali Linux

With the logging pipeline active, we simulate a **Brute Force attack** from the Kali Linux machine (`192.168.10.250`).

- **Tool used**: Hydra or custom scripts.
- **Target**: SMB/RDP services on the Windows 10 machine or DC.
- **Observation**: In Splunk, we monitor for **Event ID 4625** (Failed Logon) to visualize the attack pattern and identify the source IP.

---

## Testing with Atomic Red Team

To go beyond simple brute force, we use **Atomic Red Team** on the Windows 10 machine. This allows us to execute small, highly portable detection tests mapped to the **MITRE ATT&CK** framework.

**Simulated Techniques:**
- **T1059.001**: PowerShell execution patterns.
- **T1053.005**: Scheduled Task creation for persistence.
- **T1003**: LSASS memory dumping.

Each "Atom" executed provides immediate telemetry in Splunk, allowing us to verify if our Sysmon and Splunk configurations are capturing the malicious activity.

---

## Conclusion

This lab successfully demonstrates the "Full-Stack" of security operations: from infrastructure building and service configuration to offensive simulation and defensive analysis. By correlating Kali Linux actions with Splunk telemetry, we gain a deep understanding of how to defend an Active Directory environment.

---
**[Your Name]** | © 2024
