#### Control Pod Placement

OpenShift Scheduler:
- Pod scheduling applies 3 types of rules: node labels, affinity rules, and anti-affinity rules
- 3 Steps for Pod Scheduler
	- Node filter: node availability, but also selectors, taints, resource availability and more
	- Node prioritizing: based on affinity rules
	- Select the best nodes: best scoring node, if multiple nodes, round-robin is used

Node labels to Control Pod Placement:
- Nodes can be configured with a label
- A label is an arbitrary key-value pair, set with `oc label`
- Pods can next be configured with a `nodeSelector` property on the Pod

Applying Labels to Nodes
- A label is an arbitrary key-value pair that can be used as a selector for Pod placement
- `oc label node worker1.example.com env=dev`
- `oc label node worker1.example.com env=prod --overwrite`
- `oc label node worker1.example.com env-` remove the label
- `oc get ... --show-labels`

Applying Labels to Machine Sets
- Machine sets: a group of machines that is created when installing OpenShift using full stack automation
- Machine sets can be labeled so that nodes generated from the machine set will automatically get a label
- To view which nodes are in which machine set `oc get machines -n openshift-machine-api -o wide`
- `oc edit machineset ...` to set a label in the machine set `spec.metadata`

Configure *NodeSelector* on Pods
- Infrastructure-related Pods in OpenShift are configured to run on a controller node
- Use `nodeSelector` on the Deployment to configure its Pods to run on a node that has a specific label
- `oc edit` to apply `nodeSelector` to existing Deployments
- If a Deployment is configured with a `nodeSelector` and doesn't have any matches, the Pods will show as pending states
	- Fixing this by setting Deployment `spec.template.spec.nodeSelector` to the desired key-value pair

Configure *NodeSelector* on Projects
- It can be set on project such that resources created in the deployment are automatically placed on the right nodes
	- `oc adm new-project --node-selector "env=dev"`
- Default `nodeSelector` on an existing, add annotation to its underlying namespace resource
	- `oc annotate namespace test openshift.io/node-selector="dev" --overwrite`

```sh
# Demonstrate NodeSelector

# login as a developer
oc create deployment simple --image bitnami/nginx
oc get all
oc scale --replicas 4 deployment/simple
oc get pods -o wide
# login as a cluster admin
oc get nodes -L env
oc label node crc env=dev
```

#### Pod Scaling

- The desired number of Pods is set in the Deployment
- The ReplicaSet, or replication controller, ensures that this number of replicas is running
- The deployment uses `selector` to identify the replicated Pods

Deployment and DeploymentConfig:
- Deployment: Kubernetes resource
- DeploymentConfig: OpenShift resource
- DeploymentConfig is created when working with the console
- Deployment is the standard, but when using `oc new-app --as-deployment-config` it will create a DeploymentConfig instead

Scaling Pods manually:
- `oc scale --replicas 3 deployment myapp`
- The new desired number is added to the Deployment, hence it is written to ReplicaSet

Autoscaling Pods:
- OpenShift provides **HorizontalPodAutoscaler** resource
- This resource depends on the OpenShift Metrics subsystem, which is pre-installed in OpenShift 4
- To use autoscaling, resource requests need to be specified so that the autoscaler knows when to do what:
	- Use Resource Requests or project resource limitations to take care of this
- Autoscaling is based on CPU usage, memory utilization, and GPU

```sh
oc autoscale deploy test --min 5 --max 10 --cpu-percent 20
```

---
#### Pod Resource Limitations

- Resource requests and limits are used on a per-application basis
- Quota are enforced on a project or cluster basis
- In Pods `spec.containers.resources.requests`, a Pod can request minimal amounts of CPU and memory resources. Pod scheduler will look for a node that meets these requirements
- In Pods `spec.containers.resources.limits`, a Pod can be limited to a maximum use of resources "CGroup are used on the node to enforce the limits"
- `oc set resources` to set resource requests as well as limits, or edit the YAML
- Resource restriction can be set on individual containers and as the whole deployment
- `oc set resources deployment nginx --requests cpu=10m,memory=10Mi --limits cpu=50m,memory=50Mi`


> [!TIP]
> `oc set resources -h` for examples

```sh
oc create deployment nginx --image bitnami/nginx --replicas 3
oc get pods
oc set resources deploy nginx --requests cpu=10m,memory=1Mi --limits cpu=20m,memory=5Mi
oc get pods
oc describe pods nginx-blahblah
oc set resources deploy nginx --request cpu=0m,memory=0Mi --limits cpu=0m,memory=0Mi
oc get pods
```

Monitoring Resource Availability:
- `oc describe node nodename` to get information about current CPU and memory usage for each Pod running on the node
- `oc adm top`: requires Cluster Monitoring Operator to be running

#### Quota

Quotas are used to apply limits on resources:
- On the number of objects (Pods, Services, Routes)
- On compute resources (CPU, memory, and storage)
- Useful to prevent exhaustion of vital resources
- They are applied to new resources, but not the current resources
- *ResourceQuota* resource
- Or `oc create quota myquota --hard service=10,cpu=1400,memory=1.8Gi`

Quota scopes:
- resourcequotas: applied to projects to limit use of resources
- clusterresourcequotas: apply quota with a cluster scope
- Multiple resourcequota can be applied on the same project

