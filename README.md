
# Active Directory IAM Lab — MOVER

## Employee Role Change | Access Modification | RBAC | Least Privilege

![Status](https://img.shields.io/badge/Project-Completed-brightgreen)
![Technology](https://img.shields.io/badge/Technology-Active_Directory-blue)
![Security](https://img.shields.io/badge/Security-IAM-orange)

---

## Project Overview

This project demonstrates the **Mover phase of the Identity and Access Management (IAM) lifecycle**, focusing on modifying employee access privileges when job responsibilities change.

Using Microsoft Active Directory, Windows Server, PowerShell, and Microsoft Azure Virtual Machines, I simulated an employee transitioning into a Finance role requiring elevated file permissions.

The project involved reviewing account attributes, updating security group membership, validating NTFS and SMB permissions, and testing resource access from a domain-joined Windows Client VM.

**Project objectives:**

- Demonstrate Role-Based Access Control (RBAC).
- Apply the Principle of Least Privilege (PoLP).
- Modify employee access following a role change.
- Verify appropriate read/write permissions.
- Confirm that existing read-only restrictions remain enforced.

---

## Technologies and Environments Used

- Microsoft Azure Virtual Machines
- Windows Server
- Active Directory Domain Services (AD DS)
- Active Directory Users and Computers (ADUC)
- Windows Client VM
- Windows PowerShell
- NTFS Permissions
- SMB File Sharing
- Active Directory Security Groups
- Organizational Units (OUs)

### Lab Infrastructure

| Component | Description |
|---|---|
| Domain | myshitbox.com |
| Domain Controller | DC-1 |
| Client Machine | Windows Client VM |
| Directory Service | Active Directory |
| IAM Administration | ADUC / PowerShell |
| File Server | DC-1 |
| Shared Resource | `\\DC-1\Finance` |
| Employee | Sarah Williams |
| Department | Finance |

---

## Business Scenario

Sarah Williams transitioned into a Finance role requiring additional access privileges.

As the IAM administrator, I reviewed her Active Directory account, updated the appropriate security group membership, and verified that her permissions aligned with her new responsibilities.

The Finance environment contained two security groups:

| Security Group | Access Level |
|---|---|
| `GG_FINANCE_READONLY` | Read-only access |
| `GG_FINANCE_READWRITE` | Read and modify access |

The goal was to grant Sarah the permissions required for her Finance responsibilities without granting unnecessary access to other employees.

---

# Phase 1 — Active Directory Account Review

## Step 1: Review Sarah Williams's Group Membership

I opened Active Directory Users and Computers (ADUC) and reviewed Sarah Williams's account and security group membership.

This allowed me to verify that Sarah belonged to the appropriate Finance security group.

![Sarah Williams Group Membership](01-sarah-readwrite-group-membership.png)

**Figure 1:** Sarah Williams's Active Directory security group membership.

---

## Step 2: Verify Sarah's Account Using PowerShell

I used PowerShell to review Sarah's Active Directory account properties and assigned security groups.

**Account verification:**

```powershell
Get-ADUser -Identity "sarah.williams" -Properties Department,Title,Enabled |
Select-Object Name,SamAccountName,Department,Title,Enabled
```

**Security group verification:**

```powershell
Get-ADPrincipalGroupMembership "sarah.williams" |
Select-Object Name
```

These commands allow administrators to verify user account information and assigned access groups.

![Sarah PowerShell Verification](02-sarah-account-powershell-verification.png)

**Figure 2:** PowerShell verification of Sarah Williams's account and group membership.

---

# Phase 2 — Role-Based Access Control

## Step 3: Validate Read-Only Access Restrictions

Before validating elevated permissions, I reviewed the behavior of an account assigned read-only access to the Finance shared folder.

The account could access existing departmental resources but could not save unauthorized changes.

This demonstrated the importance of least-privilege access controls.

![Finance Read-Only Test](03-finance-readonly-control-test.png)

**Figure 3:** Read-only access test demonstrating restricted file modification.

---

## Step 4: Update Employee Role Information

I reviewed and updated Sarah Williams's Active Directory account attributes to reflect her new Finance responsibilities.

Maintaining accurate employee information supports identity governance and access management.

![Sarah Updated Job Title](04-sarah-updated-job-title.png)

**Figure 4:** Sarah Williams's updated job title in Active Directory.

---

## Step 5: Review Finance Security Groups

I reviewed the Finance security groups configured in Active Directory.

These groups separate employees requiring read-only access from employees who need permission to modify departmental files.

**Access structure:**

| Security Group | NTFS Permission |
|---|---|
| `GG_FINANCE_READONLY` | Read & Execute |
| `GG_FINANCE_READWRITE` | Modify |

Using security groups simplifies permission administration and helps enforce RBAC.

![Finance Read-Only Security Group](05-finance-readonly-group-reference.png)

**Figure 5:** Finance read-only security group configuration in ADUC.

---

## Step 6: Review Finance Organizational Unit

I reviewed the Finance organizational unit and verified the employee account's placement within the Active Directory structure.

Organizational units help administrators organize identities by department and apply administrative policies.

![Finance Organizational Unit](06-finance-analyst-ou-review.png)

**Figure 6:** Finance organizational unit review in Active Directory.

---

# Phase 3 — Finance Resource Access Testing

## Step 7: Test Finance Folder File Creation

After reviewing Sarah's updated permissions, I used the Windows Client VM to access the Finance shared folder.

**Network path:**

```text
\\DC-1\Finance
```

I performed a file creation test using:

```text
manager_test.txt
```

The objective was to confirm that Sarah could create files within the Finance shared folder.

![Sarah Manager Test File](07-sarah-manager-test-file-created.png)

**Figure 7:** Manager test file created within the Finance shared folder.

---

## Step 8: Open the Test File

I opened the manager test file using Notepad on the Windows Client VM.

This confirmed that the file was accessible through the Finance network share.

![Sarah Manager Test Opened](08-sarah-manager-test-file-opened.png)

**Figure 8:** Manager test file opened in Notepad.

---

# Phase 4 — SMB and NTFS Permission Verification

## Step 9: Review SMB Share Permissions

I used PowerShell on the Domain Controller to inspect the Finance SMB share permissions.

**PowerShell command:**

```powershell
Get-SmbShareAccess -Name "Finance"
```

**Share permissions:**

| Security Group | SMB Permission |
|---|---|
| `GG_FINANCE_READONLY` | Read |
| `GG_FINANCE_READWRITE` | Change |

These permissions support role-based access management.

![Finance SMB Share Permissions](09-finance-smb-share-access-verification.png)

**Figure 9:** PowerShell verification of Finance SMB share permissions.

### Understanding SMB and NTFS Permissions

Windows file access depends on both SMB share permissions and NTFS permissions.

When accessing a shared folder over the network, the effective permissions are limited by the applicable restrictions at both layers.

In this lab:

- Read-only users were assigned Read & Execute permissions.
- Read/write users were assigned Modify permissions.
- Administrative accounts retained appropriate administrative access.

---

# Phase 5 — File Modification and Deletion Testing

## Step 10: Validate File Modification

I tested Sarah Williams's ability to edit the manager test file within the Finance shared folder.

The objective was to verify that her assigned permissions supported the file operations required by her updated responsibilities.

![Sarah File Write Validation](10-sarah-file-write-validation.png)

**Figure 10:** File modification test using the manager test document.

---

## Step 11: Test File Deletion

I initiated a file deletion test to evaluate the permissions associated with the Finance read/write role.

The screenshot captures the Windows deletion confirmation prompt.

![Sarah File Delete Confirmation](11-sarah-file-delete-confirmation.png)

**Figure 11:** File deletion confirmation during access testing.

---

## Step 12: Review Finance Folder After Testing

After performing file access tests, I reviewed the Finance shared folder to inspect its contents.

This provided an additional opportunity to validate the state of the shared resource following testing.

![Finance Folder After Test](12-finance-folder-after-file-test.png)

**Figure 12:** Finance shared folder after file-operation testing.

---

# Phase 6 — Least-Privilege Regression Testing

## Step 13: Verify Marcus Johnson's Read-Only Restrictions

After validating Sarah Williams's updated Finance permissions, I performed an additional access test using Marcus Johnson's read-only account.

The objective was to ensure that granting Sarah elevated permissions did not unintentionally change access restrictions for other Finance employees.

Marcus attempted to modify and save an existing Finance document.

The operation was denied, demonstrating that the read-only security controls remained enforced.

<img width="1280" height="800" alt="13-marcus-readonly-regression-test" src="https://github.com/user-attachments/assets/325210c5-9fa1-417a-9d10-8f3180a4a2ed" />


**Figure 13:** Marcus Johnson's attempt to save modifications was denied, confirming continued least-privilege enforcement.

---

# Access Validation Results

| Test | Result |
|---|---|
| Review Sarah Williams's AD account | Completed |
| Verify security group membership | Completed |
| Review updated Finance job title | Completed |
| Review Finance organizational unit | Completed |
| Inspect SMB share permissions | Completed |
| Access Finance shared folder | Successful |
| Create manager test file | Successful |
| Open manager test file | Successful |
| Perform file modification test | Completed |
| Initiate file deletion test | Completed |
| Validate read-only access restrictions | Successful |
| Verify Marcus Johnson's write restrictions | Access Denied — PASS |

---

# IAM Security Concepts Demonstrated

### 1. Role-Based Access Control (RBAC)

Access was managed through Active Directory security groups based on departmental responsibilities.

This approach simplifies permission administration and supports consistent access assignments.

### 2. Principle of Least Privilege (PoLP)

Employees received permissions appropriate to their job responsibilities.

Read-only users were restricted from modifying protected Finance resources.

### 3. Identity Lifecycle Management

The Mover process demonstrated how IAM administrators manage access when employees change roles or responsibilities.

### 4. Security Group Management

Active Directory security groups were used to separate users with different permission requirements.

### 5. Access Validation

PowerShell and Windows Client VM testing were used to review identity configuration and resource permissions.

### 6. File System Security

NTFS and SMB permissions were reviewed to understand how access controls affect shared departmental resources.

### 7. Regression Testing

After modifying Sarah Williams's access, I verified that Marcus Johnson's read-only restrictions remained enforced.

This helped confirm that permission changes did not unintentionally grant excessive access to another employee.

---

# Skills Demonstrated

- Identity and Access Management (IAM)
- Active Directory Administration
- User Account Management
- Security Group Management
- Role-Based Access Control (RBAC)
- Principle of Least Privilege
- Employee Role Changes
- Windows Server Administration
- PowerShell Verification
- NTFS Permissions
- SMB Share Permissions
- Access Control Testing
- Identity Lifecycle Management
- Technical Documentation
- Access Regression Testing

---

# Project Outcome

Successfully completed an IAM Mover simulation involving an employee role change within a Windows Active Directory environment.

The project demonstrated practical experience reviewing user identities, managing role-based permissions, verifying security group membership, and testing access to departmental resources.

Additionally, regression testing demonstrated that read-only restrictions remained enforced for other Finance employees.

These activities reflect responsibilities commonly associated with Junior IAM Analysts, Identity Support Specialists, Access Management Analysts, and IT Administrators.

---

# Author

**Devin French**

CompTIA Security+ Certified

GitHub: https://github.com/devin-french

---

*This project was performed in a simulated Active Directory homelab using Microsoft Azure Virtual Machines.*
