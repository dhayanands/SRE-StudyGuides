#iam #gcloud

# Basics

## Account types

- **Corporate Identities** : 
	- for Organization employees to sign in into workstations, emails or any corp. applications. 
	- can also add non-employees.
- **Customer Identities**: 
	- accounts managed for users to access websites / customer facing applications
- **Service Identities**:
	- to enable applications to interact with other applications

## IAM Tasks 

![[IAM_Tasks.png]]

# IAM Architecture

![[IAM_Arch.png]]



## Google identities

- **Identity**
	- uniquely identifies the person who is interacting with a Google service. Google used the *email address* of the user for this.
- **User account**
	- data structure that keeps track of attributes, activities, and configurations that should be applied whenever a certain identity interacts with a Google service
	- User accounts are not created on the fly, but need to be provisioned before the first sign-on
	- User accounts are identified by an ID that is not exposed externally
	- **The relationship between user accounts and identities is not fixed.** You can change the primary email address of a user account, which associates a different identity with the user.
		![[IAM_Identity_Swap.png]]
	- **The relationship between identity and users might not be 1:1**. It's also possible that one identity refers to two different user accounts (NOT RECOMMENDED)
		![[IAM_Identity_to_User.png]]

## Google for consumers

- **Consumer account**
	- Consumer accounts are created by self-service and are primarily intended to be used for private purposes. 
	- The person who created the consumer account has full control of the account and any data created by using the account.
	- When a consumer account uses a primary email address that corresponds to the primary or secondary domain of a Cloud Identity or Google Workspace account, then the consumer account is also referred to as an ***unmanaged user account***.
	- Because you cannot add `gmail.com` as a domain to your Cloud Identity or Google Workspace account, only non-Gmail consumer accounts can ever become unmanaged user accounts.

## Google for organizations

- **Cloud Identity or Google Workspace account**
	- A Cloud Identity or Google Workspace account is not a user account, but a directory of user accounts.
	- A Cloud Identity or Google Workspace account is the top-level container for users, groups, configuration, and data.
	- It is created when a company signs up for Cloud Identity or Google Workspace and corresponds to the notion of a *tenant* and is identified by a domain name
	- Cloud Identity and Google Workspace share a common technical platform. For the purpose of managing users, groups, and authentication, the two products can largely be considered equivalent.
	- An account contains groups and one or more organizational units.
- **Organizational unit**
	- An organizational unit (OU) is a sub-container for user accounts and organized hierarchically
	- Used to segment the user accounts into disjoint sets to manage easily and apply certain configurations such as license assignment or 2-step verification.
	- Each Cloud Identity or Google Workspace account has a root OU, under which you create more OUs as needed. You can also nest your OUs.
	- A user account cannot belong to more than one OU, which makes OUs different from groups. Although OUs are useful for applying configuration to user accounts, they are not intended to be used for managing access.
	- Although OUs resemble Google Cloud folders, the two entities serve different purposes and are unrelated to another.
- **Managed user account**
	- Any account created on Cloud Identity and Google Workspace is called **managed user account** as its lifecycle and configuration can be fully controlled by the organization. 
	- The identity of a managed user account is defined by its primary email address. The primary email address has to use a domain that corresponds to one of the primary, secondary, or alias domains added to the Cloud Identity or Google Workspace account.
- **Group**
	- Groups let you bundle multiple users to manage a mailing list or to apply common access control or configuration to multiple users.
	- Cloud Identity and Google Workspace identify groups by email address—for example, `billing-admins@example.com`. Similar to a user's primary email address. The email address doesn't need to correspond to a mailbox unless the group is used as a mailing list. Authentication still happens using the user's email rather than the group email, so a user can't log in using a group email address.
	- A group can have the following entities as members:
		-   Users (managed users or consumer accounts)
		-   Other groups
		-   Service accounts

## Google Cloud

-  Google Cloud closely integrates with Google Workspace and Cloud Identity to manage resources efficiently.
- Google Cloud introduces the notion of organization nodes, folders, and projects. These entities are primarily used for managing access and configuration

- **Organization node**
	- An organization is the root node in the Google Cloud resource hierarchy and a container for projects and folders. 
	- Organizations let you structure resources hierarchically.
	- Each organization belongs to a single Cloud Identity or Google Workspace account. 
	- The name of the organization is derived from the primary domain name of the corresponding Cloud Identity or Google Workspace account.
- **Folder**
	- Folders are nodes in the Google Cloud resource hierarchy and can contain projects, other folders, or a combination of both. 
	- Folders are similar, but unrelated, to organizational units. Organizational units help you manage users and apply common configuration or policies to users, whereas folders help you manage Google Cloud projects and **apply common configurations, Identity and Access Management (IAM) policies or organizational policies** to projects.
- **Project**
	- A project is a container for resources. 
	- Projects play a crucial role for managing APIs, billing, and managing access to resources.
	- Projects are the containers for **service accounts**.
- **Service account**
	- A service account (or Google Cloud service account) is a special kind of user account that is intended to be used by applications and other types of machine users.
	- Each service account belongs to a Google Cloud project. 
	- Service accounts also use an email address as their identity and the email address always uses a Google-owned domain such as `developer.gserviceaccount.com`.
	- Service accounts don't participate in federation and also **don't have a password**. 
	- On Google Cloud, you use IAM to control the permission that a service account and for outside of Google Cloud, you can use **service account keys** to let an application authenticate by using a service account.
- **Kubernetes service account**
	- Kubernetes service accounts can be used to authenticate when an application calls the Kubernetes API of a Kubernetes cluster, but they cannot be used outside of the cluster.
	- By using Workload Identity, you can link a Kubernetes service account to a Google Cloud service account.