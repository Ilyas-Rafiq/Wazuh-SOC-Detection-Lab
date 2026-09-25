# Hands-on SOC Detection Lab: Wazuh SIEM

A practical detection engineering lab demonstrating endpoint telemetry monitoring, brute-force authentication triage, and real-time File Integrity Monitoring (FIM) using Wazuh SIEM.

---

## 🛠️ Lab Architecture & Environment

* **SIEM Manager:** Wazuh Server deployed on a Linux virtual machine
* **Monitored Endpoint:** Windows 10 (`Win10-Client`) running the Wazuh Agent service
* **Target User Account:** `ilyas` (monitored for credential abuse and administrative actions)
* **Tooling:** PowerShell for attack automation, Windows Event Viewer, Wazuh Web Dashboard

---

## 🔍 Scenario 1: Authentication Brute-Force & Account Lockout

### Objective
Simulate an automated credential-guessing attack against a local Windows account and verify detection, log ingestion, and alert correlation within the SIEM.

### Attack Simulation
Automated a burst of failed logons against user `ilyas` using PowerShell:

```powershell
1..5 | ForEach-Object { net use \\127.0.0.1\c$ /user:ilyas "WrongPassword123" }
```

### Detection & Telemetry Analysis
* Ingested low-level Windows Security **Event ID 4625** (Logon Failure).
* Wazuh generated alerts for **Rule 60122** (`Logon Failure - Unknown user or bad password`).
* The correlation engine escalated the repeated burst into a high-severity alert: **Rule 60115** (`User account locked out (multiple login errors)`).

![Authentication Failure & Lockout](auth-lockout.png)

---

## 📁 Scenario 2: Real-Time File Integrity Monitoring (FIM)

### Objective
Monitor unauthorized file additions, tampering, and deletions inside a sensitive directory using cryptographic hashes in real time.

### Configuration
Configured the Wazuh agent (`ossec.conf`) to monitor the target directory with real-time detection enabled:

```xml
<syscheck>
    <directories check_all="yes" realtime="yes">C:\Confidential</directories>
</syscheck>

```


### Attack Simulation
Simulated the complete file lifecycle inside `C:\Confidential` using PowerShell:

```powershell
# 1. Create file
Set-Content -Path "C:\Confidential\alert_test.txt" -Value "Testing FIM detection"
Start-Sleep -Seconds 3

# 2. Modify file contents
Add-Content -Path "C:\Confidential\alert_test.txt" -Value "New line added"
Start-Sleep -Seconds 3

# 3. Delete file
Remove-Item -Path "C:\Confidential\alert_test.txt"

```

### Detection & Hash Tracking
Wazuh's FIM module captured all three lifecycle events instantly:
* **Rule 554 (Level 5):** `File added to the system.` (Baseline MD5/SHA-256 hash established)
* **Rule 550 (Level 7):** `Integrity checksum changed.` (Detected hash mismatch after content alteration)
* **Rule 553 (Level 7):** `File deleted.` (Tracked file removal from the protected directory)

![File Integrity Monitoring Telemetry](fim-detection.png)

---

## 🎯 Key Skills Demonstrated

* **Endpoint Agent Deployment:** Provisioned and verified real-time communications between endpoints and the SIEM manager.
* **Configuration Management:** Customized agent XML settings (`ossec.conf`) and managed Windows service states.
* **Security Event Analysis:** Investigated Windows Event ID 4625, correlating raw log data with high-severity SIEM rules.
* **Cryptographic Integrity Monitoring:** Analyzed file modification alerts driven by cryptographic checksum comparisons.
