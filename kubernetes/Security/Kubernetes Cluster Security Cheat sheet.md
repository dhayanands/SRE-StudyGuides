#cheetsheet #authentication #authorization #kubernetes #certificates #security

## Cluster Info

|Command|Description|
|--|--|
|kubectl config get-contexts|to find which cluster context are we in|
|kubectl cluster-info|Get info about the cluster|

## View Kubernetes config

 [Doc Link](<https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/>)

- Admin Config file is present in /etc/kubernetes/admin.conf
- Can also view the config with below commands

|Command|Description|
|--|--|
|kubectl config view|to view all context info|
|kubectl config view --raw|to view context info including the certs|
|kubectl config view --minify|to view only current context info|


# Authentication

## view admin.crt info

Extract the admin cert and decode it

```bash
kubectl config view --raw -o jsonpath='{.users[*].user.client-certificate-data}' | base64 --decode > admin.crt
```

View the cert info

```bash
openssl x509 -in admin.crt -text -noout | head
```

## Token Based Authentication

### Namespace Secret

- When a namespace is created, a secret is created by default
- This namespace secret is mounted to Pods under `/var/run/secrets/kubernetes.io/serviceaccount/` and will be used by Pods to authenticate against API server
- get the pod name using label and jsonpath
```bash
PODNAME=$(kubectl get pods -l app=nginx -o jsonpath='{.items[*].metadata.name}')
```
- login into the pod
```bash
kubectl exec $PODNAME -it -- /bin/bash
```

- List & view the serviceaccount secrets

```bash
# list all the certs
ls -l /var/run/secrets/kubernetes.io/serviceaccount/
# view the CA cert
cat  /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
# view the namespace
cat  /var/run/secrets/kubernetes.io/serviceaccount/namespace
# view the token
cat  /var/run/secrets/kubernetes.io/serviceaccount/token
```

- Export the token and ca cert to use

```bash
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
CACERT=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

- curl the API url

```bash
# curl the API url without token - we get 403 error
curl --cacert $CACERT -X GET https://kubernetes.default.svc/api
# curl the API url with token
curl --cacert $CACERT --header "Authorization: Bearer $TOKEN" -X GET https://kubernetes.default.svc/api
# curl the API url for resource for which we do not have access - we get 403 error
curl --cacert $CACERT --header "Authorization: Bearer $TOKEN" -X GET https://kubernetes.default.svc/api/v1/namespaces/default/pods
```

### Generating Token for ServiceAccount

```bash
# Check is the user is authorized
kubectl auth can-i list pods --as=system:serviceaccount:default:mysvcaccount
# create an RBAC role
kubectl create role sa-role --verb=get,list --resource=pods
# bind the user to the role
kubectl create rolebinding sa-rolebinding --role sa-role --serviceaccount=default:mysvcaccount
# check the access
kubectl auth can-i list pods --as=system:serviceaccount:default:mysvcaccount
```

- From kubernetes v1.24 secrets will not be automatically generated for SA and should be maually created as below

```yaml
---
# service account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: mysvcaccount
  namespace: default
---
# secret for service account
apiVersion: v1
kind: Secret
metadata:
  name: mysvc-secret
  annotations:
    kubernetes.io/service-account.name: "mysvcaccount"
type: kubernetes.io/service-account-token
data:
  # You can include additional key value pairs as you do with Opaque Secrets
  extra: YmFyCg==
---
# pod definition
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: nginx
  name: nginx-pod
  namespace: default
spec:
  serviceAccount: mysvcaccount
  containers:
    - image: nginx
      name: nginx
```

- Test the service account

```bash
# get the pods with service account
kubectl get pods -v6 --as=system:serviceaccount:default:mysvcaccount
# Test the access with the token on the Pod
	## login into the pod
	PODNAME=$(kubectl get pods -l app=nginx -o jsonpath='{.items[*].metadata.name}')
	kubectl exec $PODNAME -it -- /bin/bash
	## Use the service account inside the Pod
	TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
	CACERT=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
	## curl without token -- get 403 error
	curl --cacert $CACERT -X GET https://kubernetes.default.svc/api
	## curl with token header
	curl --cacert $CACERT --header "Authorization: Bearer $TOKEN" -X GET https://kubernetes.default.svc/api/v1/namespaces/default/podss