> [!TIP]
> `oc create quota -h` for examples
> Try to avoid using YAML for setting up quota :)

Viewing resource quota:
- `oc get resourcequota`: overview of all resourcequota API resources
- `oc describe quota`: show quota from all resourcequota in the current project (cumulative)

Quota-related failures:
- If a modification exceeds the resource count (number of Pods), OpenShift will deny the modification immediately
- If a modification exceeds quota for a compute resource (free RAM), OpenShift will not fail immediately to give the administrator some time to fix the issue
- If a quota that restricts usage of compute resources is used, OpenShift will not create Pods that do not have resource requests or limits set
- It's recommended to use *LimitRange* to specify default values for resource requests

```sh
# login as user
oc new-project quota-project
# login as cluster admin
oc create quota my-quota --hard pods=3,cpu=100,memory=500Mi
oc describe quota
# login as user
oc create deploy nginx --image=bitnami/nginx --replicas=4
oc get all
oc describe rs/nginxc-xxx
oc set resources deploy nginx --requests cpu=100m,memory=5Mi --limits cpu=200m,memory=20Mi

oc scale deploy nginx --replicas 3
oc delete rs ... # delete replicaset
```

#### Limit Range

- Defines default, minimum and maximum values for compute resource requests
- Limit range can be set on a project, and an individual resource
- Limit range can specify CPU and memory for containers and Pods
- Limit range can specify storage for Image and PVCs
- **Template** can be used to apply limit range to any newly created project

```sh
oc new-project limits
# login as user
oc explain limitrange.spec.limits
oc create --save-config -f limits.yaml
oc get limitrange
oc describe limitrange limit-limits
```

#### Quota Range (Manual)

- *ClusterResourceQuota* resource is created at cluster level and applied to multiple projects
- Cluster admin can specify which projects are subject to this quota
	- `openshift.io/requester` annotation to specify project owner
- Or using selectors and labels

Setting a cluster resource quota for all projects owned by user developer:

```sh
oc create clusterquota user-developer --project-annotation-selector openshift.io/requester=developer --hard pods=10,secrets=10
```

Adding label:

```sh
oc create clusterquota dev --project-label-selector env=dev --hard pods=10,secrets=10,services=5

oc new-project dev-project
oc label ns dev-project env=dev
```

- Project user can use `oc describe quota` to see the quotas that applied


> [!TIP]
> Set quota on individual projects, and avoid cluster-wide quota as looking them up in large clusters takes time

#### Quota Range (Template)

A *Template* is an API resource that an set different properties when create a new project
- Quota
- LimitRange
- NetworkPolicies

Generate a YAML for a Template to use

```sh
oc adm create-bootstrap-project-template -o yaml > template.yaml
```

- Under `objects`, add a resource, specifying the `kind` of resource you want to add
- Edit `projects.config.openshift.io/cluster` to use the new template

```sh
# login as admin
oc adm create-bootstrap-project-template -o yaml > template.yaml
oc create -f limitrange.yaml -n openshift-config
oc describe limitrange test-limits
oc edit projects.config.openshift.io/cluster
```

```yaml
spec:
  projectRequestTemplate:
    name: project-request # project-request is the name of template
```

```sh
watch oc get pods -n openshift-apiserver
oc new-project test-project
oc get resourcequotas,limitranges
oc delete project test-project
# then remove the spec from the config
```

---

#### Pod Affinity and Anti-affinity Rules

- Affinity and Anti-affinity defines relations between Pods
- *podAffinity* is a Pod property that tells the scheduler to locate new Pod on the same node as other Pods
- *podAntiAffinity* tells the scheduler not to locate a new Pod in the same node as other Pods
- *nodeAffinity* tells a Pod not to schedule on nodes with specific labels
- The affinity is applied based on Pod labels
- **Required** affinity rules must be met before a Pod can be scheduled on a node
- **Preferred** rules are not guaranteed

*matchExpressions*:
- In affinity rules, it is used on the key-value specification that matches the label
- In the matchExpressions the operator can have NotIn, Exists, DoesNotExist, Lt, and Gt

Node affinity can be used to only run a Pod on a node that meets specific requirements. It works like Pod affinity, works with labels that are set on the node. The required rules must be met and the preferred should be met.

```sh
# login as cluster admin
oc create -f nodeaffinity.yaml
oc describe pod runonssd
oc label node crc disktype=nvme
oc describe pod runonssd
```


#### Taints and Tolerations

- Taint allows a node to refuse a Pod unless the Pod has a matching toleration
- taints are applied to nodes (node spec)
- tolerations are applied to a Pod (Pod spec)
- Taints and Tolerations consist of a key, value, and an effect
- The effect (NoSchedule, PreferNoSchedule, and NoExecute)
- *tolerationSeconds* specifies how long it takes before Pods are evicted when NoExecute is set

```sh
# login as a cluster admin
oc adm taint nodes crc key1:value1:NodeSchedule
oc run mypod --image bitnami/nginx
oc get pods
oc describe pods mypod
oc edit pod mypod
```

```yaml
spec:
  toleration:
    - key: key1
      value: value1
      operator: Equal
      effect: NoExecute
```

```sh
oc adm taint nodes crc key1-
```