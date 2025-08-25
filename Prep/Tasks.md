### Task 1: Setting up Authentication

- Configure your cluster to use the HTPasswd identity provider
- Create the following user and groups
	- Group admins, with users admin and anna
	- Group developers, with users linda and developer
	- Group viewer, with user lisa
	- User anouk
- Ensure that all these users can log in to the cluster
- Remove the default kubeadmin user that was created when installing the cluster

### Task 2: Setting up Authorization

- The group admins has admin permissions on the cluster
- The group developers has view and edit permissions on the cluster
- The group viewers has view permissions on the cluster
- User anouk has view and edit permissions on the namespace test-namespace

### Task 3: Creating a Project Template

- Create a new project template, according to the following requirements
	- Each project has a label with the name of the project
	- NetworkPolicy:
		- Routes can be accessed by Pods in namespaces with the `network.openshift.io/policy-group=ingress` label
		- Pods in the same namespace can communicate with each other
		- Pods are only accessible to Pods in a different namespace if that namespace is configured with the `network.openshift.io/policy-group=ingress` label
	- LimitRange:
		- Container requests 100 millicores of CPU
		- Container requests 50 MiB of memory
		- Container is limited to 200 millicores of CPU
		- Container is limited to 100 MiB of memory
	- ResourceQuotas:
		- Projects are limited to 20 Pods
		- Project can request a maximum of 1 GiB of memory
		- Projects can request a maximum of 2 CPUs
		- Projects can use a maximum of 2 GiB of memory
		- Projects can use a maximum of 4 CPUs

### Task 4: Creating a Project

- As the user developer, create a project with the name local-project
- Ensure that this project inherits settings from the new project template created in Task 3

### Task 5: Running a Secure Application

- Create the project my-project
- In this project, run a deployment that is based on the Docker image `sandervanvugt/openshift` with the name hello-app
- Configure this deployment such that it uses a TLS certificate at the expected location.

### Task 6: Creating a Passthrough Route

- Create a passthrough route to the secure-app service, which points to `secure-app-myproject.apps-crc.testing`
- Use `curl --insecure https://secure-app-myproject.apps-crc.testing` to verify the route

### Task 7: Configure AutoScaling

- Run a Deployment that starts the bitnami nginx service
- Create it to run three Pods by default, and scale to a max of five Pods if the CPU utilization exceeds 70%

### Task 8: Configure MySQL

- As the developer user, use a deployment to create an application named mysql in the microservice project
- Create a secret named mysql, using password as the key and mypassword as the value
- Use the secret to set the MYSQL_ROOT_PASSWORD environment variable to the value of the password in the secret
- Configure the MySQL application to mount a PVC to `/mnt`. The PVC must have a 1 GiB size, and the ReadWriteOnce access mode
- Use a NodeSelector to ensure that MySQL will only run on your CRC node

### Task 9: Configure WordPress

- As the developer user, use a deployment to create an application named wordpress in the microservice project
- Run this application with the anyuid SCC assigned to the wordpress-sa service account
- Create a route to the application using the hostname `wordpress-microservice.apps-crc.testing`
- Use secrets and or ConfigMaps to set environment variables:
	- WORDPRESS_DB_HOST: mysql
	- WORDPRESS_DB_NAME: wordpress
	- WORDPRESS_DB_USER: root
	- WORDPRESS_DB_PASSWORD: from the secret