```

## Certificate Based Authentication

- Kubernets provides API to submit a CSR to create and sign X.509 certificates
- the cert can be used by user & systems for authentication and encryption

**Process**:
Generate private Key -> generate CSR -> Create & Submit *CertificateSigningRequest* object with base64 encoded CSR request to Kubernets API -> approve the CSR -> retrieve the cert

### Create the CSR

```bash
# Create a private key
openssl genrsa -out dhayanand.key 2048
# generate CSR
## CN (Common Name) is the username, O (Organization) is the group
openssl req -new -key dhayanand.key -out dhayanand.csr -subj "/CN=dhayanand"
## *CertificateSigningRequest* needs to be in base64
cat dhayanand.csr | base64
## also need to trim the new lines
cat dhayanand.csr | base64 | tr -d "\n"
## save CSR in a file
cat dhayanand.csr | base64 | tr -d "\n" > dhayanand.base64.csr
```

### Create the CertificateSigningRequest Object

```bash
# Create *CertificateSigningRequest* object
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: dhayanand
spec:
  request: $(cat dhayanand.csr | base64 | tr -d "\n")
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
EOF
```

> **NOTE**
> -   `usages` has to be `client auth`
> -  `expirationSeconds` could be made longer (i.e. `864000` for ten days) or shorter (i.e. `3600` for one hour)
> -  `request` is the base64 encoded value of the CSR file content. You can get the content using this command: `cat myuser.csr | base64 | tr -d "\n"`

### Approve the CertificateSigningRequest Object

```bash
# list the CSR objects
kubectl get csr
# approve the CSR request
kubectl certificate approve dhayanand
```

### Get the certificate

```bash
# get the certificate
kubectl get csr/dhayanand -o yaml
# The certificate value is in Base64-encoded format under `status.certificate`
kubectl get csr/dhayanand -o jsonpath='{.status.certificate}'
# Export the issued certificate from the CertificateSigningRequest
kubectl get csr/dhayanand -o jsonpath='{.status.certificate}' | base64 -d > dhayanand.crt
# Check the certificate
openssl x509 -in dhayanand.crt -noout -text | head -15
```

### Create kubeconfig file

A kubeconfig file defines how to access the cluster, including:
- network location of the API server
- credentials used to authenticate to API server
- Context info

```bash
# get contexts
kubectl config get-contexts
# use a context
kubectl config use-context <context-name>
# to delete kubeconfig entries
kubectl config delete-context <context-name>
kubectl config delete-cluster <cluster-name>
kubectl config unset users.<user-name>
```

- create new kubeconfig
```bash
# define the cluster
kubectl config set-cluster kubernetes --server=https://kubernetes:6443 --certificate-authority=/etc/kubernetes/pki/ca.crt --embed-certs=true --kubeconfig=dhayanand.conf
# define the credential
kubectl config set-credentials dhayanand --client-certificate=dhayanand.crt --client-key=dhayanand.key --embed-certs=true --kubeconfig=dhayanand.conf
# define the context
kubectl config set-context dhayanand@kubernetes --cluster=kubernetes --user=dhayanand --kubeconfig=dhayanand.conf
# view the config
kubectl config view --kubeconfig=dhayanand.conf
# create a clusterrolebinding to give permission to the user to view cluster -- more info on RBAC topic
kubectl create clusterrolebinding dhayanandclusterrolebinding --clusterrole=view --user=dhayanand
# set the context
kubectl config use-context dhayanand@kubernetes --kubeconfig=dhayanand.conf
# check if the user is able to authenticate
kubectl get pods --kubeconfig=dhayanand.conf -v6
# the user will have only view permsision so below will be - 403 forbidden
kubectl scale deployment nginx --replicas=2 --kubeconfig=dhayanand.conf -v6
# to set the user config
export KUBECONFIG=dhayanand.conf
# to revert to default file
unset KUBECONFIG
```


# Authorization

### Using user impersonation to test authorization

```bash
# test if the current user can
kubectl auth can-i list pods
# impersonation as service account
kubectl auth can-i list pods --as=system:serviceaccount:default:mysvcaccount
# using verbose with impersonation
kubectl get pods -v6 --as=system:serviceaccount:default:mysvcaccount
```

## Create Role and RoleBinding

With the certificate created it is time to define the Role and RoleBinding for this user to access Kubernetes cluster resources.

```bash
# Create role
kubectl create role <role_name> --verb=create --verb=get,list --resource=pods
# full access to role
kubectl create role <role_name> --verb=create --verb=* --resource=pods
# get role
kubectl get role <role_name>
# create rolebinding for user
kubectl create rolebinding <rolebinding_name> --role=<role_name> --user=<user> --namespace=<namespace>
## clusterrole
kubectl create rolebinding <rolebinding_name> --clusterrole=<role_name> --user=<user> --namespace=<namespace>
# create rolebinding for serviceaccount
## role
kubectl create rolebinding <rolebinding_name> --role=<role_name> --serviceaccount=<namespace>:<service-account> --namespace=<namespace>
## clusterrole
kubectl create rolebinding <rolebinding_name> --clusterrole=<role_name> --serviceaccount=<namespace>:<service-account> --namespace=<namespace>
```

## Create ClusterRole and ClusterRoleBinding

With the certificate created it is time to define the Role and RoleBinding for this user to access Kubernetes cluster resources.

```bash
# Create clusterrole
kubectl create clusterrole <role_name> --verb=create,get,list,update,delete --resource=pods
# full access to clusterrole
kubectl create clusterrole <role_name> --verb=create --verb=* --resource=pods
# get clusterrole
kubectl get clusterrole <role_name>
# create clusterrolebinding for user
kubectl create clusterrolebinding <rolebinding_name> --clusterrole=<role_name> --user=<user> --namespace=<namespace>
# create clusterrolebinding for serviceaccount
kubectl create clusterrolebinding <rolebinding_name> --clusterrole=<role_name> --serviceaccount=<namespace>:<service-account> --namespace=<namespace>
```

