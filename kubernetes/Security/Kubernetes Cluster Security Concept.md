
# Securing an API server

**Flow of API request:**

Authentication -> Authorization -> Admission control

## Authentication plugins

### **Client certificates**
  - Most commonly used
  - default when using kubeadm
  - Common Name (CN) is the username
  - CON : hard task to manage , renew / reissue certs

### **Authentication token**
  - way to encode authenticating infomation in the HTTP Authorization Header of an HTTP client request
  - used in Service Accounts
  - used as bootstrap tokens while building cluster and adding nodes
  - used used for user authntication with static token file
  - CON : static token file is read only during API server startup so any chages to file will need restart of API server

### **Basic HTTP**
  - user info stored in static password file
  - only read during API server startup
  - CON : static token file is read only during API server startup so any chages to file will need restart of API server

### **OpenID connect**
  - enables external identity providers for authentication and remote authentication services
  - Authentication token and Basic HTTP methods are easy to setup so can be used in DEV environments


## Authorization plugins  

### **Role-based Access Control (RBAC)**
  - Control access to resources based on the roles of individual users and the functions these roles can perform
  - We define a role, which can act of resources in the cluster and then bind users to that role

### **Node**
  - Used to grant API access and permissions to kubelets on nodes
  - it helps in restricting the access to the resources that the kubelets need to have access to

### **Attribute-based Access Control (ABAC)**
  - Rights are granted to users through use of policies combined with attributes
  - Enables granular access controls based on users, resources and environment

> **NOTE**
> In kubeadm based clusters, the default authorization modules enabled are RBAC & node

# Authentication 

## Users in Kubernetes  

- Users are managed by external systems
- No User API objects in Kubernetes
- Authentication plugins implement authentication and is pluggable
- Usernames are used for access control and logging
- Users can be aggregated into groups

## Service Accounts

- Enable services in Pods / Pods to authenticate to API server
- Service account is a Namespaced API object that consists of info including service account name & credential, which is token used for authentication
- Each namespace created will have a service account created automatically
- All Pods running in cluster must have a ServiceAccount defined, so when there is no SA defined in the Pod Spec then the default SA in namespace will be used by the Pod to authenticate and authorize
- Can create a ServiceAccount object instead of using the one created by default using namespace creation

### Service Account Credentials

- Each SA is tied to a credential that is stored as a **Secret** in the cluster
- The sercret is used to interact with the API server
- The SA secret consists of:
  - the CA certificate - to trust the cert that the API server is using for its HTTPS endpoint
  - authentication token - for token based auth to authenticate to the API server
  - namespace of the service account - to scope the resources that we are trying to access
  - Image pull secret - to access a container registry that requires authentication
- By default, SA secrets are mounted inside a Pod as files using a Volume under `/var/run/secrets/kubernetes.io/serviceaccount`

## Certificates and PKI in Kubernetes

- Kubernetes uses certificates to provide TLS encryption
- API server is exposed on HTTPS endpoint using a certificate
- In addition to encryption, certificates are also used for authenticating both user and system compoenets
- Kubeadm creates a self-signed Certificate Authority for the cluster and the key (ca.key) & cert (ca.crt) for CA are present under `/etc/kubernets/pki`
- Kubeadm also generates certificates for system components
	- Schedular uses a kubeconfig file for suthentication, which is `/etc/kubernetes/scheduler.conf`
	- Kube-proxy specifies the cert path in configmap and can be viewed with command `kubectl get configmaps -n kube-system kube-proxy -o yaml | grep -A20`
- Kubeadm creates Kubernetes-admin user and adds it to cluster-admin role. This user uses cert to authenticate so the user cert is updated in `admin.conf` file
- Can use external / trusted CA - [Doc link](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/)

### kubeconfig file

- Kubeconfig can have one or more contexts
- The context is:
	- the cluster we want to connect to
	- the credential to authenticate to that cluster's API server
- With multiple contexts defined, we can switch between the contexts
- We can also have multiple kubeconfig files and switch between them if needed
- kubeconfig files are used by both, users and system conponents to authenticate to the API server

**Bisecting kubeconfig**
- kubeconfig file components:
	- *Users* contains:
		- username
		- credentials : cert / token / password
	- *Clusters* contains:
		- network location of api server
		- CA cert
	- *Contexts* contains access parameter to cluster:
		- user is mapped to cluster
		- namespace

