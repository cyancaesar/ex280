A role is an API resource which gives users access to OpenShift resources, based on verbs.
- get, list, watch, create, update, patch, delete

Roles:
- Cluster Roles: created when OpenShift is installed.
- Local Roles: provides access to project-based resources
- View currently existing cluster roles: `oc describe clusterrole.rbac`

Role Binding: connects a cluster role to a user or group.
- `oc describe clusterrolebinding.rbac`
- `oc describe rolebinding.rbac`: non-cluster scope
- `oc describe rolebinding.rbac -n project`: local roles assigned to a project

Default Roles:
- admin: gives full control to all project resources
- basic-user: read access to projects
- cluster-admin: perform any action in the cluster
- cluster-status: allow to request status information
- edit: create and modify application, but no access to permission, quotas, or limit ranges
- self-provisioner: allow to create new projects
- view: view but not modify project resources

*admin* role gives full project permissions, *edit* role is the developer user.

User object types:
- Regular: user object granted access to the cluster platform
- System: auto-created to allow system components to access specific resources
	- `system:admin`: full admin access
	- `system:openshift-registry`: registry access
	- `system:node:server1.example.com`: node access
- Service Account: special system accounts used to give extra privileges to pods and deployments
	- deployer: creates deployments
	- builder: creates build configs in S2I

Cluster admin uses `oc adm policy` to manage cluster and namespace roles:
- `oc adm policy [add|remove]-cluster-role-to-user ROLE USER`

Who can: `oc adm policy who-can delete user`

Create roles :
- `oc create role podview --verb=get --resource=pod -n user-project
- `oc adm policy add-role-to-user podview user --role-namespace=user-project -n user-project`

Create cluster role:
- `oc create clusterrole podviewonly --verb=get --resource=pod`
- `oc adm policy add-cluster-role-to-user podviewonly user`

---

Secrets
- Its a base64 encoded *ConfigMap*
- To protect the secret, Etcd can be encrypted
- Secret stores passwords, config file, SSH keys and tokens

Secrets type:
- docker-registry
- generic
- tls

Secrets in OpenShift are mainly used:
- To store credentials that is used by Pods in a microservice architecture
- To store TLS certificates and keys
- A TLS secret stores the certificate as tls.crt and the certificate key as tls.key 
- Secrets can be mounted as a volume and create a *pass-through* route to the application

Creating  a generic secret:

```sh
oc create secret generic mysecret --from-literal user=root --from-literal pass=password

# Storing SSH keys
oc create secret generic ssh-keys --from-file id_rsa=~/.ssh/id_rsa --from-file id_rsa.pub=~/.ssh/id_rsa.pub
```

Creating a TLS secret:

```sh
oc create secret tls secret-tls --cert certs/tls.crt keys/tls.key
```

Secrets can be referred to as variables, or as files from the Pod. To update environment variable of a pod `oc set env`

```sh
oc set env deployment/mysql --from secret/mysql --prefix MYSQL_
```

Secret can also be mounted as volumes `oc set volume`

```sh
oc set volume deployment/mysql --add --type secret --mount-path /run/secrets/mysql --secret-name mysql
```

```sh
oc create secret generic mysql --from-literal user=sqluser --from-literal password=password --from-literal database=secretdb --from-literal hostname=mysql --from-literal root_password=password

oc new-app --name mysql --image bitnami/mysql
oc set env deployment/mysql --from secret/mysql --prefix MYSQL_
```

---

ServiceAccounts:
- Used by pod to determine pod access privileges to system resources
- Default ServiceAccount is used for limited access to cluster resources
- After creating a ServiceAccount, specific access privileges need to be set

Creating a ServiceAccount:

```sh
oc create serviceaccount mysa # -n to assign SA to a namespace
```

Use role binding to connect to specific role, or Security Context Constraint (SCC).

Security Context Constraint (SCC) is a resource similar to Kubernetes Security Context resource. It provides a way to limit access from a Pod to the host environment.

SCCs to control:
- Running privileged containers (root)
- Requesting capabilities to a container
- Use host directories as volumes
- SELinux context and the fun stuff
- Changing user ID

Viewing SCCs

```sh
oc get scc

# More details
oc describe scc NAME
```

Viewing SCC in  a Pod

```sh
oc describe pod POD | grep scc
```

If a Pod can't run because of an SCC, debug with

```sh
oc get pod POD -o yaml | oc adm policy scc-subject-review -f -
```


> [!NOTE] Change SCC in a Pod
> For changing a SCC of a Pod, create a service account and use it in the Pod.

Run an nginx as a non cluster admin user and see the Pod errors out

```sh
oc run pod --image=nginx
oc get pods

# Shows that the pod must use restricted-v2 SCC
oc get pods POD -o yaml | oc adm policy scc-subject-review -f -
```

Recap:

- Create a service account
- Connect the service account with an SCC
- Then it can be bound to a Pod or a Deployment
- `anyuid` SCC make the pod to run anyway


Non-root containers:

- OpenShift, by default, restrict running containers as root
- Containers that run as root, has root privileges on the container host
- quay.io images are made with OpenShift in  mind
- bitnami also, but not at the end of 2025 August

Consideration when running non-root containers:

- Privileged ports in the container cannot be bind
- In OpenShift, not an issue, as it is accessed through services
- Limitation when accessing files

