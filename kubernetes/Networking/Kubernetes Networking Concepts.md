# Kubernetes Networking Model

## Why use Networking Model

- Maintain simplicity
- Hide implementation details
- Administrator Controlled
- Define & consume networking resources as code in  manifests
- Provide simpler service discovery and app configuration

### Networking Model Rules

- All Pods can communicate with eachother in all Node
- Agents (kubelets, kube-proxy etc.,) on a node can communicate with all Pods on that Node
- No Network Address Transalation (NAT)

## Network Types

- **Node Network** - where the nodes are connected to, part of
- **Pod Network** - where Pods are connected to , IPs are assigned based on Pod CIDR range given druing cluster setup
- **Cluster Network** - used by services using `clusterIP` service type, IPs are assigned based on range defined as `service cluster IP` in API server & controller manager configurations


## Pod Networking and Communication

### Inside the Pod 

- The containers inside pod communicate over `localhost`

### Pod to Pod within a Node

- Pods will use the Pod IP of the interface attached to the Pod
- The interface of the Pod is attched to a local software bridge or tunnel interface

### Pod to Pod on another Node

- Pods will use the Real IP (node IP) and the network between the nodes must be able to facilitate this communication
- Can be done using a Layer2 or Layer3 connectivity or an overlay network
- **Overlay network** gives the appearance of a single network between the nodes in a cluster, and used tunnel or other mecahnism to move encapsulated Pod network packets between the nodes

### Access To Services

- Services are implemented in `kube-proxy`
- Exposes services to both users internal & external to cluster depending on service type

## Pod Networking Internals

- Containers inside a Pod:
	- share a single network namespace
	- commnunicate over `localhost`
	- share the same IP address and source port ranges for applications
- The network namespace in a Pod is implemented by a special conrainer called the `pause` or thr infrastructure conatiner
- When a Pod is created and started, the `pause` container starts first and setus up the network namespace for the Pod and then the application containers start inside the Pod and share this network namespace
- This `pause` container enables application containers to be restarted without interruping the network namespace inside of the Pod. ie., even if the application container restarts, the network will persist
- The `pause` container has lifecyscle of the Pod

## Container Network Interface (CNI)

- The implementation of the kubernetes network model is abstracted out of kubernetes and CNI is used to implement container and Pod networking in a cluster - Kuberbnetes -> CNI -> Container Runtime & OS
- CNI's Responsibilities include setting up namespaces, interfaces, bridge or tunnel configurations and IP addresses
- CNI defines standard specification for managing container networking
- **CNI Plugins** like Calico, Fannel etc., are used to implement Kubernetes Networking Model
- On nodes, kubelets controls the local network configuration and its configuration defines a network plugin, either CNI or Kubenet
	- if CNI : kubelet will load the CNI plugin and its configuration, usually a tunnel is setup on the node to implement Pod network
	- id Kubenet : node will be configured with routes and bridges inside the BaseOS to implement Pod network

# Cluster DNS

