The need for certificates in OpenShift:
- PKI certificates are everywhere in OpenShift
- To secure resources (routes)
- To use public keys, need to be signed by Certificate Authority
- Self-signed certificates for internal uses, instead of externally signed, which is helpful when exposed to the Internet

Creating a self-signed certificates procedure:
- Creating the CA
- Creating the certificate
- Self-signing the certificate

Creating the CA
```sh
mkdir openssl
openssl genrsa -des3 -out my_ca.key 2048
openssl req -x509 -new -nodes -key my_ca.key -sha256 -days 3650 -out my_ca.pem
```

Creating the certificate
```sh
openssl genrsa -out tls.key 2048
openssl req -new -key tls.key -out tls.csr
```

Self-signing the certificate
```sh
openssl x509 -req -in tls.csr -CA my_ca.pem -CAkey my_ca.key -CAcreateserial -out tls.crt -days 1650 -sha256
```

---

#### Edge Routes

TLS termination happens on the route, then the connection between the route and the application is insecure.

```sh
oc new-project myproject # the project name must match the certificate common name
oc create configmap linginx1 --from-file linginx1.conf
oc create sa linginx-sa
oc adm policy add-scc-to-user anyuid -z linginx-sa # as cluster admin
oc create -f linginx-v1.yaml
oc get pods
oc get svc
oc create route edge linginx1 --service linginx1 --cert=tls.crt --key=tls.key --ca-cert=my_ca.pem

curl -svv https://linginx-myproject.apps-crc.testing
curl -s -k https://linginx-myproject.apps-crc.testing
```

#### Pass-through Routes

Pass-through routes configures an end-to-end encryption by making the route pass forward the certificate. The application must have the secret certificate and key mounted.

```sh
openssl genrsa -des3 -out my_ca.key 2048
openssl req -x509 -new -nodes -key my_ca.key -sha256 -days 3650 -out my_ca.pem
```

```sh
openssl genrsa -out tls.key 2048 # CN must matches the route name
openssl req -new -key tls.key -out tls.csr
openssl x509 -req -in tls.csr -CA my_ca.pem -CAkey my_ca.key -CAcreateserial -out tls.crt -days 1650 -sha256
```

```sh
oc create secret tls linginx-certs --cert tls.crt --key tls.key
oc get secret linginx-certs -o yaml
```

```sh
oc create configmap nginxconfigmap --from-file default.conf
oc create sa linginx-sa
oc adm policy add-scc-to-user anyuid -z linginx-sa # runs as cluster admin
```

```sh
oc create -f linginx-v2.yaml
oc create route passthrough linginx --service linginx2 --port 8443 --hostname linginx-default.apps-crc.testing
oc get route
oc get svc
```

```sh
oc debug -t deployment/linginx2 --image registry.access.redhat.com/ubi8/ubi:8.0
curl -s -k https://172.25.201.41:8443 # works from same network
curl https://linginx-default.apps-crc.testing
curl --insecure https://linginx-default.apps-crc.testing
```

---

#### Network Policies

By default in OpenShift and Kubernetes, there is no restriction to network traffic and Pods can always communicate, even in different namespaces. Network Policies can be used to limit this.
- No policy matches, traffic is denied
- If no Network Policy is used, all traffic is allowed

Network policy identifiers:
- Pods: podSelector
- Namespaces: namespaceSelector
- IP blocks: ipBlock

If cluster monitoring or exposed routes are used, Ingress from them needs to be included in the network policy:
- `spec.ingress.from.namespaceSelector.matchlabels`
	- `network.openshift.io/policy-group: monitoring`
	- `network.openshift.io/policy-group: ingress

```sh
oc apply -f netwrkpolicy.yaml
oc expose pod nginx --port 80
oc exec -it busybox -- wget --spider --timeout 1 nginx
oc label pod busybox access=true
oc exec -it busybox -- wget --spider --timeout 1 nginx
```


> [!NOTE] 
> Some advanced example to restrict communication between namespaces. Check it out

---

#### Troubleshooting OpenShift Networking

*debug* command:
- Gets an exact copy of a Pod and troubleshoot from there, without touching the original failing Pod
- *debug* is useful when you cannot *exec* or *rsh*
- It starts a shell inside the **first** container of the referenced Pod
- It copies and strips all labels, no probes, and command changed to */bin/sh*
- `--as-root` or `--as-user=1000` arguments to change user

```sh
oc create deployment tnginx --image nginx
oc get pods
oc debug deployment/tnginx --as-user 10000 # range might differs
# nginx

oc debug deployment/tnginx --as-root
# nginx
```

