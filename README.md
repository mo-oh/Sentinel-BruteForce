# Sentinel-BruteForce
This repository showcases my project deploying, investigating and implementing an incident response plan with Microsoft Sentinel

# 🚀 Microsoft Sentinel SOAR — Automated Cloud Incident Response

## 🛡️ Project Overview

Welcome to an advanced cloud security automation project! This repository showcases how to build an **end-to-end, automated incident response playbook** using a fully cloud-native lab. By simulating a brute-force attack against a Windows server, you’ll see how Microsoft Sentinel (SIEM), Kusto Query Language (KQL), and Azure Logic Apps can seamlessly work together to **detect, contain, and remediate threats** — all without human intervention.

---

## 🎯 Project Goals

- **Simulate** a brute-force attack in a controlled Azure lab.
- **Ingest and analyze logs** with Microsoft Sentinel for real-time threat detection.
- **Create custom KQL rules** to identify malicious activity.
- **Automate the response** with a SOAR playbook using Azure Logic Apps.
- **Document the attack and response** for learning and portfolio presentation.

---

## 🏗️ Lab Architecture

```
Azure Cloud Environment
├── Evil-VM      [Attacker]   IP: 10.0.0.4
├── Victim-VM    [Target]     IP: 10.0.0.5 (forwards logs to Sentinel)
└── Microsoft Sentinel (SIEM)
      └── Automated Logic Apps Playbook
```

---

## 🔬 Process Walkthrough

### 1️⃣ Attack Simulation

A brute-force attack is simulated from **Evil-VM** to **Victim-VM** using a PowerShell script that generates failed logon attempts, creating Event ID 4625 logs.

```powershell
# Simulates brute-force attack by generating Event ID 4625 logs on Victim-VM
for ($i = 1; $i -le 1000; $i++) {
    net use \\10.0.0.5\c$ /user:Adm!n /password:wrongpassword
}
```

---

### 2️⃣ Detection with KQL (Sentinel Analytics Rule)

A custom KQL query detects suspicious spikes in failed login attempts, correlates them with historical baselines, and surfaces meaningful context for investigation.

```kql
let starttime = 8d;
let endtime = 1d;
let threshold = 0.333;
let countlimit = 50;
SecurityEvent
| where TimeGenerated >= ago(endtime)
| where EventID == 4625 and AccountType =~ "User"
| where IpAddress !in ("127.0.0.1", "::1")
| summarize StartTime = min(TimeGenerated), EndTime = max(TimeGenerated), CountToday = count() by EventID, Account, LogonTypeName, SubStatus, AccountType, Computer, WorkstationName, IpAddress, Process
// (additional logic omitted for brevity)
```

Features:
- Compares current spike vs previous 7-day baseline
- Identifies root causes using SubStatus codes
- Flags abnormal login attempts for automated response

---

### 3️⃣ Automated Response (SOAR Playbook)

When the KQL rule triggers, **Azure Logic Apps** launches a playbook to:
- Parse Sentinel alert (extract malicious IP, timestamp, etc.)
- Notify the security team via email/Teams with attack details
- Optionally trigger further automated actions (e.g., block IP, isolate VM)

---

## 📸 Evidence & Artifacts

- ![ ] Brute-force attack logs from Victim-VM’s Event Viewer
- ![ ] KQL query results in Sentinel showing detected malicious IP
- ![ ] Logic Apps playbook workflow screenshot

---

## ⚙️ Key Technologies Used

- **Microsoft Azure:** Cloud platform for VMs and services
- **Microsoft Sentinel:** SIEM & SOAR platform
- **Kusto Query Language (KQL):** Log analytics for detection & hunting
- **Azure Logic Apps:** Automated incident response orchestration

---

## 📁 Folder Structure

```
/sentinel-soar-automated-cloud-incident-response
│
├── README.md
├── powershell/
│   └── brute_force_attack.ps1
├── kql/
│   └── brute_force_detection.kql
├── logicapps/
│   └── playbook_workflow.json
└── evidence/
    ├── eventviewer_screenshot.png
    ├── kql_results_screenshot.png
    └── logicapp_workflow_screenshot.png
```

---

## 🚦 How to Use

1. **Deploy** two Windows Server VMs in Azure (see architecture above).
2. **Configure** Victim-VM to forward security logs to Microsoft Sentinel.
3. **Run** the PowerShell brute-force attack script on Evil-VM.
4. **Set up** the KQL analytics rule in Sentinel.
5. **Connect** the Logic Apps playbook for automated response.
6. **Document** your results with evidence screenshots (add to `/evidence`).

---
