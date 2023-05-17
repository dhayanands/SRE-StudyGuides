# Google Cloud

- [Google Cloud](#google-cloud)
  - [Placement of resources](#placement-of-resources)
  - [Resource Hierarchy](#resource-hierarchy)
    - [Project](#project)
    - [Folder](#folder)
    - [Organization Node](#organization-node)
  - [Networking](#networking)
    - [Virtual Private Cloud](#virtual-private-cloud)
    - [Loadbalancing](#loadbalancing)

## Placement of resources

Location -> Region -> Zone

## Resource Hierarchy

Organization Node -> Folder -> Project -> Resources

### Project

- Basis for enabling and using all Google Services
- Can Manage APIs, enable billing, add / remove collaborators and enable Google Services
- Each project is a seperate entity under the Organization node
- Each resource belong to exactly one project
- Projects can have different owners and users
- Projects are billed and managed seperately
- Each project has 3 identifying attributes
- Project ID - gobally unique ID assigned by Google, immutable after creation - to inform Google Cloud which projecvt to work with
- Project Name - user created, can be changed any time - used by users to manage
- Project Number - globally unique, assigned by google, immutable - used internally by Google Cloud to keep track of resources

**Resource Manager Tool** is API used to manage the projects.

- Gather list of projects
- Create new projects
- Update existing projects
- Delete projects
- Recover previously deleted projects
- Access through RPC API & REST API

### Folder

- Assign policies to resources
- Can contain Projects or other folders
- Used to organize projects under Organization in a hierarchy
- also can deligate administravite rights to folders with multiple projects

### Organization Node

- Organization policy Administrator role
- Project creator role

## Networking

### Virtual Private Cloud

- Created witin a Folder
- secure, individual, private cloud computing model hosted within a public cloud
- Connects resources to eachother and internet
- Google VPC networks are global and can have subnets in any region worldwide
- so resources can be in diffrent region within same subnet

**Routing & Firewall:**

- Routing tables are built-in for a VPC
- VPCs provide Global Distributed firewall
- firewall rules can also be defined using metadata tags

**VPC Peering** can be used to connect 2 different VPCs

### Loadbalancing

- **Cloud Loadbalancing** provides single as well as cross-region loadbalacing including multi-region failover
- **Global HTTP(S)** loadbalacer for web application loadbalacing
- **Global SSL Proxy** loadbalacer for SSL traffic that is TCP but not HTTP
- **Global TCP Proxy** loadbalacer for other TCP traffics that does not use SSL
- **Regional** loadbalacer for UDP traffic or traffic on any port no
- **Regional Internal** loadbalacer for all internal traffic between applications


Compute
Storage
Big Data
Machine Learning
Application Servies - web, mobile, analytics & backend