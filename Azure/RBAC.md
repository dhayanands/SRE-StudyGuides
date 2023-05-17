# Role Based Access Control (RBAC)

- [Role Based Access Control (RBAC)](#role-based-access-control-rbac)
  - [Assigning permission](#assigning-permission)

## Assigning permission

- Can use RBAC to assign permission to 
  - users
  - groups
  - applications
- Scope of the role assignment can be:
  - subscription
  - resource group
  - single resource

|                          |                                                                                                                           |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------- |
| **Security Principals**  | **Description**                                                                                                           |
| Users                    | - who has a profile in Azure AD or another Tenant                                                                         |
| Groups                   | - Multiple users can be assigned to a group                                                                               |
| Service Principal        | - security ID for applications or services to access Azure resources                                                      |
| Managed Identity         | - Used when developing cloud applications to handle credential management<br>- Automatically managed by Azure - eg: Vault |
| **Roles**                | **Description**                                                                                                           |
| Owner                    | Owner has full access to all resources and can grant access to other users                                                |
| Contributor              | Contributor can create and manage all resources but cannot grant access to others                                         |
| Reader                   | Reader can view existing resources                                                                                        |
| User Access Adminitrator | ets user to manage user access to Azure resources                                                                         |
| **Deny assignments**     | **Description**                                                                                                           |
|                          |                                                                                                                           |
