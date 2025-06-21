
#  Home Lab: Active Directory Group Strategy and Group Policy Implementation

##  Table of Contents
- [Project Overview](#project-overview)
- [Business Requirements](#business-requirements)
- [IGDLA Strategy Explanation](#igdla-strategy-explanation)
- [Architecture Design](#architecture-design)
- [Implementation Steps](#implementation-steps)
- [Security Configuration](#security-configuration)
- [Group Policy Management](#group-policy-management)
- [Testing and Validation](#testing-and-validation)

##  Project Overview

![image](https://github.com/user-attachments/assets/55d0ffe4-f5c2-4ebf-97fb-1e156730225c)

***Ref 1 - Scenario***

This project implements an Active Directory infrastructure for BP Crop using the IGDLA (Identity, Global, Domain Local, Access) strategy to manage access control for their Marketing division, including Research and Advertising departments.

### Environment
- **Domain:** mysam.local (for demonstration)
- **Platform:** Windows Server Active Directory Domain Services
- **Client Systems:** Windows domain-joined workstations

##  Business Requirements

BP Crop needs to:
1. **Enable quick market response** - Marketing data must be accessible to all marketing personnel
2. **Executive access** - Executives need read-only access to all marketing data
3. **Department segregation** - Separate access controls for Research and Advertising departments
4. **Cross-department collaboration** - Each department needs read access to the other's data
5. **Minimal administrative overhead** - Efficient group management structure

### Access Requirements Matrix

| User Group | Research Data | Advertising Data |
|------------|---------------|------------------|
| Executives | Read Only | Read Only |
| Research Staff | Read/Write | Read Only |
| Advertising Staff | Read Only | Read/Write |

##  IGDLA Strategy Explanation

IGDLA (Identity, Global, Domain Local, Access) is a Microsoft best practice for organizing Active Directory security groups:

### Strategy Breakdown
- **I (Identity)** - User accounts are placed in Identity groups
- **G (Global)** - Identity groups are members of Global groups
- **DL (Domain Local)** - Global groups are members of Domain Local groups  
- **A (Access)** - Domain Local groups are assigned permissions to resources

### Benefits
- **Scalability** - Easy to manage as organization grows
- **Flexibility** - Simple to modify permissions without touching individual users
- **Security** - Clear separation of roles and responsibilities
- **Administrative Efficiency** - Reduced overhead for permission management

##  Architecture Design

### Group Structure

#### Identity Groups (I_)
- `I_Executive` - Executive users
- `I_Research` - Research department users  
- `I_Advertisement` - Advertising department users

#### Global Groups (G_)
- `G_Executive` - Executive global group
- `G_Research` - Research global group
- `G_Advertisement` - Advertising global group

#### Domain Local Groups (DL_)
- `DL_Res_Folder_RO` - Research folder read-only access
- `DL_Res_Folder_RW` - Research folder read/write access
- `DL_Adv_Folder_RO` - Advertising folder read-only access
- `DL_Adv_Folder_RW` - Advertising folder read/write access

#### Resources
- `C:\Research_Data_Folder` - Research department data
- `C:\Advertisement_Data_Folder` - Advertising department data

### Group Membership Flow

![image](https://github.com/user-attachments/assets/2cd9fa21-ab4c-4fe7-8369-b39eccc3765b)

***Ref 2 - Flow diagram***

## Implementation Steps

### Step 1: Domain Setup
1. Install Windows Server with Active Directory Domain Services
2. Promote server to Domain Controller
3. Configure domain: `mysam.local` (demo) 
4. Join client machines to domain (Please review my other Active Directory project for reference)

### Step 2: Organizational Structure
1. On your **domain controller**, press `Windows + R`, type `dsa.msc`, and press **Enter**  
  → This will launch the **Active Directory Users and Computers** console.
2. In the left-hand panel, expand the domain (e.g., `mysam.local`)
3. Right-click on the domain name `mysam.local` - Select: New > Organizational Unit
4. Name the OU: BP Crop

### Step 3: Create Folder Structure
```bash
# Create folders in C:\ root
mkdir C:\Research_Data_Folder
mkdir C:\Advertisement_Data_Folder
```

### Step 4: Create User Accounts (Identity Groups)
```powershell
# Create Identity users
New-ADUser -Name "I_Executive" -UserPrincipalName "I_Executive@mysam.local" -Path "OU=BP Crop,DC=mysam,DC=local" -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) -Enabled $true

New-ADUser -Name "I_Research" -UserPrincipalName "I_Research@mysam.local" -Path "OU=BP Crop,DC=mysam,DC=local" -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) -Enabled $true

New-ADUser -Name "I_Advertisement" -UserPrincipalName "I_Advertisement@mysam.local" -Path "OU=BP Crop,DC=mysam,DC=local" -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) -Enabled $true
```

### Step 5: Create Global Groups
```powershell
# Create Global Groups
New-ADGroup -Name "G_Executive" -GroupScope Global -Path "OU=BP Crop,DC=mysam,DC=local"
New-ADGroup -Name "G_Research" -GroupScope Global -Path "OU=BP Crop,DC=mysam,DC=local"
New-ADGroup -Name "G_Advertisement" -GroupScope Global -Path "OU=BP Crop,DC=mysam,DC=local"
```

### Step 6: Create Domain Local Groups
```powershell
# Create Domain Local Groups
New-ADGroup -Name "DL_Res_Folder_RO" -GroupScope DomainLocal -Path "OU=BP Crop,DC=mysam,DC=local"
New-ADGroup -Name "DL_Res_Folder_RW" -GroupScope DomainLocal -Path "OU=BP Crop,DC=mysam,DC=local"
New-ADGroup -Name "DL_Adv_Folder_RO" -GroupScope DomainLocal -Path "OU=BP Crop,DC=mysam,DC=local"
New-ADGroup -Name "DL_Adv_Folder_RW" -GroupScope DomainLocal -Path "OU=BP Crop,DC=mysam,DC=local"
```

### Step 7: Configure Group Memberships
```powershell
# Add Identity users to Global groups
Add-ADGroupMember -Identity "G_Executive" -Members "I_Executive"
Add-ADGroupMember -Identity "G_Research" -Members "I_Research"  
Add-ADGroupMember -Identity "G_Advertisement" -Members "I_Advertisement"

# Add Global groups to Domain Local groups
# Executive access (RO to both folders)
Add-ADGroupMember -Identity "DL_Res_Folder_RO" -Members "G_Executive"
Add-ADGroupMember -Identity "DL_Adv_Folder_RO" -Members "G_Executive"

# Research access (RW to Research, RO to Advertising)
Add-ADGroupMember -Identity "DL_Res_Folder_RW" -Members "G_Research"
Add-ADGroupMember -Identity "DL_Adv_Folder_RO" -Members "G_Research"

# Advertising access (RO to Research, RW to Advertising)  
Add-ADGroupMember -Identity "DL_Res_Folder_RO" -Members "G_Advertisement"
Add-ADGroupMember -Identity "DL_Adv_Folder_RW" -Members "G_Advertisement"
```
![image](https://github.com/user-attachments/assets/930bb963-f9ac-4903-bec3-32148dc4f9e1)

***Ref 3 - Created Required Users, global groups & domain local groups***
 
##  Security Configuration

### Step 8: Configure Folder Permissions

#### Research Data Folder
1. **Share Permissions:** Everyone - Full Control
2. **NTFS Permissions:**
   - Remove inherited permissions
   - Remove Users group
   - Add `DL_Res_Folder_RW` - Full Control
   - Add `DL_Res_Folder_RO` - Read & Execute, List folder contents, Read

![image](https://github.com/user-attachments/assets/2e1535c0-38c1-4742-80a4-ba95543d012a)

***Ref 4 - Remove inherited permissions***

#### Advertisement Data Folder  
1. **Share Permissions:** Everyone - Full Control
2. **NTFS Permissions:**
   - Remove inherited permissions
   - Remove Users group
   - Add `DL_Adv_Folder_RW` - Full Control
   - Add `DL_Adv_Folder_RO` - Read & Execute, List folder contents, Read

### Verify Permissions with ICACLS
```cmd
icacls C:\Research_Data_Folder
icacls C:\Advertisement_Data_Folder
```
![image](https://github.com/user-attachments/assets/1fbae22a-fd40-40f1-a009-92b56b061e2b)

***Ref 5 - Verify ACLs***

### Permission Testing – Folder Access Verification

To validate that NTFS permissions are correctly enforced for the `I_EXECUTIVE` user, a test login was performed on a domain-joined client machine.

#### Test Scenario:
- **User**: `I_EXECUTIVE`
- **Target Folder**: `\\ServerName\Research_Data_Folder`
- **Expected Access**: Read-Only

#### Test Result:
When attempting to write to the `Research_Data_Folder`, the following access denied error was encountered:

![image](https://github.com/user-attachments/assets/303d6108-d97e-4ed5-9cac-8b958c83b366)

***Ref 6 - Result***

##  Group Policy Management

### Policy 1: Delegation of Control 
Q-Create an OU called Research. Delegate control of this OU to the previously created user account I_Research so that they can create, delete, and manage user accounts and groups.
To allow `I_Research` limited administrative rights, we delegate control of the `Research` OU so they can manage users and groups.

#### Steps:

1. Open **Active Directory Users and Computers** (`dsa.msc`)
2. Right-click the domain → `New > Organizational Unit` → Name it `Research`
3. Right-click the `Research` OU → Select `Delegate Control...`
4. In the wizard:
   - Add `I_Research` as the user
   - Select `Create a custom task to delegate`
   - Choose `Only the following objects in the folder` → Check `User objects` and `Group objects`
   - Assign permissions:
     - Create/Delete selected objects
     - Read/Write, Change Password, Reset Password
5. Click **Finish**
   
![image](https://github.com/user-attachments/assets/de14a4ff-0be4-41d1-9a37-1ff3acc2b0f3)

***Ref 7 - Task to Delegate***

> ✅ Now `I_Research` can manage user and group accounts only within the `Research` OU, following least privilege principles.

#### Policy 2: Account Lockout (Wrong Password Policy)
Q-Ensure that users are locked out for 90 minutes if they enter the wrong password 4 times

#### Steps:

1. Open `Group Policy Management` (`gpmc.msc`)
2. Create a new GPO named: `Wrong Password Policy`
3. Navigate to:
   Computer Configuration >
     Policies >
       Windows Settings >
         Security Settings >
           Account Policies >
             Account Lockout Policy
4. Set:
   - Threshold: 4 invalid attempts  
   - Lockout Duration: 90 minutes  
   - Reset Counter After: 90 minutes

![image](https://github.com/user-attachments/assets/99320661-854f-43e6-bf98-5cb02fd4cf83)

***Ref 8 - Policy is working***

Optional Command-Line:
```powershell
net accounts /lockoutthreshold:4 /lockoutduration:90 /lockoutwindow:90
```

---

#### Policy 3: Audit Logon Activity
Q-Audit all attempts to log on to the domain. Demonstrate that the
policy works Computer Configuration 

#### Steps:

1. In the same (or another) GPO, navigate to:
   Computer Configuration >
     Policies >
       Windows Settings >
         Security Settings >
           Advanced Audit Policy Configuration >
             Audit Policies >
               Logon/Logoff
2. Enable:
   - Audit Logon: Success and Failure

3. Apply changes:
```cmd
gpupdate /force
```

4. Monitor Logs via Event Viewer:
   - Event ID 4624 → Successful logon  
   - Event ID 4625 → Failed logon attempt

![image](https://github.com/user-attachments/assets/e7b8eda3-2c56-4f81-adf7-a5d9b4f2a6ff)

***Ref 9 - Event Viewer***

✅ These policies strengthen system security by locking out accounts after multiple failed attempts and allowing login activity monitoring through the Event Viewer.

