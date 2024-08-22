
# Kubernetes

Kubernetes is an open-source containrer orchestration tool developed by google which helps you manage containerized applications in different deployment environments

What problems does Kubernetes solve?

- Increased usage of containers
- Trend from Monolithic architecture to Microservices
- Demand for an efficient way to manage hundreds of containers

What features do orchestration tool offers ?

- High availability or no downtime: Applications are always accessible to users
- Scalability and high performance: Ability to handle increased loads efficiently
- Disaster recovery: Automated backup and restore capabilities

## Kubernetes Components:

### Node:

- A worker machine in the Kubernetes cluster
- Can be a physical or virtual machine
- Runs pods and is managed by the control plane

### Pod:

- Smallest unit of Kubernetes
- Abstraction over container, providing a layer on top of a container
- Usually hosts one application per pod
- Each pod gets its own IP address
- Pods can die very easily, and a new one is created on its place with a new IP address, so IP address can be changed
- If a pod dies, a new one is created with a new IP address
- When a pod dies and restarts, it pulls the image from the remote repository and reruns the application, which may cause brief downtime.

### Service:

- Provides a permanent IP address for accessing pods
- Lifecycle of Pod and Service are not connected
- Two types:
    - External Service (e.g., your website)
    - Internal Service (e.g., database)

### Secret:

- Used to store sensitive data
- Base64 encoded
- Keeps confidential information separate from application code

### ConfigMap: 

- Stores external configuration for your application
- Allows separation of configuration from application code

### Deployments: 

- If one pod dies, our application will also go down, so to overcome this we use Deployments and ReplicaSets to create replicas of the same pod, so it will create a number of pods, if one goes down, other will handle the traffic.
- Manages a set of identical pods
- Ensures high availability by creating replicas of pods
- Acts as a load balancer to manage high traffic with minimal latency
- In production environments, you typically work with Deployments rather than individual Pods

### StatefulSet: 

- Used for stateful applications like databases
- Maintains a sticky identity for each pod
- Ensures stable storage and network identifiers across pod rescheduling

### Volumes: 

- Provides persistent storage for pods
- Allows data to survive container restarts and pod rescheduling
- Various types available, including local storage and cloud-provider specific options

### Storage in Kubernetes:

- If a database pod goes down, data stored within the container is lost
- Kubernetes doesn't manage data persistence by default
- Persistent Volumes (PV) and Persistent Volume Claims (PVC) are used to handle persistent storage
- Storage Classes can be used to dynamically provision storage resources

### Additional Kubernetes concepts:

- Namespace: Virtual cluster within a physical cluster, used for resource isolation\
- Ingress: Manages external access to services within a cluster
- Helm: Package manager for Kubernetes, simplifying application deployment
- Horizontal Pod Autoscaler: Automatically scales the number of pods based on CPU utilization or custom metrics

## Kubernetes Architecture:

### Worker Nodes:

- Perform the actual workload
- Key components:
    - Kubelet: Agent that runs on each node, ensuring containers are running in a pod
    - Container Runtime: Software responsible for running containers (e.g., Docker, containerd)
    - Kube Proxy: Network proxy that maintains network rules on nodes
- Cluster Structure:
    - A Kubernetes cluster consists of multiple nodes
    - Each node must have container runtime and kubelet services installed
    - Communication between nodes is facilitated by Services, which act as load balancers
    - Kube Proxy, installed on every node, is responsible for forwarding requests from services to pods and maintaining network rules

How do you interact with this cluster ? like how to schedule pod, monitor, reschedule/restart pod? join a new Node ?

### Master Node (Control Plane):

The master node manages the cluster state and worker nodes. Four essential services run on every master node:

- API Server:
    - Acts as the cluster gateway
    - Handles all API operations
    - Serves as a gatekeeper for authentication and authorization
    - Validates and processes API requests
- Scheduler:
    - Receives requests from the API server
    - Decides where to place pods based on resource requirements, policies, and constraints
    - Considers factors such as resource availability, affinity/anti-affinity rules, and taints/tolerations
- Controller Manager:
    - Monitors cluster state
    - Detects and responds to changes, such as pod failures
    - Attempts to maintain the desired cluster state
    - Makes requests to the scheduler to reschedule failed pods
    - Includes various controllers: Node Controller, Replication Controller, Endpoints Controller, etc.
- etcd:
    - Distributed key-value store that serves as the cluster's "brain”
    - Stores all cluster configuration data and state
    - Provides a reliable way to store and retrieve cluster information
    - Does not store application data
    - Crucial for communication between master processes and worker processes
    - A cluster state information which is used for master proceses to communicate with the workprocess and vice versa.

In production environments, a Kubernetes cluster usually consists of:

- Multiple master nodes for high availability and load distribution
- A load balancer in front of API servers to distribute client requests
- Multiple worker nodes to handle application workloads

