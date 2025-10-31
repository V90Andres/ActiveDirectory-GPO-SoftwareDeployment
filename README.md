# ActiveDirectory-GPO-SoftwareDeployment
Automated software deployment and user access control using Active Directory Group Policy. Includes step-by-step configuration for 7-Zip deployment via MSI, Control Panel restriction policy, OU structure setup, GPO linking, permissions management, and troubleshooting logs.

![Windows Server](https://img.shields.io/badge/Windows%20Server-2022-blue?style=flat-square&logo=windows)
![Active Directory](https://img.shields.io/badge/Active%20Directory-GPO%20Automation-brightgreen?style=flat-square&logo=microsoft)
![PowerShell](https://img.shields.io/badge/PowerShell-Scripting-blue?style=flat-square&logo=powershell)
![Status](https://img.shields.io/badge/Status-Completed-success?style=flat-square)

---

## 🗂️ Project Overview
This project demonstrates **centralised software deployment** using **Active Directory Group Policy (GPO)**.  
The goal was to automatically install **7-Zip** across all domain-joined computers in the `LabUsers` OU — simulating how enterprise administrators automate installations at scale.

---

## 🧩 Infrastructure Overview

| Component | Description |
|------------|-------------|
| **Domain Controller (DC01)** | Windows Server 2022 running AD DS and GPO |
| **Client Machine (WIN10CLIENT)** | Windows 10 Pro joined to `lab.local` domain |
| **Domain** | `lab.local` |
| **Organizational Units (OUs)** | LabUsers → Peter Rendon → IT Team |
| **Software Deployed** | 7-Zip 25.01 (x64 edition) |
| **GPOs Used** | Disable Control Panel, Software Deployment – 7Zip |

---

## ⚙️ Technologies Used
- 🪟 **Windows Server 2022** (Domain Controller)  
- 💻 **Windows 10 Client Machine** (Domain Joined)  
- 🧱 **Active Directory Domain Services (AD DS)**  
- 🧩 **Group Policy Management Console (GPMC)**  
- ⚡ **PowerShell & Command Prompt** for admin automation  

---

## 🧱 Step-by-Step Implementation

## 🔒 1. Disable Control Panel for Lab Users

This policy restricts non-admin users from accessing the Windows Control Panel to maintain system security and prevent unwanted configuration changes.

### Steps Taken
1. Created a new **GPO** named **“Disable Control Panel”**.
2. Edited the policy under  
   **User Configuration → Administrative Templates → Control Panel → Prohibit access to Control Panel and PC settings** → **Enabled**.
3. Linked the GPO to the **LabUsers OU** only (not Admins or Domain Controllers).
4. Verified it applied successfully by logging in as **John Williams (Lab User)**.

### Verification
- When logged in as a Lab User, attempting to open the Control Panel shows a restriction message.  
- Confirmed with `gpresult /r` showing:
- [GPO_DisableControlPanel](ActiveDirectory_GPO_Project_Screenshots/GPO_Disable_Control_Panel_Settings.png)
- [GPO_LinkedToLabUsers](ActiveDirectory_GPO_Project_Screenshots/GPO_LinkedToLabUsers.png)
- [ControlPanel_Blocked](ActiveDirectory_GPO_Project_Screenshots/Control_Panel_Restriction.png)


### 🏗️ 2. OU and User Setup
- Created the following structure in **Active Directory Users and Computers (ADUC):**
  ```
  LabUsers
   └── Peter Rendon
        └── IT Team
             ├── John Williams
             ├── Maria Perez
             ├── Claire Donagham
             └── WIN10CLIENT
  ```
- Added domain users and computer objects inside the correct OUs.

---

### 🗂️ 3. Create and Share Software Folder
- On DC01, created `C:\Software`
- Shared it as `\\DC01\Software`
- Set permissions:
  - **Share Permissions:** `Everyone` → Read
  - **NTFS Permissions:** `Domain Computers` & `Everyone` → Read & Execute  

📸 *Screenshot: Software_Folder_Permissions.png*
-[Link 1 ](ActiveDirectory_GPO_Project_Screenshots/Software_Folder_Permissions.png)
-[Link 2](ActiveDirectory_GPO_Project_Screenshots/Software_Folder_Permissions1.png)

---

### 🧠 4. Create GPO for Software Deployment
- Opened **Group Policy Management Console (GPMC)**
- Created new GPO: **Software Deployment - 7Zip**
- Linked it to `LabUsers/Peter Rendon/IT Team`

📸 *Screenshot: GPO_Links_View.png*
-[Link](ActiveDirectory_GPO_Project_Screenshots/GPO_Links_View.png)

---

### ⚙️ 5. Configure GPO Settings
**Path:**  
`Computer Configuration → Policies → Software Settings → Software Installation`

- Added new package → `\\DC01\Software\7z2501-x64.msi`
- Chose **Assigned**
- Saved and closed GPMC

📸 *Screenshot: GPO_Software_Deployment_Settings.png*
[Link](ActiveDirectory_GPO_Project_Screenshots/GPO_Software_Deployment_Settings.png)

---

### 🔗 6. Move the Target Computer to the OU
Moved the **WIN10CLIENT** computer object to the **IT Team OU**  
to ensure the policy would apply during startup.

---

### 🔄 7. Force Policy Update & Restart
On the client machine, ran:
```cmd
gpupdate /force
shutdown /r /t 0
```
After reboot, the GPO was applied, and 7-Zip was installed automatically.

📸 *Screenshot: GPResult_Verification.png*
-[Link 1](ActiveDirectory_GPO_Project_Screenshots/GPResult_Verification_Command.png)

---

### ✅ 8. Verify Installation
Used these commands to confirm successful deployment:

```cmd
wmic product get name
gpresult /r
```
Also verified presence of:
```
C:\Program Files\7-Zip\7zFM.exe
```

📸 *Screenshot: Client_7Zip_Installed.png*
[Link](ActiveDirectory_GPO_Project_Screenshots/Client_7Zip_Installed.png)

---

## 🚧 Issues Encountered & Fixes

| Issue | Root Cause | Resolution |
|--------|-------------|-------------|
| GPO didn’t apply to users | GPO linked to wrong OU | Moved users and computer objects to correct OU |
| Software not installing | Path used `C:\Software` instead of UNC | Corrected to `\\DC01\Software` |
| Access denied to share | Incorrect NTFS/share permissions | Added read access for Domain Computers |
| Policy not visible in gpresult | Filtering disabled | Enabled Authenticated Users under Security Filtering |
| Control Panel blocked | GPO restriction applied | Verified install via command line and file explorer |

📸 *Screenshot: Troubleshooting_Gpresult.png*
[Link1](ActiveDirectory_GPO_Project_Screenshots/GPResult_Verification.png)
[Link2](ActiveDirectory_GPO_Project_Screenshots/GPResult_Verification1.png)

---

## 🔍 Verification Results

✅ **7-Zip automatically deployed**  
✅ **Control Panel restrictions applied**  
✅ **Group Policies linked, filtered, and enforced correctly**  
✅ **OU and domain structure verified through ADUC**  

📸 *Screenshot: OU_Structure_ADUC.png*
[Link](ActiveDirectory_GPO_Project_Screenshots/OU_Structure_ADUC.png)

---

## 🧾 Key Learning Outcomes
- Mastered **GPO-based MSI software deployment**
- Understood **OU targeting**, **security filtering**, and **enforcement**
- Practiced **permissions configuration** for shared deployment folders
- Learned to troubleshoot **Group Policy application issues**
- Combined **policy automation and security restrictions**

---

## 🧰 Useful Commands Reference
```cmd
gpupdate /force
gpresult /r
net view \\DC01
wmic product get name
```

---

## 🧩 Screenshot Documentation

| Screenshot | Description |
|-------------|-------------|
| **OU_Structure_ADUC.png** | ADUC structure showing users and computers in LabUsers OU |
| **GPO_Links_View.png** | GPO links for Disable Control Panel and Software Deployment |
| **GPO_Software_Deployment_Settings.png** | Software Installation GPO configuration |
| **Software_Folder_Permissions.png** | Folder sharing and security permissions |
| **GPResult_Verification.png** | gpresult output confirming applied GPOs |
| **Client_7Zip_Installed.png** | 7-Zip installation verified on the client |

---

## 🧭 Project Impact
This project demonstrates enterprise-level **automation and management** through Active Directory, combining:
- Centralised control  
- Security enforcement  
- Hands-off software deployment  

It’s an essential skill for **system administrators**, **IT support engineers**, and **network administrators**.

---

## 👨‍💻 Author

**Andrés Vanegas**  
📍 London, UK  
🎓 BSc in Computing | IT Systems & Network Enthusiast  
💼 Vending Operator @ UBS Bank | Aspiring IT Professional  
🌐 [Tortuga Island Headwear](#) | 💻 [LinkedIn Profile](#)
