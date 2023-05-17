# Investigating Kubernetes Networking

## A CNI Network

- Below is from a CNI network implemented using Calico Plugin

```bash
# get all nodes and IP information, INTERNAL-IP is the real IP if the node
kubectl get nodes -o wide
# get node info to see there is no IPs in Annotations, InternalIP under Addresses and PodCIDR (if only of IPv4/IPv6 is used) and PodCIDRs(if both IPv4 & IPv6 are used)
# Pods in the nodes get the IP adddress from the PodCIDR when kubenet is used
kubectl describe node <node_name> | less
# get all pod and IP information, IP is the IP if the Pod
kubectl get pods -o wide
# get pod info
kubectl describe pod <pod_name> | less
# check the routing table to see how the Pods communicate within nodes
## net-tools package needs to be installed to work with `route` command
route
### OUTPUT SNIPPET ###
## can see that the interface to communicate between the Pods is the tunnel (tunl0)
## the destination ip of the target node is the IPv4IPIPTunnelAddr
dhayanand@master1:~$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         _gateway        0.0.0.0         UG    0      0        0 eth0
10.10.10.0      0.0.0.0         255.255.255.0   U     0      0        0 eth0
10.20.137.64    0.0.0.0         255.255.255.192 U     0      0        0 *
10.20.137.95    0.0.0.0         255.255.255.255 UH    0      0        0 calia75060ec7d6
10.20.137.96    0.0.0.0         255.255.255.255 UH    0      0        0 cali88fcec22536
10.20.137.97    0.0.0.0         255.255.255.255 UH    0      0        0 calidd539c56ba3
10.20.189.64    worker2         255.255.255.192 UG    0      0        0 tunl0
10.20.235.128   worker1         255.255.255.192 UG    0      0        0 tunl0
dhayanand@master1:~$ 
```

## A kubenet Network

```bash
# get all nodes and IP information, INTERNAL-IP is the real IP if the node
kubectl get nodes -o wide
## snippet from GKE
dhayanand_sahadevan@cloudshell:~ (kubernetes)$ kubectl get nodes -o wide
NAME	STATUS   ROLES    AGE   VERSION            INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-1   Ready    <none>   10d   v1.23.8-gke.1900   10.0.4.30     <none>        Container-Optimized OS from Google   5.10.127+       containerd://1.5.13
gke-2   Ready    <none>   10d   v1.23.8-gke.1900   10.0.4.45     <none>        Container-Optimized OS from Google   5.10.127+       containerd://1.5.13
gke-3   Ready    <none>   10d   v1.23.8-gke.1900   10.0.4.27     <none>        Container-Optimized OS from Google   5.10.127+       containerd://1.5.13
gke-4   Ready    <none>   10d   v1.23.8-gke.1900   10.0.4.31     <none>        Container-Optimized OS from Google   5.10.127+       containerd://1.5.13
dhayanand_sahadevan@cloudshell:~ (kubernetes)$
# get node info to see the IPv4IPIPTunnelAddr under Annotations, InternalIP under Addresses and PodCIDR (for single IPv4 / IPV6) and PodCIDRs(if both IPv4 & IPv6 are used)
# CNI does not assign the IP address to the Pod from the PodCIDR
kubectl describe node <node_name> | less
# get all pod and IP information, IP is the IP if the Pod
kubectl get pods -o wide
# get pod info
kubectl describe pod <pod_name> | less
# check the routing table to see how the Pods communicate within nodes
## net-tools package needs to be installed to work with `route` command
route
### OUTPUT SNIPPET ###
## can see the node interface (10.0.4.1) and the routes to the Pods via Bridge (vethxxxxxx)
## the route between the pod networks are defined on the vNet on cloud
== MORE INFO on the pod network routing in GKE need to be explored yet==
dhayanand_sahadevan@gke ~ $ sudo route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         10.0.4.1        0.0.0.0         UG    1024   0        0 eth0
10.0.4.1        0.0.0.0         255.255.255.255 UH    1024   0        0 eth0
10.4.0.2        0.0.0.0         255.255.255.255 UH    0      0        0 veth88df2b85
10.4.0.3        0.0.0.0         255.255.255.255 UH    0      0        0 vethb55d49ae
10.4.0.5        0.0.0.0         255.255.255.255 UH    0      0        0 veth722e6345
10.4.0.6        0.0.0.0         255.255.255.255 UH    0      0        0 vethdd6d741f
10.4.0.9        0.0.0.0         255.255.255.255 UH    0      0        0 vethb93c281e
10.4.0.134      0.0.0.0         255.255.255.255 UH    0      0        0 vethb14988cd
169.254.123.0   0.0.0.0         255.255.255.0   U     0      0        0 docker0
dhayanand_sahadevan@gke ~ $
```


