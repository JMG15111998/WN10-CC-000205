# WN10-CC-000205
# 📡 Vulnerability Management Lab – WN10-CC-000205

**Title:** Windows Telemetry Must Not Be Configured to Full  
**STIG ID:** WN10-CC-000205  
**Compliance Frameworks:** DISA STIG, HIPAA, NIST 800-53, FedRAMP  
**Tools Used:** Azure, Tenable, PowerShell  
**Category:** Data Collection & Privacy  
**Remediation:** Registry Modification

---

## 📋 Lab Objective

Demonstrate the complete vulnerability management workflow for a **Windows 10 system configured with excessive telemetry** ("Full")—a condition prohibited by federal and STIG compliance baselines. The lab includes:

- Creating a misconfigured VM  
- Scanning for the telemetry misconfiguration  
- Applying secure baseline settings  
- Verifying the fix via registry and Tenable

---

## 📁 Table of Contents

1. [Azure VM Setup](#azure-vm-setup)  
2. [Vulnerability Simulation](#vulnerability-simulation)  
3. [Tenable Scan Configuration](#tenable-scan-configuration)  
4. [Initial Vulnerability Scan](#initial-vulnerability-scan)  
5. [Remediation via PowerShell](#remediation-via-powershell)  
6. [Post-Remediation Verification](#post-remediation-verification)  
7. [Security Rationale](#security-rationale)  
8. [Appendix: PowerShell Commands](#appendix-powershell-commands)

---

## ☁️ Azure VM Setup

### 🔹 Configuration Parameters

| Parameter         | Value                          |
|------------------|-------------------------------|
| VM Name          | `Win10-STIGLab-Telemetry`      |
| OS Image         | Windows 10 Pro (Gen 2)         |
| VM Size          | Standard D2s v3                |
| Resource Group   | `vm-lab-telemetry`             |
| Region           | Closest Azure region           |

### 🔹 Security Controls

- **Use strong, unique credentials** — avoid defaults like `labuser / Cyberlab123!`
- **NSG Rules**:
  - ✅ Allow RDP (TCP 3389)
  - ✅ Optionally allow WinRM (TCP 5985) for remote PowerShell
  - ❌ Deny all unnecessary inbound ports

### 🔹 Firewall & Remote Registry Prep

Disable Windows Defender Firewall:
- Open `wf.msc` → Disable all profiles (Domain, Private, Public)

Enable remote PowerShell for Tenable:
```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -Name "LocalAccountTokenFilterPolicy" -Value 1 -Type DWord -Force
```

---

## ⚠️ Vulnerability Simulation

### 🔸 Objective:
Simulate a **non-compliant telemetry configuration** where Windows is set to report Full diagnostic data back to Microsoft.

### 🔸 Create Misconfiguration

```powershell
# Simulate vulnerability: Set telemetry level to Full (3)
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\DataCollection" `
  -Name "AllowTelemetry" -Value 3
```

> `3` = Full telemetry (prohibited by DISA STIG)  
> `0` = Security level telemetry (compliant)

📸 **Screenshot Placeholder:** `Screenshot_01_Telemetry_Level_Full.png`

---

## 🔍 Tenable Scan Configuration

### 🔸 Template: Advanced Network Scan

| Setting Category | Required Options |
|------------------|------------------|
| **Targets**      | Azure VM IP      |
| **Plugins**      | STIG Compliance, Windows checks |
| **Auth Method**  | Local Admin (Windows credentials) |
| **Compliance**   | Upload and enable `Windows 10 STIG v3r4.audit` |
| **Remote Services** | Remote Registry, Server service, Admin shares |

---

## 🧪 Initial Vulnerability Scan

| STIG Control        | WN10-CC-000205                           |
|---------------------|------------------------------------------|
| Description         | Telemetry must not be set to "Full"      |
| Scan Result         | ❌ **Fail**                               |
| Detected Setting    | `AllowTelemetry = 3`                     |
| Required Setting    | `AllowTelemetry = 0`                     |

📸 **Screenshot Placeholder:** `Screenshot_02_Tenable_Telemetry_Fail.png`

---

## 🛠️ Remediation via PowerShell

### 🔸 Apply Compliant Telemetry Setting

```powershell
# Enforce STIG-compliant telemetry level (Security only)
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\DataCollection" `
  -Name "AllowTelemetry" -Value 0
```

📸 **Screenshot Placeholder:** `Screenshot_03_PowerShell_Telemetry_Fix.png`

> 🔄 **Reboot recommended** to ensure telemetry services reload with new configuration.

---

## 🔁 Post-Remediation Verification

### 🔸 Validate Registry Setting

```powershell
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\DataCollection" `
  -Name "AllowTelemetry"
```

**Expected Output:**
```
AllowTelemetry : 0
```

### 🔸 Re-scan with Tenable

| STIG Control        | WN10-CC-000205 |
|---------------------|----------------|
| Compliance Status   | ✅ **Pass**     |
| Verified Value      | `AllowTelemetry = 0` |

📸 **Screenshot Placeholder:** `Screenshot_04_Tenable_Telemetry_Pass.png`

---

## 🔐 Security Rationale

Windows Telemetry controls how much diagnostic data is sent to Microsoft. Full telemetry (value `3`) includes:

- App usage statistics  
- System events  
- Potentially sensitive logs

STIGs and federal guidance require that only **minimal (Security level)** telemetry be collected in enterprise or government environments.

### Compliance References

| Framework        | Control                                     |
|------------------|---------------------------------------------|
| **DISA STIG**    | WN10-CC-000205                              |
| **HIPAA**        | §164.308(a)(1)(ii)(A) – Risk Management      |
| **NIST 800-53**  | SI-4, AC-20 – System Monitoring, Use Control |
| **CMMC**         | SC.3.183 – Limit data collection             |

---

## 🧼 Post-Lab Cleanup

- 🔄 Reboot VM to enforce all registry changes
- 🧹 Delete Azure Resource Group to clean up lab:

```bash
az group delete --name vm-lab-telemetry --yes --no-wait
```

- 🔐 Remove cached credentials from Tenable

---

## 📎 Appendix: PowerShell Commands

| Task                     | PowerShell Command |
|--------------------------|--------------------|
| Simulate Vulnerability   | `Set-ItemProperty -Path HKLM:\...\DataCollection -Name AllowTelemetry -Value 3` |
| Apply Secure Config      | `Set-ItemProperty -Path HKLM:\...\DataCollection -Name AllowTelemetry -Value 0` |
| Verify Setting           | `Get-ItemProperty -Path HKLM:\...\DataCollection -Name AllowTelemetry` |
| Enable Remote Access     | `Set-ItemProperty -Path HKLM:\...\System -Name LocalAccountTokenFilterPolicy -Value 1` |

---

✅ **Lab Complete**

You've now simulated and remediated a real-world privacy control issue using Tenable + Azure + PowerShell. Continue documenting additional STIG controls as part of your secure baseline rollout.

