OpenShift Networking resources:
- Services: provides load balancing to replicated Pods in an application, and provides also access to applications. It connects to an Endpoint, which are Pod individual IP addresses
- Ingress: Kubernetes resource that exposes services to external users. Not used in OpenShift
- Routes: alternative to Ingress

![[Attachments/network-components-in-openshift.png]]

OpenShift SDN:
- Connectivity in OpenShift is via SDN
- Control and data plane
- SDN solves five requirements
	- Manage network traffic and resources as software
	- Communication between containers within the same project
	- Communication between pods within and beyond project
	- Communication from Pod to a Service
	- Communication from external network to Service
- Network Operator is responsible for managing networks

DNS Operator:
- Implements CoreDNS DNS server
- Internal CoreDNS server is used by Pods for DNS resolution
- To view its config `oc describe dns.operator/default`
- Operator's role:
	- Create a default cluster DNS name `cluster.local`
	- Assign DNS names to namespaces
	- Assign DNS names to services
	- Assign DNS names to pods

DNS names for services: `service_name.project_name.cluster-dns-name`:
- `db.project.cluster.local`

The CoreDNS implements also SRV record besides A record, in which port name and protocol are prepended to the service A record:
- `_443._tcp.webserver.project.cluster.local`

If service has no IP address, DNS records are created for the IP addresses of the pods, in a roundrobin fashion.

Cluster Network Operator defines how the network is shaped and provides information about the following:
- Network address
- Network modes
- Network provider
- IP address pools

For details about this operator: `oc describe network/cluster -o yaml`

Network Policy:
- Allows to define Ingress and Egress filtering
- If no policy exists, all traffic is allowed
- If a policy exists, it will block all traffic with the exception of allowed Ingress and Egress traffic

---

Service is used as a load balancer that provides access to a group of Pods that is addressed by using a label as the selector. Also it is needed for scalability.

When using `oc new-app`, a service resource is created to expose access to the application.

Service types:
- ClusterIP: default, exposes IP internally to the cluster, for microservices use cases
- NodePort: exposes a port on the node IP address
- LoadBalancer: exposes the service through a cloud provider load balancer.
	- The cloud provider LB talks to the OpenShift network controller to automatically create a node port to route incoming requests
- ExternalName: creates a CNAME in DNS to match an external host name

Ingress resource (Kubernetes):
- Its managed by Ingress operator and accepts external requests that will be proxied
- Route resource is an Ingress with more features
	- TLS re-encryption
	- TLS pass-through
	- split traffic for blue-green deployments

OpenShift Route:
- Implemented by the shared router service that runs as a Pod in the cluster
- Router Pods bind to the public-facing IP addresses on the nodes
- DNS wildcard is required for this to work
- Routes can be secure or insecure

Creating a route:
- Name of the service
- Host name for the route, related to the cluster DNS wildcard domain
- (Optional) path for path-based routes
- Target port, which the app listens
- Encryption strategy
- (Optional) labels to be used as selectors

Route options and types:
- Secure routes can use different types of TLS termination
	- Edge: certificates are terminated at the route, so TLS certificates must be configured in the route
	- Pass-through: termination at the Pods, the Pods are responsible for serving the certificate. Mutual authentication
	- Re-encryption: TLS traffic is terminated at the route, new encrypted traffic is established with the Pods
- Unsecure routes require no key or certificates

Creating insecure route

```sh
oc expose service SERVICE --hostname service.example.com
```

If hostname is not specified, the name `routename.projectname.defaultclusterdomain` is used.

OpenShift route only knows about the route name not the CoreDNS DNS server.

> [!NOTE] CoreDNS
> DNS has a wildcard domain name that sends traffic to the IP address that runs the router software, which will further take care of the specific name resolution.
> 
Route name must always be a subdomain of the cluster wildcard domain

---

OpenShift DNS:
- Services is the first level of exposing applications
- Route provides DNS names to make services accessible
- OpenShift has an internal DNS server, which only reachable from the cluster
- To make OpenShift services accessible by name, from outside, Wildcard DNS is needed on the external DNS
- The Wildcard DNS resolves to all resources created within a domain
- External DNS has wildcard DNS to the OpenShift LoadBalancer
- The LoadBalancer provides a front-end to the control nodes
- The control nodes run the Ingress controller and  are a part of the cluster

