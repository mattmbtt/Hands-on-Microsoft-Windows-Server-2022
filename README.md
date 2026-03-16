# Hands-on Microsoft Windows Server 2022
## Exercise 1 — Configuring SMB and NFS File Shares

![Windows Server 2022](https://img.shields.io/badge/Windows%20Server-2022-0078D4?style=flat-square&logo=windows)
![PowerShell](https://img.shields.io/badge/PowerShell-5.1+-5391FE?style=flat-square&logo=powershell)
![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)

---

## Overview

This exercise covers configuring SMB (Server Message Block) and NFS (Network File System) file shares on Windows Server 2019/2022. You will create shares, manage permissions, query share state via PowerShell, and harden the environment by disabling the legacy SMB1 protocol.

---

## Lab Environment

| Device     | Role                            | OS                    |
|------------|---------------------------------|-----------------------|
| PLABDC01   | Domain Controller / File Server | Windows Server 2019   |
| PLABDM01   | Domain Member / NFS Host        | Windows Server 2019   |
| PLABWIN10  | Client Workstation              | Windows 10 Pro        |

---

## Learning Objectives

- Create an SMB share using Server Manager
- Install the **Server for NFS** Windows feature
- Create an NFS share with Kerberos v5 authentication
- Query and manage shares using PowerShell cmdlets
- Disable the legacy SMB1 protocol

---

## Repository Structure

```
.
├── README.md
├── scripts/
│   ├── 01_Create-SMBShare.ps1       # Task 1 — Automate SMB share creation
│   ├── 02_Install-NFSFeature.ps1    # Task 2 — Install Server for NFS
│   ├── 03_Create-NFSShare.ps1       # Task 3 — Create NFS share
│   ├── 04_Query-Shares.ps1          # Task 4 — View share info via PowerShell
│   └── 05_Disable-SMB1.ps1          # Task 5 — Disable legacy SMB1 protocol
```

---

## Tasks

### Task 1 — Create an SMB Share (Server Manager GUI)

1. On **PLABDC01**, open **Server Manager → File and Storage Services → Shares**
2. Click **TASKS → New Share…**
3. Select profile: **SMB Share – Quick** → Next
4. Select volume: **C:** → Next
5. Share name: `corpdata` → Next
6. Enable **Access-Based Enumeration** → Next
7. Click **Customize permissions…**
   - Add principal: `Domain Users`
   - Grant **Modify** permission
8. Click **Create**

> The share will be available at `\\PLABDC01\corpdata`

For the PowerShell equivalent, see [`scripts/01_Create-SMBShare.ps1`](scripts/01_Create-SMBShare.ps1)

---

### Task 2 — Install Server for NFS Feature

Run on **PLABDM01**. See [`scripts/02_Install-NFSFeature.ps1`](scripts/02_Install-NFSFeature.ps1)

```powershell
Add-WindowsFeature FS-FileServer, FS-NFS-Service -IncludeManagementTools
```

---

### Task 3 — Create an NFS Share

1. On **PLABDM01**, open **Server Manager → File and Storage Services → Shares**
2. Click **TASKS → New Share…**
3. Select profile: **NFS Share – Quick** → Next
4. Select volume: **D:** → Next
5. Share name: `LinuxFolder1` → Next
6. Authentication: enable **Kerberos v5 (Krb5)** → Next
7. Permissions: Add **All Machines** with **Read/Write** → Next
8. Click **Create**

For the PowerShell equivalent, see [`scripts/03_Create-NFSShare.ps1`](scripts/03_Create-NFSShare.ps1)

---

### Task 4 — Query Share Information via PowerShell

Run on **PLABDM01**. See [`scripts/04_Query-Shares.ps1`](scripts/04_Query-Shares.ps1)

```powershell
# List NFS shares
Get-NfsShare

# Detailed NFS share properties
Get-NfsShare LinuxFolder1 | fl *

# List local SMB shares
Get-SmbShare

# Reset WinHTTP proxy (required for CimSession in lab environments)
netsh winhttp reset proxy

# Establish a CIM session to the remote domain controller
$smb = New-CimSession -ComputerName plabdc01

# List SMB shares on PLABDC01 via CIM session
Get-SmbShare -CimSession $smb

# --- On PLABWIN10 ---
# Map network drive to the corpdata share
net use x: \\plabdc01\corpdata

# --- Back on PLABDM01 ---
# View active SMB sessions on PLABDC01
Get-SmbSession -CimSession $smb

# Detailed session info for a specific user
Get-SmbSession -CimSession $smb -ClientUserName practicelabs\administrator | fl *
```

---

### Task 5 — Disable Legacy SMB1 Protocol

Run on **PLABDC01**. See [`scripts/05_Disable-SMB1.ps1`](scripts/05_Disable-SMB1.ps1)

```powershell
# Audit current SMB server protocol configuration
Get-SmbServerConfiguration | fl enable*

# Disable SMB1
Set-SmbServerConfiguration -EnableSMB1Protocol $false
```

> **Security Note:** SMB1 is a legacy protocol with known critical vulnerabilities (e.g., EternalBlue / WannaCry). It must be disabled in all production environments where clients support SMB2 or SMB3.

---

## Prerequisites

- Windows Server 2019 or 2022
- Active Directory domain environment
- Local Administrator or Domain Admin privileges
- PowerShell 5.1+

---

## References

- [Microsoft Docs — SMB Overview](https://learn.microsoft.com/en-us/windows-server/storage/file-server/file-server-smb-overview)
- [Microsoft Docs — NFS Overview](https://learn.microsoft.com/en-us/windows-server/storage/nfs/nfs-overview)
- [Stop using SMB1 — Microsoft Tech Community](https://techcommunity.microsoft.com/t5/storage-at-microsoft/stop-using-smb1/ba-p/425858)

---

## License

This project is licensed under the [MIT License](LICENSE).
