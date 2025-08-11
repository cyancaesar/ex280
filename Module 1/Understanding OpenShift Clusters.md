Red Hat OpenShift Container Platform is just a Kubernetes on steroids.

OpenShift offerings:

- Red Hat OpenShift Container Platform: on premise or in the cloud
- Red Hat OpenShift Dedicated: a Red Hat managed cluster offered in major cloud providers
- Cloud-provider managed OpenShift
- Red Hat OpenShift Online: a Red Hat managed OpenShift infrastructure
- Red Hat OpenShift Kubernetes Engine: a Kubernetes
- Red Hat Code Ready Containers: a minimal OpenShift that can be used for homelabs and developments
- OKD: OpenShift open source upstream

OpenShift component:

- Control node: a node that runs OpenShift services
- Compute node: a container orchestrated by the control node
	- From OpenShift v4, the compute node is CoreOS, before was a traditional RHEL
- The container runtime is CRI-O
- Kublet is found on the compute node
- Pre-installed features like registry, monitoring, S2I
- CoreOS, the immutable OS

OpenShift features:

- High Availability
- Load balancing
- Automatic scaling
- Logging and monitoring
- Service Discovery
- Storage
- Source to Image
- The application catalog
- Cluster extensibility
- Operators

Operators in OpenShift:

- They are applications that extends default Kubernetes functionality
- They automate tasks like deploying, update, and backup
- Operators are started from container images and use Custom Resources to store their settings

Operator Framework is a toolkit for building, testing, and packaging operators. It provides:
- Operator Software Development Kit: contains code examples and a container image that can be used as a template
- Operator Life Cycle Manager (OLD): provides an application that manages the operator lifecycle

OperatorHub a web interface that is used as an operator registry.

OpenShift cluster operators are managed by OpenShift cluster version operator. It provides OpenShift extension APIs and cluster infrastructure services.
- The OAuth server for authenticating access to the control node and APIs
- Core DNS, the internal DNS server
- Web Console, the management interface
- Internal image registry, for storing internal images
- Monitoring, which generates metrics and alerts about cluster health

Operator is used to run the interesting things in OpenShift.

OpenShift operators and its managed applications share the same project, found as the `openshift-*` projects.

Every cluster operator defines a custom resource of the type *ClusterOperator*. The *ClusterOperator* API exposes information about the specific operator components, such as health status or version information.

### OpenShift Architecture

- There are a one controller node, and more for high availability
- There are worker node that runs different services
- There are kublets
- There are static pods, OpenShift extends it and add:
	- OpenShift API server
	- OpenShift Controller Manager
	- CoreDNS
	- Operator
- Load balancer

 Installation methods:
 
 - Full-stack Automation: requires minimum installation data to set up a fully functional OpenShift cluster on pre-existing cloud or virtualization provider
 - Pre-existing Infrastructure: requires you to setup everything which then used by the OpenShift

Deployment process:

- A bootstrap machine is created
- The bootstrap machine boots and starts hosting the resource required for booting control plane machines
- Control plane machines get remote resources from the bootstrap machine and finish booting
- Control plane machines create an etcd cluster and start a temporary Kubernetes cluster
- The temporary control plane schedules the final control plane
- Temporary control plane is replaced by the final control plane
- The bootstrap node injects OpenShift-specific components into the control plane
- The installer remove the bootstrap machine

---

### Setting up Local OpenShift (Code Ready Container)

- Head to Red Hat website and download the installer and pull secret
- Install and run `crc setup`
- Then `crc start -p pull-secret`
- It will print the information to login. Login with the administrator: `oc login -u kubeadmin -p password https://api.crc.testing:6443`
- Navigate the projects under the administrator `oc projects`

### CRC Considerations

- CRC clusters need to be **rebuilt** often
- `crc cleanup` to delete the old cluster
- `crc setup; crc start` to run a new cluster
- Keep the pull secret
- `crc console --credentials` print current credentials
- `crc console` log in on the console
- `crc oc-env` to print the command needed to add the `oc` executable to the environment variabls
- `oc login -u developer`
- `oc get co` verify the availability of operators