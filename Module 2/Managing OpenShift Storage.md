OpenShift storage:
- OpenShift uses persistent volumes to provision storage
- Storage can be provisioned in a static or dynamic way
- Static provisioning means that the cluster administrator creates the persistent volume manually
- Dynamic provisioning uses *StorageClass* to create persistent volumes on demand
- OpenShift default solution is *StorageClass*
- Developers uses persistent volume claims (PVC) to dynamically add storage to the application

Persistent Volumes (PV):

- PV provides a storage in a decoupled way
- Administrators create persistent volumes of a type that matches the site-specific storage solution
- Alternatively, StorageClass can be used for auto provisioning
- Persistent volumes are available for the entire cluster and not bound to a specific project
- Once a PV is bound to a persistent volume claim (PVC), it cannot service any other claims

Persistent Volume Claims (PVC):

- Defined by the developer to add access to persistent volumes to their applications
- The pod volume uses the PVC to access storage in a decoupled way
- PVC does not bind to a specific PV, but uses any PV that matches the claim requirements
- If no matching PV is found, the PVC will wait until it becomes available
- When matches the PV binds to PVC
- Once bound, it the PV cannot bind to other PVC

StorageClass:

- PV are used to statically allocate storage
- StorageClass allows containers to use the default storage that is provided in a cluster
- Based on properties, PVC can bind to any StorageClass
- If StorageClass is set as default, it will allow to bind automatically without specifying any PVC
- If not, then a PVC needs to specify the name of the StorageClass

Setting StorageClass as the cluster-wide default: 
```sh
oc annotate storageclass standard -- overwrite "storageclass.kubernetes.io/is-default-class=true"
```

Or inside a yaml:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
	storageclass.kubernetes.io/is-default-class: "true"
```

This enabled any PVC that requests a storage without specifying a specific StorageClass to automatically be provisioned with the default StorageClass, same mention as above

Default StorageClass provisioners:

- AWS EBC
- Azure Disk
- Azure File
- Cinder (OpenStack Block Storage)
- VMWare vSphere
- GCE Persistent Disk

A non-default provisioners can be used using the storage class provisioner value `provisioner: kubernetes.io/no-provisioner`

More can be found on OpenShift Container Platform documentations: https://docs.redhat.com/en/documentation/openshift_container_platform/4.8/html/storage/dynamic-provisioning

ConfigMap:

- Used to decouple information
- Types of information that can be stored:
	- Command line parameters
	- Variables
	- ConfigFiles

The procedure of creating a ConfigMap:
- Creating a ConfigMap
	- `kubectl create cm myconf --from-file=my.conf`
	- `kubectl create cm variables --from-env-file=.env`
	- `kubectl create cm special --from-literal=var1=val1`
- Or use `oc` to create the `configmaps`
- Verify it with `kubectl describe cm <configmap_name>`

Operators for storage:

- Operators can be used to configure additional resources based on custom resource definitions
- Different types of storage  are provided as operators
- Local storage operator creates a new LocalVolume resource and sets up RBAC to allow integration of this resource in the cluster

Create a persistent volume for nginx pods to store `/usr/share/nginx/html` persistently