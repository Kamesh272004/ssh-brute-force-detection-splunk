# SSH Brute Force Detection using Splunk Cloud

## Project Overview

This project demonstrates how to simulate and detect an **SSH brute force attack** in a lab environment using **Kali Linux**, **Ubuntu**, and **Splunk Cloud**.

The goal of this project was to:

* Generate failed SSH login attempts from an attacker machine
* Collect authentication logs from the target Ubuntu system
* Ingest those logs into Splunk Cloud
* Build a dashboard to visualize brute force indicators such as:

  * Total failed SSH logins
  * Attacker source IP addresses
  * Failed login attempts over time

This project is useful for understanding how a SOC analyst can detect **credential attacks** and monitor authentication abuse using a SIEM.

---

## Architecture

### Lab Setup

* **Attacker Machine:** Kali Linux
* **Target Machine:** Ubuntu VM with OpenSSH enabled
* **SIEM Platform:** Splunk Cloud
* **Log Source:** `/var/log/auth.log`

### Attack Flow

1. Kali Linux connects to the Ubuntu machine over SSH.
2. Multiple invalid password attempts are made against a fake / invalid username.
3. Ubuntu logs the failed authentication attempts in `/var/log/auth.log`.
4. The log file is copied and ingested into Splunk Cloud.
5. Splunk parses the logs and visualizes brute force activity on a dashboard.

---

## Objectives

* Simulate a brute force style SSH authentication attack in a safe lab environment
* Understand Linux authentication logs
* Detect failed SSH login attempts in Splunk
* Identify attacker IP addresses
* Build a SOC-style dashboard for monitoring SSH brute force activity

---

## Tools & Technologies Used

* **Kali Linux** – attacker machine used to initiate SSH login attempts
* **Ubuntu** – target machine hosting SSH service and authentication logs
* **OpenSSH Server** – service exposed on the Ubuntu target
* **Splunk Cloud** – log ingestion, search, and dashboard visualization
* **Linux `auth.log`** – source of SSH authentication events

---

## Attack Scenario

In this lab, the Kali machine attempts to log in to the Ubuntu host using SSH with an **invalid user / incorrect password** multiple times.

The Ubuntu target records events such as:

* `Failed password for invalid user`
* `Invalid user`
* `Connection closed by invalid user`

These events are then analyzed in Splunk to confirm the brute force attempt and identify the source IP responsible.

---

## Environment Details

| Component    | Role                    |
| ------------ | ----------------------- |
| Kali Linux   | Attacker                |
| Ubuntu VM    | SSH target / log source |
| Splunk Cloud | SIEM / log analysis     |
| `auth.log`   | Authentication log file |

### Example IPs observed in the lab

* **Attacker IP:** `192.168.64.1`
* **Target Ubuntu IP:** `192.168.64.4`

> **Note:** IP addresses may vary depending on your VM / network setup.

---

## Project Workflow

### 1) Configure SSH on Ubuntu

The first step was to enable SSH access on the Ubuntu target.