This architecture ensures that Kubernetes can efficiently manage containerized applications at scale, providing features like self-healing, load balancing, and automated rollouts and rollbacks.

## Kubernetes Networking:

### Service networking

- Abstracts the way Pods are accessed within a Kubernetes cluster
- Provides a stable endpoint for a set of Pods, regardless of their actual IP addresses or locations
- Uses labels and selectors to group Pods under a Service
- Offers load balancing across multiple Pod instances
- Supports different types: ClusterIP, NodePort, LoadBalancer, and ExternalName

### Pod networking

- Each Pod gets its own IP address
- Containers within a Pod share the same network namespace
- Containers can communicate with each other using localhost
- Pods on the same node can communicate directly
- Pods on different nodes communicate through overlay network

### Network policies

- Define how groups of Pods are allowed to communicate
- Act as a firewall for controlling traffic flow between Pods
- Can specify both ingress and egress rules

### Ingress controllers

- Manages external access to services within a cluster
- Provides HTTP/HTTPS routing, SSL termination, and name-based virtual hosting
- Acts as a reverse proxy and load balancer
- Typically runs as pods within the cluster
- Popular implementations include NGINX Ingress Controller, HAProxy, and Traefik

## Kubectl commands

```bash
kubectl get node
kubectl get pods
kubectl get pods -o wide # for more details
kubectl get secrets
kubectl get configmap
kubectl get services
kubectl get deployments
kubectl get replicaset
kubectl get all

kubectl describe pod [pod_name] # for complete information

# To add file
kubectl apply -f [yaml_file_name]
kubectl create -f [yaml_file_name]

kubectl apply -f [directory]
kubectl delete -f [directory]

# To remove files
kubectl delete pods --all
kubectl delete deployment [name]
kubectl delete service [name]
kubectl delete secret [name]
kubectl delete configmap [name]

kubectl logs [pod_name]
kubectl get pods --namespace=kube-system

# To check the status of rolling updates
kubectl rollout status deployment [deployment_name]

minikube start --driver=hyperv
minikube status 
minikube help
minikube ip
minikube service [service_name]
minikube dashboard # open dashboard
minikube service [service_name]

kubectl cluster-info
# SSH into Kubernetes node
minikube ssh
ssh docker@IP-Address-Of-Minikube
default password: tcuser 

```

### Pod Yaml File

```bash
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: myapp
      image: repo-link
      ports:
        - containerPort: 8000
```

### Service Yaml File

```bash
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  type: LoadBalancer
  ports:
    - port: 7000
      targetPort: 8000
```

### Secret Yaml file

```bash
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secret
type: Opaque
data:
  MONGO_URI: value-in-Base64
  PORT: value-in-Base64

```

### ConfigMap Yaml file

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  mongo-url: mongo-service

```

### Deployment Yaml file

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-name
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp-container
          image: prodanish203/myapp
          ports:
            - containerPort: 8000
          env:
            - name: MONGO_URI
              valueFrom:
                secretKeyRef:
                  name: myapp-secret
                  key: MONGO_URI
            - name: PORT
              valueFrom:
                secretKeyRef:
                  name: myapp-secret
                  key: PORT
```


### Kubernetes Dashboard

- Create a file dashboard.yaml and run it using `kubectl apply -f dashboard.yaml`
- Create a sample user

```
apiVersion: v1
kind: ServiceAccount
metadata:
name: admin-user
namespace: kubernetes-dashboard

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
name: admin-user
roleRef:
apiGroup: rbac.authorization.k8s.io
kind: ClusterRole
name: cluster-admin
subjects:

- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard

```

### To run dashboard

```
# without minikube
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443

https://localhost:8443/

# With minikube
minikube dashboard
```

- Create a Bearer token with: `kubectl -n kubernetes-dashboard create token admin-user` and paste the token in dashbooard

### To install Nginx Ingress Controller
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.43.0/deploy/static/provider/cloud/deploy.yaml


