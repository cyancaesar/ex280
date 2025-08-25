Setting up an exam environment:
- Start with a clean CRC instance
- Generate self-signed TLS certificates

Review in the exam documentation resources:
- Generic: Post installation configuration
- Secrets: Node > Working with Pods > Understanding Secrets
- Identity Provider: Authentication and Authorization > Understanding Identity Provider
- RBAC: Authentication and Authorization > Using RBAC
- Users and Groups: Authentication and Authorization > Understanding Authentication
- SCC: Authentication and Authorization
- Routes: Networking > Configuring Routes
- Network Policy: Networking > Network Policy
- Limit Range: Nodes > Working with Clusters > Setting Limit Ranges
- Quota: Applications > Quotas
- Autoscaler: Nodes > Working with Pods > Automatically scaling Pods
- Project Templates: Applications > Projects > Configuring project creation

Essential helps

Configuring completion
```sh
source <(oc completion)
```

Setting env
```sh
oc set env -h
```

Setting volumes
```sh
oc set volumes -h
```

Setting resource restriction
```sh
oc set resource -h
```

Creating a project template
```sh
oc adm create-bootstrap-project-template
```

Configure policies
```sh
oc adm policy -h
```

Creating a quota
```sh
oc create quota -h
```

Creating a TLS secret
```sh
oc create secret tls -h
```

View cluster role
```sh
oc describe clusterrole.rbac [| grep ^Name]
```