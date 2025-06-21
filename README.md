
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

```
Identity Groups → Global Groups → Domain Local Groups → Folder Permissions

I_Executive → G_Executive → DL_Res_Folder_RO + DL_Adv_Folder_RO
I_Research → G_Research → DL_Res_Folder_RW + DL_Adv_Folder_RO  
I_Advertisement → G_Advertisement → DL_Res_Folder_RO + DL_Adv_Folder_RW
```

## Implementation Steps

### Step 1: Domain Setup
1. Install Windows Server with Active Directory Domain Services
2. Promote server to Domain Controller
3. Configure domain: `mysam.local` (demo) / `swin.com` (production)
4. Join client machines to domain

### Step 2: Organizational Structure
1. Create Organization: **BP Crop**
2. Create Organizational Units as needed for department structure

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

##  Security Configuration

### Step 8: Configure Folder Permissions

#### Research Data Folder
1. **Share Permissions:** Everyone - Full Control
2. **NTFS Permissions:**
   - Remove inherited permissions
   - Remove Users group
   - Add `DL_Res_Folder_RW` - Full Control
   - Add `DL_Res_Folder_RO` - Read & Execute, List folder contents, Read

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

## 📋 Group Policy Management

### Policy 1: Delegation of Control
1. Create OU: **Research**
2. Delegate control to `I_Research` user
3. Grant permissions:
   - Create, delete, and manage user accounts
   - Create, delete, and manage groups

### Policy 2: Account Lockout Policy
**Policy Name:** Wrong Password Policy
**Location:** Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Account Lockout 
- **Account lockout threshold:** 4 invalid attempts
- **Account lockout duration:** 90 minutes
- **Reset account lockout counter:** 90 minutes

```powershell
# Apply via Group Policy or directly
net accounts /lockoutthreshold:4 /lockoutduration:90 /lockoutwindow:90
```

### Policy 3: Audit Policy Configuration
**Location:** Computer Configuration > Policies > Windows Settings > Security Settings > Advanced Audit Policy Configuration > Audit Policies > Logon/Logoff

**Configure:**
- Audit Logon: Success and Failure
- Monitor Event IDs:
  - **4624** - Successful logon
  - **4625** - Failed logon attempt

**Apply Policy:**
```cmd
gpupdate /force
```

##  Testing and Validation

### Test 1: Executive Access Verification
1. Login as `I_Executive`
2. Access both Research and Advertisement folders
3. Verify read-only access (cannot create/modify files)
4. Confirm access to view folder contents

### Test 2: Department Access Testing
1. **Research User Test:**
   - Login as `I_Research`
   - Verify R/W access to Research folder
   - Verify RO access to Advertisement folder

2. **Advertisement User Test:**
   - Login as `I_Advertisement`  
   - Verify RO access to Research folder
   - Verify R/W access to Advertisement folder

### Test 3: Security Policy Validation
1. **Account Lockout Test:**
   - Attempt 4 incorrect passwords
   - Verify account locks for 90 minutes
   - Check Event Viewer for Event ID 4625

2. **Audit Log Verification:**
   - Check Windows Logs > Security
   - Verify Event ID 4624 for successful logons
   - Verify Event ID 4625 for failed attempts
