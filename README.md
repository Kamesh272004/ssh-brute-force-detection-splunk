# SSH Brute Force Detection using Splunk Cloud

## Project Overview
This project demonstrates how to simulate and detect an **SSH brute force attack** in a small lab environment using **Kali Linux**, **Ubuntu Server/Desktop**, and **Splunk Cloud**.

The objective of this lab is to:
- Generate failed SSH login attempts from an attacker machine
- Collect authentication logs from the target Ubuntu system
- Ingest the logs into Splunk Cloud
- Build a dashboard to visualize brute force indicators such as:
  - Total failed SSH logins
  - Attacker source IP addresses
  - Failed login attempts over time

This project is useful for understanding how a SOC analyst can detect **credential attacks** and monitor authentication abuse using SIEM.

---

## Architecture

### Lab Setup
- **Attacker Machine:** Kali Linux
- **Target Machine:** Ubuntu VM with OpenSSH enabled
- **SIEM Platform:** Splunk Cloud
- **Log Source:** `/var/log/auth.log` from Ubuntu

### Attack Flow
1. Kali connects to the Ubuntu machine over SSH.
2. Multiple invalid password attempts are made against a fake/invalid username.
3. Ubuntu logs the failed authentication attempts in `/var/log/auth.log`.
4. The log file is cleaned/copied and served to Splunk Cloud for ingestion.
5. Splunk parses the logs and visualizes brute force activity on a dashboard.

---

## Objectives
- Simulate a brute force style SSH authentication attack in a safe lab environment
- Understand Linux authentication logs
- Detect failed SSH login attempts in Splunk
- Identify attacker IP addresses
- Build a simple SOC-style dashboard for monitoring SSH brute force activity

---

## Tools & Technologies Used
- **Kali Linux** – attacker machine used to initiate SSH login attempts
- **Ubuntu** – target machine hosting SSH service and authentication logs
- **OpenSSH Server** – service exposed on the Ubuntu target
- **Splunk Cloud** – log ingestion, search, and dashboard visualization
- **Linux auth.log** – source of SSH authentication events

---

## Attack Scenario
In this lab, the Kali machine attempts to log in to the Ubuntu host using SSH with an **invalid user / incorrect password** multiple times.

The Ubuntu target records events similar to:
- `Failed password for invalid user ...`
- `Invalid user ...`
- `Connection closed by invalid user ...`

These events are then analyzed in Splunk to confirm the brute force attempt and identify the source IP responsible.

---

## Environment Details

| Component | Role |
|----------|------|
| Kali Linux | Attacker |
| Ubuntu VM | SSH target / log source |
| Splunk Cloud | SIEM / log analysis |
| auth.log | Authentication log file |

### Example IPs observed in the lab
- **Attacker IP:** `192.168.64.1`
- **Target Ubuntu IP:** `192.168.64.4`

> Note: IPs may vary depending on your VM/network setup.

---

## Project Workflow

## 1) Configure SSH on Ubuntu
The first step was to enable SSH access on the Ubuntu target.

### Commands used
```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
