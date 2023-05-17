
What is Service Mesh

- Modern applications are typically architected as **distributed collections of microservices**, with each collection of microservices performing some discrete business function.
- A service mesh, is a way to control **how different parts of an application share data with one another**. 
- A service mesh is a **dedicated infrastructure layer** that we can add to the applications. It allows you to transparently add capabilities like observability, traffic management, and security, without adding them to your own code. 
- The term “service mesh” describes both the type of software you use to implement this pattern, and the security or network domain that is created when you use that software.

## Service Mesh Features

### Traffic Management

- Discover the service endpoints and also control the routing
- Load balancing
- Failure Handling

### Security

- When registering a service with Service Mesh, it creates certificates for each of them. So each component has its own identity.
- Encryption using TLS with the certs generated for both server & client.
- Authentication between components using the certs.
- Authorization to define which components can call other components. This can be extended to users.

### Observability

- Visualization of all the services and communication between them
- Distributed Tracing
- Monitoring of services

 