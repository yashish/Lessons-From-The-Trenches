Kubernetes is an open source project that can be used with any cloud provider in any machine you own.
It works with Docker to deploy docker containers. Kubernetes is like Docker compose for multiple machines!
While Docker compose allow multiple containers in your single machine.
Standardized way of describing the resources to be created and managed in the Kubernetes Cluster.
This can be used in any cloud platform unlike the Task Definitions yaml in ECS for AWS only deployment.
Without rewriting the entire config file, you can also swap cloud-provider specific settings in this file.
Standardized way of deploying resources

apiVersion: v1
kind: Service
metadata:
	name: auth-service
	annotations:

spec:
	selector:
		app: auth-app
ports:
	- protocol: TCP
	  port: 80
	  targetPort: 8080
type: LoadBalancer
...

------------------

Kubernetes Object Model
Namespaces, Nodes, Pods, ReplicaSets, Deployments, DaemonSets
Labels, Selectors

Nodes:
Nodes are virtual identities assigned by Kubernetes to the systems part of the cluster - whether Virtual Machines, bare-metal, Containers. Each node is managed with the help of two Kubernetes node agents - kubelet and kube-proxy, while it also hosts a container runtime. The container runtime is required to run all containerized workload on the node - control plane agents and user workloads. The kubelet and kube-proxy node agents are responsible for executing all local workload management related tasks - interact with the runtime to run containers, monitor containers and node health, report any issues and node state to the API Server, and manage network traffic to containers.

There are two distinct types of nodes - control plane and worker. A typical Kubernetes cluster includes at least one control plane node, but it may include multiple control plane nodes for the High Availability (HA) of the control plane. Collectively, the control plane node(s) and the worker node(s) represent the Kubernetes cluster. A cluster’s nodes are systems distributed either on the same private network, across different networks, even across different cloud networks.

Namespaces:

$ kubectl create namespace new-namespace-name 

Namespaces provides a solution to the multi-tenancy requirement of today's enterprise development teams. 

If multiple users and teams use the same Kubernetes cluster we can partition the cluster into virtual sub-clusters using Namespaces. The names of the resources/objects created inside a Namespace are unique, but not across Namespaces in the cluster.

To list all the Namespaces, we can run the following command:

$ kubectl get namespaces

NAME              STATUS       AGE
default           Active       11h
kube-node-lease   Active       11h
kube-public       Active       11h
kube-system       Active       11h

Pods:
A Pod is the unit of deployment in Kubernetes, which represents a single instance of the application. A Pod is a logical collection of one or more containers, enclosing and isolating them to ensure that they:

* Are scheduled together on the same host with the Pod.
* Share the same network namespace, meaning that they share a single IP address originally assigned to the Pod.
* Have access to mount the same external storage (volumes) and other common dependencies

Pods are ephemeral in nature and cannot self heal. Stand-alone Pod object YAML below.

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    run: nginx-pod
spec:
  containers:
  - name: nginx-pod
    image: nginx:1.22.1
    ports:
    - containerPort: 80

The above definition manifest, if stored by a def-pod.yaml file, is loaded into the cluster to run the desired Pod and its associated container image. Use 'create' or 'apply' (advanced Kubernetes practitioners) to run the pod

$ kubectl create -f def-pod.yaml

$ kubectl apply -f def-pod.yaml