`admin.conf` content
```yaml
## /etc/kubernetes/admin.conf
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: <ca.crt>
    server: https://kubernetes:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: <admin.crt>
    client-key-data: <admin.key>
```

  Here, 
- **apiVersion**
  - Where to assign the data
- **cluster**
  - ***cluster***:
    - Provides the cluster name, API URL and the cert (when using TLS) or metion cluster uses username & password (insecure-skip-tls-verify) to authenticate curl requests
  - ***context***
    - Define context and maps cluster, namespaces & credentials (users) to the context
  - ***current-context***
    - The context that is used by *kubectl* command
    - This can also be changed by explicitly passing context per command basis
- **kind**
  - kind of object. In this case its Config
- **preferences**
  - Optional setting for kubectl command (like colorizing output)
- **users**
  - Nickname to cert / username:password / token that is used for authntication

Below is a example config from kubernetes.io
```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: fake-ca-file
    server: https://1.2.3.4
  name: development
- cluster:
    insecure-skip-tls-verify: true
    server: https://5.6.7.8
  name: scratch
contexts:
- context:
    cluster: development
    namespace: frontend
    user: developer
  name: dev-frontend
- context:
    cluster: development
    namespace: storage
    user: developer
  name: dev-storage
- context:
    cluster: scratch
    namespace: default
    user: experimenter
  name: exp-scratch
current-context: ""
kind: Config
preferences: {}
users:
- name: developer
  user:
    client-certificate: fake-cert-file
    client-key: fake-key-file
- name: experimenter
  user:
    password: some-password
    username: exp
```

# Authorization

## RBAC

### Concept

- Follows RESTful API semantics : perform VERB on a NOUN
- There is default *DENY* in place so rules are written to permit actions on resources
	- there is no *deny* permission as default is *deny*

**API objects to implement RBAC**

- Role 
	- Roles are **namespaced**
	- Defines what can be done on resources within a namespace in the cluster
- ClusterRole
	- non-namespaced
	- ClusterRoles are **cluster-scoped**
	- Defines what can be done on resources in the cluster
	- Gives access accross more than one namespace or all namespaces
- RoleBinding
	- Defines the Subject and refers to a namespaced Role
	- Can be used with ClusterRole to bind Subject to serveral or all namespaces
- ClusterRoleBinding
	- non-namespaced
	- Defines the Subject and refers to a ClusterRole

**When to use what**

- Use Role & RoleBinding to scope security to a single namespace
- Use ClusterRole & RoleBinding to scope security to several or all namespaces
- Use ClusterRole & ClusterRoleBinding to scope security to all or cluster-scoped resources

### Defining Roles & Bindings

- In a **Role** definition, rules are lists that consists of:
	- apiGroup - empty string designates the core API group
	- Resources - Pods, Deployments, Services etc.,
	- Verbs - get, list, create, delete, patch, watch, update, deletecollection 
- Roles are bound to subjects (users, groups or service accounts) using RoleBindings
- **RoleBindings** consist of:
	- roleRef 
		- when defining RoleBinding -> Role / ClusterRole 
		- when defining ClusterRoleBinding -> ClusterRole
	- Subjects 
		- kind - User/ Group / ServiceAccount
		- Name - name of the subject
		- Namespace

**Role definition**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: sa-role
  namespace: default
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list
```

**RoleBind definition**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sa-rolebinding
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: sa-role
subjects:
- kind: ServiceAccount
  name: mysvcaccount
  namespace: default
```

**ClusterRole definition**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list
  - watch
```

**ClusterRoleBinding definition**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dhayanandclusterrolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: view
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: dhayanand
```

#### Default ClusterRoles

- Kubernets provides some default ClusterRoles for us to use:

**cluster-admin**
- Provides cluster-wide super-user access to all resources
- With RoleBinding, provides full admin access to the namespace
	- can edit Roles, RoleBindings and Resource Quotas

**admin**
- Provides full access to resources in a namespace
- With RoleBinding, provides full admin access to resources in namespace
	- can edit Roles, RoleBindings

**edit**
- Provides read/write access to resources within a namespace
- With RoleBinding, provides access to manage resources in namespace
	- cannot view or edit Roles, RoleBindings and Resource Quotas
	- but can access secrets in the namespace

**view**
- Provides read-only access to resources within a namespace
- Cannot view or edit Roles, RoleBindings and Resource Quotas
- Also cannot access secrets in the namespace