#### Commands used

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```

---

### 2) Validate Connectivity from Kali

Before generating attack logs, connectivity between Kali and Ubuntu was verified using `ping`.

#### Command used

```bash
ping 192.168.64.4
```

---

### 3) Simulate SSH Failed Login Attempts

A test SSH login was initiated from Kali to the Ubuntu machine using a fake or invalid account.

#### Command used

```bash
ssh fakeuser@192.168.64.4
```

Multiple incorrect password attempts were intentionally entered to generate failed authentication events.

---

### 4) Review Authentication Logs on Ubuntu

After the failed SSH attempts, the Ubuntu authentication log was reviewed.

#### Command used

```bash
sudo tail -f /var/log/auth.log
```

#### Example log events observed

```log
Failed password for invalid user fakeuser from 192.168.64.1 port 58624 ssh2
pam_unix(sshd:auth): check pass; user unknown
pam_unix(sshd:auth): authentication failure
Connection closed by invalid user fakeuser 192.168.64.1 port 58624 [preauth]
```

These events confirm:

* An SSH authentication attempt was made
* The username was invalid
* Password attempts failed
* The source IP was recorded in the log

---

### 5) Prepare Log File for Splunk Ingestion

A cleaned copy of the authentication log was created and moved to a directory for ingestion.

#### Commands used

```bash
sudo cp /var/log/auth.log /home/ubuntu/Desktop/auth_clean.log
sudo chmod 777 /home/ubuntu/Desktop/auth_clean.log
cd /home/ubuntu/Desktop
python3 -m http.server 8000
```

> In a real production environment, logs should be forwarded securely using a proper log forwarder instead of a temporary HTTP server.

---

### 6) Ingest Logs into Splunk Cloud

The cleaned authentication log was uploaded / ingested into Splunk Cloud and indexed for searching.

Once the data was available in Splunk, searches were created to identify failed SSH login activity.

---

## Splunk Detection Logic

### 1. Basic Search for Failed SSH Logins

```spl
index=* "Failed password"
```

### 2. Search for SSH Brute Force Attempts

```spl
index=* source="*auth*" "Failed password"
```

### 3. Extract Attacker IPs and Count Attempts

```spl
index=* source="*auth*" "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count by src_ip
| sort - count
```

### 4. Failed Login Attempts Over Time

```spl
index=* source="*auth*" "Failed password"
| timechart count
```

### 5. Threshold-Based Brute Force Detection

```spl
index=* source="*auth*" "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| bucket _time span=5m
| stats count by _time src_ip
| where count >= 5
```

---

## Dashboard Built in Splunk Cloud

A dashboard titled **SSH Brute Force Detection** was created with the following panels:

1. **Total Failed SSH Logins**
   Displays the total number of failed SSH login events detected in the ingested logs.

2. **Top Attacker IP Addresses**
   Displays the source IP address responsible for the failed login attempts.

3. **Failed Login Attempts Over Time**
   Visualizes the brute force activity timeline to show when authentication failures occurred.

---

## Sample Findings

From the test data generated in this lab:

* **Total failed SSH logins detected:** 3
* **Primary attacker IP identified:** `192.168.64.1`
* **Attack type observed:** Failed SSH authentication / brute force style login attempts
* **Target service:** OpenSSH on port 22

---

## Key Log Indicators

The following log patterns are useful for SSH brute force detection in Linux:

* `Failed password for invalid user`
* `Invalid user`
* `pam_unix(sshd:auth): authentication failure`
* `Connection closed by invalid user`
* Repeated failed logins from the same IP in a short period

---

## MITRE ATT&CK Mapping

* **T1110 – Brute Force**
  Repeated SSH authentication attempts using invalid credentials

* **T1078 – Valid Accounts** *(possible follow-on impact if credentials are guessed successfully)*
  Relevant if the attacker later gains access with valid credentials

---

## Detection Use Case

This project simulates a common SOC monitoring use case:

**Detect repeated failed SSH authentication attempts against Linux systems and identify potential brute force activity.**

### Why it matters

Repeated SSH login failures may indicate:

* Password guessing
* Brute force attacks
* Unauthorized access attempts
* Reconnaissance against exposed Linux systems

---

## Skills Demonstrated

This project demonstrates hands-on knowledge in:

* Linux log analysis
* SSH service monitoring
* Brute force attack detection
* Splunk Cloud search and dashboard creation
* Basic SOC investigation workflow
* Source IP extraction from raw logs
* Security event visualization

---

## Investigation Summary

During the simulation, multiple failed SSH login attempts were generated from the Kali attacker machine against the Ubuntu target using an invalid user account. These attempts were recorded in the target system’s `/var/log/auth.log` file and later ingested into Splunk Cloud.

The analysis confirmed:

* Repeated failed SSH authentication attempts
* Invalid user login activity
* Source IP of the attacker machine
* Time-based spike in failed logins visible on the dashboard

This indicates a brute-force style authentication attack against the SSH service.

---

## Project Structure

```bash
ssh-brute-force-detection-splunk/
│── README.md
│── auth_clean.log
│── screenshots/
│   ├── ssh-service-status.png
│   ├── kali-ssh-attempt.png
│   ├── ubuntu-auth-log-failures.png
│   └── splunk-dashboard.png
│── queries/
│   └── splunk_searches.txt
```

---

## Screenshots

### 1. SSH Service Enabled on Ubuntu
![SSH Service Enabled](SSH%20connection%20established.png)

### 2. Kali Attacker Reaching the Ubuntu Target and Initiating SSH Attempts
![Kali SSH Attempt](Host%20ip%20ping%20and%20performing%20attack.png)

### 3. Failed Password Attempts Recorded in Ubuntu auth.log
![Ubuntu Auth Log Failures](failed%20passwords%20attempts.png)

### 4. Splunk Dashboard for Brute Force Detection
![Splunk Dashboard](log%20analysis%20dashboard%20(splunk).png)

---

## Limitations

* This is a **lab simulation**, not a production environment
* Attack volume is small and manually generated
* Logs were prepared manually for Splunk ingestion
* No automated alerting or response workflow was implemented in this version

---

## Future Improvements

This project can be extended by adding:

* **Splunk alerts** for repeated failed SSH attempts
* **Threshold-based brute force detection** with alerting
* **GeoIP enrichment** for source IPs
* **Correlation rules** for successful login after multiple failures
* **Linux log forwarding** using Splunk Universal Forwarder or syslog
* **MITRE ATT&CK tagged dashboards**
* **Detection of password spray vs brute force behavior**

---

## Conclusion

This project successfully demonstrates how to detect **SSH brute force activity** using Linux authentication logs and Splunk Cloud. By generating failed SSH login attempts from Kali, reviewing `/var/log/auth.log`, and visualizing the results in Splunk, the project shows a practical SOC-style detection workflow for identifying credential attacks against Linux systems.

---

## Author

**Kamesh**
Cybersecurity / SOC / Detection Engineering Lab Project

