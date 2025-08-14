- The Linux kernel provides namespaces to offer strict isolation between processes
- Kubernetes implement namespaces in a cluster environment
- OpenShift implements Kubernetes namespaces to manage access to resources for users
- OpenShift namespaces are reserved for cluster-level admin access
- User work with projects to store their resources

Listing all projects
```sh
oc get projects # or oc projects
```

Listing all namespaces
```sh
oc get ns
```

To get all API resources 
```sh
oc api-resources
```

Application is the next step after creating a project, creating an application using `oc new-app`.

While creating an application, Kubernetes resources are created like:
- Deployment (new version) or DeploymentConfig: the application itself, including its cluster properties
- ReplicationController or ReplicaSet: takes care of running pods in a scalable way, using multiple instances
- Pod: the instance of the application that typically runs one container
- Service: a load balancer that exposes access to the application
- Route: the resource that allows incoming traffic by exposing an FQDN

The alternative way is to use Source 2 Image (S2I).

When running application in OpenShift, different Kubernetes resources are used. Each resources is defined in the API to offer specific functionality. Many resources are used to define how a component in the cluster is running:
- Pods define how containers are started using images
- Services implement a load balancer to distribute incoming workload
- Routes define an FQDN for accessing the application
- Resources are defined in the API

![[overview-openshift-architecture.png]]

Create an nginx app `oc new-app bitnami/nginx`

View services status: `oc status`

View overall status: `oc get all`


### Monitoring

- The pod is the representation of the running containers
- `oc logs <pod_name>` to see the logs
- `oc describe` to see how the Pods and other resources are created  in the cluster
- `oc get pod podname -o yaml` to see all status information about the pods

Create a MariaDB deployment

```sh
oc create deployment mymariadb --image=mariadb
```

It will fail because MariaDB expects environment variables.

Get all statuses related to mymariadb

```sh
oc get all --selector app=mymariadb
```

Find the deployment pod and view the logs

```sh
oc get pods
```

View the pod's container logs

```sh
oc logs mymariadb-66d5b94f69-9snn4
```

### The API

- OpenShift is based on the Kubernetes APIs
- On top of Kubernetes APIs, OpenShift-specific APIs are added
- To explore the API `oc api-resources`
- The version of APIs `oc api-versions`
- To see what inside the APIs `oc explain [--recursive]`

