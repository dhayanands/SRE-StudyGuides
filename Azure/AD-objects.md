# Active Directory Objects

## User Accounts

### User account types

Cloud Identities
    - Local Azure AD
    - External Azure AD (Azure AD business to Business)

Hybrid Identities
    - Directory synchronized (on-prem & Azure)

Guest Identities
    - Azure AD B2B collaborations (can be from another AD or from Google or any email address )
    - External Identities

### Creating User

- Using Powershell

```powershell
## Install the AzureAD module
Install-Module -Name AzureAD

## Connect to AzureAD
Connect-AzureAD
```

- Bulk AD User Creation

  - Done using Azure Console
  - Bulk create, Bulk invite & Bulk delete
  - Download User list as CSV file

## AD Groups

- Security Groups
  - used for resource access, application access etc.,

- Microsoft 365 Groups
  - only relevnant when we have MS365 license
  - for list, emails, sharepoint online, exchange disctribution lists etc.,

- Owners
  - delegate group ownership to a specific user who might not have admin privileges in Azure AD

### Memebership Types

- Assigned Membership
  - User (group owner) controlled group and the users are manully added / deleted from the group

- Dynamic Membership
  - Azure AD controls group memebership of the user based on expressions created using user properties
  - Dynamic groups will have owners who can manage the expressions & licences for the group

### Group Assigned Roles & Licences

- Automate the distribution of licences : Azure AD premium licenses, MS Intune licences, MS365 licences etc., by assigning to group
- Also applicable for RBAC

## Administrative Units

- Azure Administrative Units (AUs) are Azure AD user and group containers very similar to Oraganization Units (OUs)
- Used for:
  - Logically organize the Azure AD users & groups
  - to delegate administrative permissions

## Administer Devices

**Azure AD Join vs Registration:**
  
- AD Join
  - Corporate owned devices
  - Windows 10 1809 and later, Windows Server 2019 VMs in Azure
  - Must be Azure AD joined & system assigned managed identity
  - Must have RBAC access for VMs with either groups: Virtual Machine Administrator Login or Virtual Machine User Login

- Registration
  - BYOD Devices
  - Windows 10, iOS, Android, macOS

- Hybrid Azure AD Join
  - Hybrid environment - on-prem AD Service is in syncc with Azure Tenant AD using Azure AD Connect
  - Corporate owned devices with AD DS sign-in with cloud identity (AADC - Azure AD Connect)
  For Windows 7, 8, 10 and Windows Server 2018 or later

**MyApps Portal:**

- myapplications.microsoft.com
- SSO access to cloud apps