- Kubernets provides DNS as a cluster service
- DNS service is the core to Service Discovery inside the cluster
- DNS service used is [**coreDNS**](https://coredns.io/manual/toc/) and the service run in 2 pods to provide HA
- By default, all Pods created are configured to use this DNS service
- DNS records are created for all the resources deployed in the cluster
- Services will get A/AAAA records depending on IPv4 or IPv6
- Namespaces will get DNS subdomains created
- We can customize both the DNS service and Pods configuration
	- common one is configuring specific DNS forwarder for DNS zones that aren't hosted in the cluster
	- also configure DNS client configuration in the Pods like overide cluster DNS or provide a custom DNS suffix search list

## Configuring a DNS Forwarder in Cluster

- Configuration for coreDNS is stored as a ConfigMap named `coredns` in the `kube-system` namespace
- This ConfigMap is mapped as a volume into the Pod's filesystem under `/etc/coredns` with file name `Corefile`
- By default, the cluster DNS service will forward to DNS servers specified at `/etc/resolv.conf`
	- In below example, we see `forward . /etc/resolv.conf` , where `.` represents all DNS zones
- To change the configuration to a specific provider, we need to update the code as    `forward . 1.1.1.1` 

## Configuring DNS client inside Pod

- To configure DNS clinet inside a Pod, we have two Pod specs:
	- **dnsPolicy** - defines how a Pod's DNS client acts. by default, it is set to `ClusterFirst`. available dnsPolicies are:
		- **default** - The Pod inherits the name resolution configuration from the node that the Pods run on
		- **ClusterFirst** - Any DNS query that matchs the configured cluster domain suffix, such as "cluster.local", is forwarded to the cluster DNS service and all other DNS queries are  forwarded to the upstream nameserver inherited from the node. 
		- **ClusterFirstWithHostNet** - For Pods running with hostNetwork, you should explicitly set its DNS policy "`ClusterFirstWithHostNet`". ClusterFirstWithHostNet is not supported for Pods that run on Windows nodes.
		- **None** -  It allows a Pod to ignore DNS settings from the Kubernetes environment. All DNS settings are supposed to be provided using the `dnsConfig` field in the Pod Spec.
	- dnsConfig
		- Can be used when dnsPolicy is set to `none`
		- used to define `nameservers` and `searches`

# Services

- Service is networking abstarction providing persistent virtual IP and DNS
- Services provide persistent enpoint access for clients and loadbalancing to the backend pods
- as Pods come and go due to policies and controller operations, services are automatically updated to reflect those changes and the loadbalancing configuration is updated during Pod controller operations
- This adds persistency to the ephemerality of Pods - pods can change but the service will not be interrupted

## How Service work

- Services use **labels** and **selectors** to determine which Pods are a member of which service in a cluster
- A controller will monitor which pods satisfy the selector, and for each pod that matches the selector, an **endpoint** object is created consisting of the pod IP and the target application port
- Each matching endpoint is added to a list of endpoints supporting the service and then the list is used to load balance the traffic across all the pods supporting that service
- The implementation of the service is the reponsibility of the `kube-proxy`
- The kube-proxy exposes the services onto the network
- The default way this is implemented is with the IP tabels. other options include IPVS and user space proxy mode
- kube-proxy watches the API server and the endpoint objects avaialbe and updates the local configuration to match the desired state

## Service Types

### ClusterIP

- ClusterIP is only available inside of the cluster and is the default service type if we dont specify a service type in the service configuration
- ClusterIP gets IP address from cluster network specified in the service cluster IP range parameter on both the API server's & controller manager's configuration
- kube-proxy will configure the needed iptables rules on all the nodes in the cluster defining service's cluster IP address and port, and also implementing the load balancing rule sto the back-end Pod endpoints
- ClusterIP address is completely virtual and exists only inside iptables in the nodes
- ClusterIP is the foundation for rest of the service types on which they work
- Common use case for ClusterIP is for service that dont need to be accessed outside of the cluster or if we provide access via other means like Ingress

### NodePort

- When using NodePort, kube-proxy will expose the service on the real IP of the node on each node in the cluster on a dynamic port
- A NodePort port is assigned to the NodePort service and by default, the port range is `30000 to 32767`
- The port can be dynamically allocated by the cluster or statically defined into the sercice configuration
- When creating a NodePort service, we will also get a clusterIP service and port allocated
	- External Request on NodeIP:NodePort -> ClusterIP -> Pod endpoint
- NodePort is commonly used when intergrating with external loadbalacer and cloud scenarios
- Also used for testing / debugging application in development scenarios

### LoadBalancer

- Used to provision a Cloud Load Balancer on a Cloud provider
- Cloud Load Balancer will get a public IP address from cloud provider
- When creating a Cloud Load Balancer, we also get a NodePort and ClusterIP service
	- External Request -> Cloud Load Balancer -> NodePort -> ClusterIP -> Pod endpoint

## Other service types

- **ExternalName**
	- provides benefits of service discovery for services external to the cluster
	- In the definition of ExternalName, we provide a DNS name for the service outside the cluster and kubernetes will create `CNAME` record for thet service pointing to the external service DNS name
	- This way, internal applications can use service discovery like DNS & environment variables to find the external service
- **Headless service**
	- Headless service is a service with no clusterIP
	- When defining Headless service with selectors, we get DNS records defined for each endpoint IP that matches the selector
	- When we query the Headless service, we get all the pod IPs in the result set and the client can decide on which IPs it will use
	- Commonly used in database systems and also in stateful sets
- **Service without selectors**
	- Service without selectors enables us to map services to specific endpoints
	- Since there are no selectors defined, we will nedd to manually create the endpoint objects
	- The endpoint objects can point to any IP and the IP can be inside or outside the cluster
	- Used when we want to manually control where to send traffic to but still have the benifits of service discovery

## Service Discovery

- Infrastructure independence comes from avoiding configurations pointing to static configurations like IP address or pod names and services helped in implementing this
- Service discovery provides way to access service information dynamically inside the cluster
- Two ways to discover services running in cluster:
	- DNS
		- For Services
			- each service created will get a DNS record
			- normal services get `A/AAAA` records
			- cluster DNS is in format `<svcname>.<ns>.svc.<clusterdomain>`
			- eg: `nginx.default.svc.cluster.local`
		- For Namespaces
			- each namespace created will get a DNS subdomain in format `<ns>.svc.<clusterdomain>`
			- eg: `ns1.svc.cluster.local`
	- Environment variables
		- For each service available at the time a pod is created, there is a collection of environment variables defined giving information about the services that are up and running
		- These are defined in Pods for each Service availabe at the Pod start up
		- Environment variables include  IP address, port, name of service etc.,

# Ingress

## Components

**Ingress resource**

- Defines how HTTP service in a cluster can be accessed externally
- Defining the rules for how requests are passed into the cluster to access pod endpoints eunning web applications

**Ingress Controller**

- Ingress Controller is the actual implementation or the software that implements the ingress resource's rules
- Its the ingress controller that actually provides access to the cluster-based services
- This is generally an HTTP reverse proxy
- It receives the connection, finds a matching rule, and passes the traffic directly to the services pod and points to respond to the request

**Ingress Class**

- In a cluster we can have more than one ingress controller, and the ingress class gives the ability to associate an ingress reource with a specifci ingress controller in a cluster
- It also defines a default Ingress class by defining the `isDefault` class annotation on the Ingress class

## Overall Flow

Deploy Ingress controller -> Create ingress class for the ingress controller -> define Ingress resources refering to Ingress class to use and define the routing rules for the traffic

## Additional Funtionalities

- In addition to providing external access, the Ingress will provide **load balancing directly to the endpoints** in the cluster, bypassing the clusterIP service
- Ingress gives ability to do **name-based virtual hosting**, enabling us to provide access to many different services inside of cluster, based on the host header in the HTTP request
	- Ingress will read the host header and send the request to the correct backend cluster service
	- This enables us to use a sinle public ip and seperate traffic based on the contents of host headers
- Additionally Ingress can provide **path-based routing**, enabling us to direct request based on the path in the URL to unique services in cluster 
- Ingress also gives ability to implement **TLS termination**, so in our Ingress resource we can define a certificate configuration and use that to provide encryption from client browser to the Ingress controller

## Ingress Controller

 - Ingress controller is the one that implements the Ingress rules
 - Many types:
	 - can run as Pod in cluster - nginx
	 - can be a hardware external ro cluster - Citrix, F5 etc.,
	 - cloud Ingress controllers - AppGW, Google LB and AWS ALB Ingress
- Ingress controllers have defined spec
- Why Ingress and not an LB?
	- Ingress funtions as Layer 7 - can do : path based routing & Name-based virtual host
	- can implement : URL rerouting, session persistency and dynamic waiting
	- Ingress controller is single resource that can provide access to multiple internal services, where LB expose single service on single IP and port
	- redecued latency as traffic is sent directly to Pod endpoints

## Example definitions

#### Exposing single service

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
spec:
  ingressClassName: nginx
  # all traffic will go to the defaultBackend, which is a service here
  defaultBackend:
    service:
      name: hello-world
      port:
        number: 80
```

#### Exposing multiple services

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
spec:
  ingressClassName: nginx
  rules:
  # define host for ingress to listen on
  - host: "path.example.com"
    http:
      paths:
      - path: "/red"
        pathType: Prefix
        backend:
          service:
            name: service-red
            port:
              number: 8080
      - pathType: exact
        path: "/blue"
        backend:
          service:
            name: service-blue
            port:
              number: 8081
  # all other traffics with no rules defined will go to the defaultBackend
  defaultBackend:
     service:
      name: hello-world
      port:
        number: 80
```

pathType:
- prefix : will perform partial prefix match based on the path
- exact: request coming in must be an exact case-sensitive match
- implementation specific: pushes the configuration responsibility from the ingress into IngressClass resource 

#### Name Based Virtual Host

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: "red.example.com"
    http:
      paths:
      - path: "/"
        pathType: Prefix
        backend:
          service:
            name: service-red
            port:
              number: 8080
  - host: "blue.example.com"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: service-blue
            port:
              number: 8081
```

#### Using TLS certificates

- Need to store certificate as a secret in the cluster

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: testsecret-tls
  namespace: default
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
type: kubernetes.io/tls
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
spec:
  ingressClassName: nginx
  tls:
  - hosts:
      - tls.example.com
    secretName: secret-tls
  rules:
  - host: "tls.example.com"
    http:
      paths:
      - path: "/"
        pathType: Prefix
        backend:
          service:
            name: hello-world
            port:
              number: 80
```