# To check installation
kubectl get pod -n ingress-nginx
```

### Some other points

- when creating a pod, label is important as it will be used in creating the service to specify on which port should the service be applied
- there are 3 types of Service type: ClusterIP, NodePort, LoadBalancer
- NodePort is used to externally communicate on the port
- If our application crashes then the service will automatically spin the image again and restart our pod/service but there is some down time of like 2-6 seconds (depends).
- To overcome this downtime we use Deployment Object


## Namespaces

 Namespaces are a powerful feature in Kubernetes that allow for the organization and management of resources within a cluster. They provide a way to create virtual clusters within a physical cluster, enabling better resource allocation, access control, and isolation among different teams and applications. 

### **Key Features of Namespaces**

- **Organize Resources**: Namespaces help in organizing resources logically, making it easier to manage applications and services. For example, you can group resources related to databases, monitoring tools, the Elastic Stack, and Nginx Ingress under specific namespaces.
- **Virtual Clusters**: Namespaces act as virtual clusters inside a physical cluster, allowing multiple teams to work independently without interfering with each other's resources.

![namespaces](https://github.com/user-attachments/assets/7128c49c-4a12-4a10-b9d2-f3db0e3e2b21)

### **Default Namespaces in Kubernetes**

Kubernetes comes with four default namespaces, each serving a specific purpose:

1. **kube-system**
    - Contains system processes managed by Kubernetes.
    - It is advisable **not to create or modify resources** in this namespace unless necessary, as it can affect cluster stability.
    - Includes critical components such as the Kubernetes master processes and `kubectl`.
2. **kube-public**
    - Designed for publicly accessible data.
    - Contains a ConfigMap that holds cluster information, which can be accessed by all users, including those who are not
    authenticated.
3. **kube-node-lease**
    - Stores information about the heartbeats of nodes in the cluster.
    - Each node has an associated lease object in this namespace, which helps determine the availability of nodes.
4. **default**
    - This is the namespace where resources created without a specified namespace are located.
    - It serves as the default working area for users who do not define a namespace for their resources.

### Need for Namespaces

Namespaces are essential for several reasons:

- **Resource Grouping**: They allow for logical grouping of resources, such as databases, monitoring tools, and application services, making management easier.
- **Conflict Resolution**: In environments with multiple teams working on the same application,
namespaces help avoid resource conflicts by allowing each team to operate within its own isolated namespace.
- **Resource Sharing**: Namespaces facilitate resource sharing for staging and deployment environments, enabling teams to collaborate effectively.
- **Access Control**: Namespaces can enforce access controls and resource limits, ensuring that teams have the necessary permissions while maintaining security.
- **Isolation**: Each team can have its own isolated environment, reducing the risk of accidental interference with other teams’ resources.

### Characteristics of Namespaces

- **Isolation**: Most resources in Kubernetes cannot be accessed across namespaces. Each namespace must define its own ConfigMap, Secrets, and other resources.
- **Global Resources**: Some components, such as Volumes and Nodes, exist outside of namespaces and are global within the cluster. They cannot be isolated within a namespace.

![namespace-characteristics](https://github.com/user-attachments/assets/ac1f3cc6-8e4d-4e8e-966f-3f5cba2b375c)


### **Example of Creating a ConfigMap in a Namespace**

 Here is an example of how to create a ConfigMap within a specific namespace: 

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongo-config
  namespace: my-namespace
data: 
  db_url: mongo-service.database  # The dot notation specifies the service's namespace
```

### **Switching Between Namespaces**

To switch between active namespaces in your Kubernetes environment, you can use the `kubens` command-line tool. This tool simplifies the process of managing and navigating through different namespaces, enhancing your productivity when working with Kubernetes.


## **Ingress**

Ingress is a powerful resource in Kubernetes that manages external access to services within a cluster. It provides HTTP and HTTPS routing to services based on defined rules, allowing you to control how external users access your applications.

### **Key Features of Ingress**

- **No Open IP Address or Port**: Ingress allows you to expose your services without opening specific IP addresses or ports directly. Instead, it routes incoming requests to internal services based on defined rules.
- **Traffic Routing**: Incoming requests are forwarded to the appropriate internal service based on the specified rules, enabling flexible routing configurations.

