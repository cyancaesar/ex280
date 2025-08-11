The Web Console:

- The front-end for managing OpenShift
- `crc console` to start it in the default browser
- In full clusters, `oc get routes -n openshift-console` to find the URL to the console pod
- `oc whoami --show-console` to show the URL

Login as `kubeadmin` for full access and `developer` for limited access.

Now within the console, create a project, and add an application using Container Image like nginx. nginx will not run because nginx container requires root privileges and OpenShift is designed to work with rootless containers, use bitnami images.

You can create a resource using Operator, head to OperatorHub install Postgres-related operator, and after installation you configure it.

Lab:

- Run an nginx with the name `mynginxapp` which is based on bitnami/nginx image
- Test nginx service
- Check if any error happened in the cluster


> [!NOTE] Windows
> The deployed URL cannot be resolved in Windows, the quick
> solution for this is to add that URL to hosts file
