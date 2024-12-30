# Azure Red Teaming Guide

## Overview
Azure Red Teaming focuses on identifying, simulating, and exploiting vulnerabilities in Microsoft Azure cloud environments. This guide covers all stages of a red team operation, from initial enumeration to persistence techniques, ensuring a comprehensive understanding of attack surfaces and defensive measures.

---

## Table of Contents
1. **Introduction to Azure Red Teaming**
2. **Enumeration**
   - Azure Environment Reconnaissance
   - Identifying Resources and Permissions
3. **Initial Access**
   - Exploiting Misconfigurations
   - Credential Harvesting
4. **Privilege Escalation**
   - Azure AD Privilege Escalation
   - Exploiting Role Assignments
5. **Lateral Movement**
   - Moving Between Azure Resources
   - Exploiting Virtual Machines and Storage Accounts
6. **Persistence**
   - Establishing Backdoors
   - Manipulating Azure AD Applications
7. **Tools and Scripts**
   - PowerShell Scripts
8. **Defensive Considerations**

---

## 1. Introduction to Azure Red Teaming
Azure Red Teaming focuses on attacking Azure-specific components, such as:
- Azure Active Directory (Azure AD)
- Virtual Machines (VMs)
- Key Vaults
- Role-Based Access Control (RBAC)

Key concepts include:
- **Azure Resource Manager (ARM):** Central to resource deployment and management.
- **Azure AD Authentication:** Backbone for identity and access control.
- **Networking:** Understanding VNets, NSGs, and Azure-specific routing.

---

## 2. Enumeration
### 2.1 Azure Environment Reconnaissance
- **Tools:**
  - `az cli`
  - [PowerZure](https://github.com/PowerZure/PowerZure)
  - [RoadRecon](https://github.com/RoadTools/RoadRecon)

**Steps:**
1. Enumerate subscriptions:
   ```bash
   az account list --output table
   ```
2. List resources in each subscription:
   ```bash
   az resource list --subscription <SUBSCRIPTION_ID>
   ```
3. Identify public-facing endpoints:
   - Azure Portal
   - `nmap` for scanning IP ranges

### 2.2 Identifying Resources and Permissions
- Enumerate RBAC roles:
  ```bash
  az role assignment list --output table
  ```
- Check for over-permissioned roles:
  - Owner
  - Contributor

---

## 3. Initial Access
### 3.1 Exploiting Misconfigurations
1. Misconfigured Azure Storage Accounts:
   - Look for public blob access:
     ```bash
     az storage container list --account-name <STORAGE_ACCOUNT>
     ```
2. Improper Network Security Groups (NSGs):
   - Scan for exposed ports.

### 3.2 Credential Harvesting
- Common sources:
  - Leaked credentials in GitHub repositories.
  - Azure AD Connect synchronization logs.
- Exploit exposed Service Principal credentials:
  ```bash
  az ad sp credential reset --name <SERVICE_PRINCIPAL_NAME>
  ```

---

## 4. Privilege Escalation
### 4.1 Azure AD Privilege Escalation
1. Abuse Azure AD Connect Sync:
   - Dump credentials using `Mimikatz` or similar tools.
2. Escalate to Global Admin:
   - Target misconfigured Privileged Identity Management (PIM).

### 4.2 Exploiting Role Assignments
- Identify roles with excessive permissions:
  ```bash
  az role assignment list --all
  ```
- Elevate roles via RBAC misconfigurations.

---

## 5. Lateral Movement
### 5.1 Moving Between Azure Resources
- Use Managed Identities to access other resources:
  ```bash
  az account get-access-token --resource <RESOURCE>
  ```
- Enumerate VMs for credential reuse.

### 5.2 Exploiting Virtual Machines and Storage Accounts
- Dump credentials from VMs using `BloodHound`.
- Extract sensitive data from Azure Storage Blobs.

---

## 6. Persistence
### 6.1 Establishing Backdoors
1. Create a new Service Principal:
   ```bash
   az ad sp create-for-rbac --name <NAME>
   ```
2. Assign elevated roles to the Service Principal.

### 6.2 Manipulating Azure AD Applications
- Add hidden permissions to existing applications.
- Use automation accounts to maintain persistence.

---

## 7. Tools and Scripts
- [PowerZure](https://github.com/PowerZure/PowerZure): Azure enumeration and attack tool.
- [AzureHound](https://github.com/BloodHoundAD/AzureHound): Enumerates Azure AD for BloodHound.
- [RoadTools](https://github.com/RoadTools): Azure AD exploration.

### PowerShell Scripts
#### List Azure Resources
```powershell
# Enumerate all resources in the current subscription
Get-AzResource | Select-Object ResourceName, ResourceType, Location
```

#### Check NSG Rules
```powershell
# List all Network Security Group (NSG) rules in a subscription
Get-AzNetworkSecurityGroup | ForEach-Object {
    $_.SecurityRules | Select-Object Name, Access, Direction, Priority, SourceAddressPrefix, DestinationAddressPrefix
}
```

#### Enumerate Azure AD Users
```powershell
# Get a list of all Azure AD users
Get-AzADUser | Select-Object DisplayName, UserPrincipalName, AccountEnabled
```

#### Identify Over-Permissioned Roles
```powershell
# Check for high-privileged roles assigned in the environment
Get-AzRoleAssignment | Where-Object { $_.RoleDefinitionName -in ('Owner', 'Contributor') } | Select-Object RoleDefinitionName, PrincipalName
```

#### Enable Persistence via Azure Automation Account
```powershell
# Create an automation account and assign a runbook for persistence
New-AzAutomationAccount -ResourceGroupName "<ResourceGroupName>" -Name "<AccountName>" -Location "<Location>"
New-AzAutomationRunbook -AutomationAccountName "<AccountName>" -Name "<RunbookName>" -Type PowerShell -ResourceGroupName "<ResourceGroupName>"
```

---

## 8. Defensive Considerations
1. **Enable Conditional Access Policies:**
   - Restrict access based on location, device, or risk level.
2. **Monitor Azure AD Sign-ins:**
   - Check for suspicious logins.
3. **Enable Logging:**
   - Enable Diagnostic Logs for all Azure resources.
4. **Review Role Assignments Regularly:**
   - Use least privilege principles.

---

This guide provides a solid technical foundation for Azure Red Teaming operations. Tailor the techniques to your specific engagement and always act ethically.