![ingress](https://github.com/user-attachments/assets/b2fc2eee-713f-4cad-a6f2-1c5715ad3b33)

## Example Ingress Resource

Here is an example of how to define an Ingress resource in Kubernetes:

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec: 
  rules:
  - host: myapp.com
    http:
      paths:
        - path: /
          pathType: Prefix
          backend: 
            service:
              name: myapp-internal-service
              port:
                number: 8081 
```

**Corresponding Service Definition:**

To route traffic to the appropriate service, you need a corresponding Service resource. Here’s an example of a Service definition: 

```bash
apiVersion: v1
kind: Service
metadata:
  name: myapp-internal-service
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
```

### Ingress Controller

The implementation of Ingress is handled by an **Ingress Controller.** This component is responsible for managing the routing of external traffic to the appropriate services based on the Ingress rules defined. Key responsibilities of the Ingress Controller include:

- **Evaluating Rules**: The Ingress Controller evaluates all defined Ingress rules to determine how to route incoming requests.
- **Managing Redirection**: It handles redirection and rewriting of URLs as specified in the Ingress rules, allowing for flexible traffic management.
- **Entry Point to the Cluster**: The Ingress Controller acts as the entry point for external traffic
into the Kubernetes cluster, enabling secure and organized access to
services.
- **Popular Implementations**: One of the most widely used Ingress Controllers is the **Kubernetes Nginx Ingress Controller**. It is easy to install and configure, providing robust features for managing Ingress resources.

Ingress is a vital component in Kubernetes for managing external access to services. By using Ingress resources and an Ingress Controller, you can effectively route traffic to your applications while maintaining security and flexibility in your cluster architecture. This enhanced version provides clearer explanations, additional context, and maintains a structured format for better readability.

## **Helm - Package Manager**

### What is Helm?

- **Helm** is a package manager for Kubernetes that simplifies the deployment and management of applications. It allows users to package YAML files into reusable charts and distribute them through public and private repositories.
- Helm acts as a **templating engine**, enabling users to define complex Kubernetes applications with customizable configurations.
- It facilitates **release management**, allowing users to install, upgrade, and roll back applications easily.

![helm](https://github.com/user-attachments/assets/6e8a5057-4191-490d-89ac-f4faea282009)

### What are Helm Charts ?

- **Helm Charts** are bundles of YAML files that define the resources needed to deploy an application on Kubernetes. Each chart contains all the necessary configuration files, including deployments, services, secrets, and config maps.
- Users can create their own Helm charts using Helm, push them to a Helm repository, or download and use existing charts from public repositories.

## when to use them ?

Helm is particularly useful in complex Kubernetes environments where applications consist of multiple microservices and configurations. It simplifies the management of these applications by allowing users to:

- Maintain consistent deployments across different environments (e.g., development, staging, production).
- Easily manage application versions and configurations.
- Streamline CI/CD processes by integrating Helm into deployment pipelines.

### What is Tiller? (Server)

- **Tiller** was the in-cluster server component of Helm in version 2, responsible for managing the interaction between the Helm client and the Kubernetes API. It evaluated Helm charts and deployed applications based on defined configurations.
- Tiller was removed in **Helm 3** for security reasons, as it had extensive permissions within the
Kubernetes cluster. The removal of Tiller simplified the architecture and improved security by allowing direct interaction with the Kubernetes API without an intermediary.

### Directory Structure

A typical Helm chart follows this directory structure: 

```bash
mychart/         # Folder name of the chart
	Chart.yaml     # Metadata about the chart (name, version, etc.)
	values.yaml    # Default values for the templates
	charts/        # Folder for chart dependencies
	templates/     # Directory containing the actual template files
```

### Common Helm commands

```bash
helm install [chart_name]    # Installs a Helm chart and deploys the application.
helm upgrade [chart_name]    # Upgrades an existing release to a new version.
helm rollback [chart_name]   # Rolls back to a previous version of a release.
```

Helm is an essential tool for managing Kubernetes applications, providing a streamlined approach to packaging, deploying, and maintaining applications. By using Helm charts, developers can automate complex deployments, manage application configurations more effectively, and enhance the overall efficiency of their Kubernetes workflows.

## Data Persistence In Kubernetes

In Kubernetes, a **volume** is a directory accessible to containers within a pod. Volumes provide a way to persist data beyond the lifecycle of a container or pod. These volumes are essential for data persistence in Kubernetes.

![pvc](https://github.com/user-attachments/assets/89430ee5-4261-49f1-9136-bd6e41f6c845)

### Need for volume:

- **Stateful applications**: Applications like databases require persistent storage to store and retrieve data. If a pod containing a database crashes, the data must be preserved.
- **Shared data**: Volumes enable sharing of data among containers within a pod or across pods. This is useful for scenarios where multiple containers need to access the same data.
- **Data backup and restore**: Volumes provide a mechanism to back up and restore data, ensuring data integrity and enabling disaster recovery.

### Persistent Volume

**Persistent Volumes (PVs)** are  cluster-level resources created using YAML files. They represent actual physical or cloud storage and are not namespaced, making them accessible to the entire cluster across all namespaces.

PVs can be provisioned statically by administrators or dynamically using **Storage Classes**.

### Persistent Volume Claim

**Persistent Volume Claims (PVCs)** are requests for storage resources made by applications. They specify the desired storage size, access mode, and other requirements. Applications need to claim a PV by creating a PVC in the same namespace.

PVCs are bound to PVs based on the requested storage size, access 
mode, and other constraints. If a matching PV is available, the PVC is 
bound to it. If no matching PV exists, the PVC remains unbound until a 
suitable PV becomes available or a Storage Class provisions a new PV 
dynamically.

### Storage Class

**Storage Classes** are used to provision PVs dynamically based on the requested storage requirements in a PVC. They abstract the underlying storage implementation, allowing administrators to define different storage options with varying performance characteristics, backup policies, and access modes.

When a PVC requests storage using a specific Storage Class, the Kubernetes control plane dynamically provisions a new PV that matches the PVC's requirements. This simplifies storage management and enables self-service provisioning for application developers.