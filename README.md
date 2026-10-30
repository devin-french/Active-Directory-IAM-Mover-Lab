
# Active Directory IAM Lab — MOVER
## Employee Role Change | Access Modification | RBAC | Least Privilege

![Status](https://img.shields.io/badge/Project-Completed-brightgreen)
![Technology](https://img.shields.io/badge/Technology-Active_Directory-blue)
![Security](https://img.shields.io/badge/Security-IAM-orange)

---

# Project Overview

This project demonstrates the **Mover phase of the Identity and Access Management (IAM) lifecycle**, focusing on modifying employee access privileges when their job responsibilities change.

Using Microsoft Active Directory, Windows Server, PowerShell, and Azure Virtual Machines, I simulated an employee transitioning into a Finance role requiring elevated file permissions.

The project involved reviewing account attributes, updating security group membership, validating NTFS and SMB permissions, and testing resource access from a domain-joined Windows Client VM.

The objective was to demonstrate **Role-Based Access Control (RBAC)**, the **Principle of Least Privilege (PoLP)**, and identity lifecycle management.

---

# Technologies and Environments Used

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

## Lab Infrastructure

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

# Business Scenario

Sarah Williams was assigned a Finance role requiring additional access privileges.

As the IAM administrator, I was responsible for reviewing her Active Directory account, updating the appropriate security group membership, and verifying that her permissions aligned with her new job responsibilities.

The Finance environment contained two security groups:

- **GG_FINANCE_READONLY** — Read-only access
- **GG_FINANCE_READWRITE** — Read and modify access

The objective was to provide Sarah with the appropriate permissions while ensuring employees assigned read-only access remained restricted.

---

# Phase 1 — Review Existing Account and Permissions

## Step 1: Review Active Directory Group Membership

I opened Active Directory Users and Computers (ADUC) and reviewed Sarah Williams's account and security group membership.

This allowed me to identify her assigned access and verify that she belonged to the appropriate Finance security group.

![Sarah Williams Group Membership](01-sarah-readwrite-group-membership.png)

**Figure 1:** Sarah Williams's Active Directory security group membership.

---

## Step 2: Verify Account Information Using PowerShell

I used PowerShell to review Sarah's Active Directory account properties and security group assignments.

### Account Verification

```powershell
Get-ADUser -Identity "sarah.williams" -Properties Department,Title,Enabled |
Select-Object Name,SamAccountName,Department,Title,Enabled
```

### Security Group Verification

```powershell
Get-ADPrincipalGroupMembership "sarah.williams" |
Select-Object Name
```

These commands verify that the employee account exists and identify the security groups assigned to the user.

![Sarah PowerShell Verification](02-sarah-account-powershell-verification.png)

**Figure 2:** PowerShell verification of Sarah Williams's account and group membership.

---

# Phase 2 — Review Role-Based Access Controls

## Step 3: Validate Read-Only Access Restrictions

Before validating elevated permissions, I reviewed the behavior of an account assigned read-only access to the Finance shared folder.

The user could access existing departmental resources but could not save unauthorized changes.

This demonstrated the importance of enforcing least-privilege permissions.

![Finance Read Only Test](03-finance-readonly-control-test.png)

**Figure 3:** Read-only access test showing that unauthorized modifications were denied.

---

## Step 4: Update Employee Role Information

I reviewed and updated Sarah Williams's Active Directory account attributes to reflect her new Finance responsibilities.

Maintaining accurate employee information supports identity governance and access management.

![Sarah Updated Job Title](04-sarah-updated-job-title.png)

**Figure 4:** Sarah Williams's updated job title in Active Directory.

---

## Step 5: Review Finance Security Groups

I reviewed the Finance security groups configured in Active Directory.

The groups were designed to separate users requiring read-only access from users requiring permission to modify departmental files.

### Security Group Structure

| Security Group | Access Level |
|---|---|
| GG_FINANCE_READONLY | Read & Execute |
| GG_FINANCE_READWRITE | Modify |

Using security groups allows IAM administrators to manage access based on employee roles instead of assigning permissions directly to individual users.

![Finance Read Only Security Group](05-finance-readonly-group-reference.png)

**Figure 5:** Finance read-only security group configuration in ADUC.

---

## Step 6: Review Finance Organizational Unit

I reviewed the Finance organizational unit and verified the employee account's placement within the Active Directory structure.

Organizational units help administrators organize identities by department and apply administrative policies.

![Finance Organizational Unit](06-finance-analyst-ou-review.png)

**Figure 6:** Finance Analyst organizational unit in Active Directory.

---

# Phase 3 — Validate Elevated Access

## Step 7: Test Finance Folder File Creation

After reviewing Sarah's updated access, I used the Windows Client VM to test access to the Finance shared folder.

The shared resource was accessed using:

```text
\\DC-1\Finance
```

I performed a file creation test using a document named:

```text
manager_test.txt
```

This test confirmed that Sarah could perform file operations associated with her updated role.

![Sarah Manager Test File](07-sarah-manager-test-file-created.png)

**Figure 7:** Manager test file created within the Finance shared folder.

---

## Step 8: Open the Test File

I opened the manager test file using Notepad on the Client VM.

This verified that the file was accessible from the Finance network share.

![Sarah Manager Test Opened](08-sarah-manager-test-file-opened.png)

**Figure 8:** Manager test file opened in Notepad.

---

# Phase 4 — Verify SMB and NTFS Permissions

## Step 9: Review SMB Share Permissions

I used PowerShell on the Domain Controller to inspect the permissions configured on the Finance SMB share.

### PowerShell Command

```powershell
Get-SmbShareAccess -Name "Finance"
```

The share was configured with separate access levels for the Finance security groups.

| Security Group | SMB Permission |
|---|---|
| GG_FINANCE_READONLY | Read |
| GG_FINANCE_READWRITE | Change |

This configuration supports role-based access management.

![Finance SMB Share Permissions](09-finance-smb-share-access-verification.png)

**Figure 9:** PowerShell verification of Finance SMB share permissions.

### NTFS and SMB Permissions

Windows file access depends on both SMB share permissions and NTFS permissions.

The effective access granted to a user is limited by the applicable permissions at both layers.

For this project:

- Read-only users received Read & Execute permissions.
- Read/write users received Modify permissions.
- Administrative accounts retained appropriate administrative access.

---

# Phase 5 — Test File Modification and Cleanup

## Step 10: Validate File Modification

I tested Sarah Williams's ability to edit the manager test file within the Finance shared folder.

The purpose was to confirm that her assigned permissions supported the file operations required by her updated job responsibilities.

![Sarah File Write Validation](10-sarah-file-write-validation.png)

**Figure 10:** File modification test performed using the manager test document.

---

## Step 11: Test File Deletion

I initiated a file deletion test to evaluate the permissions associated with the Finance read/write role.

The screenshot captures the Windows deletion confirmation prompt.

![Sarah File Delete Confirmation](11-sarah-file-delete-confirmation.png)

**Figure 11:** File deletion confirmation during access testing.

---

## Step 12: Review Finance Folder After Testing

After performing file access tests, I reviewed the Finance shared folder to inspect its contents.

![Finance Folder After Test](12-finance-folder-after-file-test.png)

**Figure 12:** Finance shared folder after file-operation testing.

---

# Phase 6 — Least-Privilege Regression Testing

## Step 13: Verify Read-Only Restrictions Remain Enforced

After validating Sarah Williams's updated Finance permissions, I performed an additional access test using Marcus Johnson's read-only account.

The objective was to ensure that granting Sarah elevated permissions did not unintentionally modify access restrictions for other Finance employees.

Marcus attempted to modify and save an existing Finance document.

The operation was denied, confirming that the read-only security controls remained enforced.

![Marcus Read-Only Regression Test](13-marcus-readonly-regression-test.png)

**Figure 13:** Marcus Johnson's attempt to save modifications was denied, demonstrating continued least-privilege enforcement.

---

# Access Validation Results

The following table summarizes the activities performed during the Mover lab.

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

## 1. Role-Based Access Control (RBAC)

Access was managed through Active Directory security groups based on departmental responsibilities.

This approach simplifies permission administration and supports consistent access assignment.

## 2. Principle of Least Privilege (PoLP)

Employees were assigned permissions appropriate to their job responsibilities.

Read-only users were restricted from modifying protected Finance resources.

## 3. Identity Lifecycle Management

The Mover process demonstrated how IAM administrators manage access when an employee changes roles or responsibilities.

## 4. Security Group Management

Active Directory security groups were used to separate users with different permission requirements.

## 5. Access Validation

PowerShell and Windows Client VM testing were used to review identity configuration and resource permissions.

## 6. File System Security

NTFS and SMB permissions were reviewed to understand how access controls affect shared departmental resources.

## 7. Regression Testing

After modifying Sarah Williams's access, I verified that Marcus Johnson's read-only restrictions remained intact.

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

Additionally, regression testing confirmed that read-only restrictions remained enforced for other Finance employees.

These activities reflect responsibilities commonly associated with Junior IAM Analysts, Identity Support Specialists, Access Management Analysts, and IT Administrators.

---

# Author

**Devin French**

CompTIA Security+ Certified

GitHub: https://github.com/devin-french

---

*This project was performed in a simulated Active Directory homelab using Microsoft Azure Virtual Machines.*