FROM AKS

![[Pasted image 20221106174534.png]]

# Cluster DNS

## DNS configuration of cluster

```bash
# cluster DNS is running as a clusterIP service with multiple ports
kubectl get service -n kube-system
# investigate the codedns deployment -- 2 replicas,  Args with location of Corefile whicg us ConfigMap mounted as volume
kubectl describe deployments coredns -n kube-system | less
# check the ConfigMap defining codeDNS configuration - default forwarder is /etc/resolv.conf-
kubectl get configmaps coredns -n kube-system -o yaml
# check coreDNS logs
kubectl logs -n kube-system --selector 'k8s-app=kube-dns' --follow
```

## Change the configuration to a specific provider

we need to update the code as    `forward . 1.1.1.1` 
  
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        # below code forwards to the DNS server in /etc/resolv.conf
        # to change forwarder, we can use `forward . 1.1.1.1`
        forward . /etc/resolv.conf {
           max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }

```

## Change the dnsConfig of the Pod

set `dnsPolicy` to `none`

```yaml
...
	spec:
	  automountServiceAccountToken: true
	  containers:
	  - image: nginx
		imagePullPolicy: Always
		name: nginx
		ports:
		- containerPort: 80
		  protocol: TCP
	  dnsPolicy: none
	  dnsConfig:
		nameservers:
		  - 1.1.1.1
		searches:
		  - app1.ns1.svc.cluster.local
...
```

# Services

## ClusterIP

```bash
# create deployment with replica
kubectl create deployment nginx --image=nginx --replicas=2
# expose the deployment, if no service is provided, the default will be ClusterIP
kubectl expose deployment nginx --port=80 --target-port=80 --type=ClusterIP
# check if the service is created
kubectl get services
# get the ClusterIP and curl
SERVICEIP=$(kubectl get services nginx -o jsonpath='{ .spec.clusterIP }')
echo $SERVICEIP
curl http://$SERVICEIP
# check the endpoint for the service wich points to the clusterIP
kubectl get endpoints
kubectl get pods -o wide
# check the pod labels used by the serice 
kubectl get pods --show-labels
```

## NodePort

```bash
# create deployment with replica
kubectl create deployment nginx --image=nginx --replicas=2
# expose the deployment, if no service is provided, the default will be ClusterIP
kubectl expose deployment nginx --port=80 --target-port=80 --type=NodePort
# check if the service is created
kubectl get services
# get the ClusterIP, port & NodePort and curl the url
CLUSTERIP=$(kubectl get services nginx -o jsonpath='{ .spec.clusterIP }')
PORT=$(kubectl get services nginx -o jsonpath='{ .spec.ports[].port }')
NODEPORT=$(kubectl get services nginx -o jsonpath='{ .spec.ports[].nodePort }')
echo $NODEPORT
curl http://<node-ip>:$NODEPORT
curl http://$CLUSTERIP:$PORT
# check the endpoint for the service wich points to the clusterIP
kubectl get endpoints
kubectl get pods -o wide
# check the pod labels used by the serice 
kubectl get pods --show-labels
```


## LoadBalancer

```bash
# create deployment with replica
kubectl create deployment nginx --image=nginx --replicas=2
# expose the deployment, if no service is provided, the default will be ClusterIP
kubectl expose deployment nginx --port=80 --target-port=80 --type=LoadBalancer
# check if the service is created
kubectl get services
# get the ClusterIP, port & NodePort and curl the url
LOADBALANCERIP=$(kubectl get services nginx -o jsonpath='{ .status.loadbalancer.ingress[].ip }')
PORT=$(kubectl get services nginx -o jsonpath='{ .spec.ports[].port }')
echo $LOADBALANCERIP
curl http://$LOADBALANCERIP:$PORT
# check the endpoint for the service wich points to the clusterIP
kubectl get endpoints
kubectl get pods -o wide
# check the pod labels used by the serice 
kubectl get pods --show-labels
```

# Service Discovery

## Envionment variables

```bash
# get the pod name
PODNAME=$(kubectl get pods -o jsonpath='{ .items[].metadata.name }')
# check the ENVIRONMENT variables for the defined services
kubectl exec -it $PODNAME -- env | sort
```