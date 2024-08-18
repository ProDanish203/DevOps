# Kubernetes - MERN stack deployment

## Getting Started

Common terminologies in Kubernetes:

- Pods: Smallest deployable units in Kubernetes
- Services: Expose applications running on Pods
- Deployment: Manage Pods and ReplicaSets
- Secrets: Store sensitive information
- ConfigMap: Store configuration data

### Some common commands

```
kubectl get pods
kubectl get pods -o wide
kubectl get secrets
kubectl get configmap
kubectl get services
kubectl get deployments

# To add file
kubectl apply -f [yaml_file_name]

kubectl apply -f [directory]
kubectl delete -f [directory]

# To remove files
kubectl delete pods --all
kubectl delete deployment [name]
kubectl delete service [name]
kubectl delete secret [name]
kubectl delete configmap [name]

minikube ip
minikube service [service_name]
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
kubectl proxy

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login
```

- Create a Bearer token with: `kubectl -n kubernetes-dashboard create token admin-user` and paste the token in dashbooard

### Best Practices

- Use meaningful labels for pods
- Understand Service types: ClusterIP, NodePort, LoadBalancer
- Use Deployments for automatic pod restarts and updates
- Manage configurations with ConfigMaps and Secrets
- when creating a pod, label is important as it will be used in creating the service to specify on which port should the service be applied
- there are 3 types of Service type: ClusterIP, NodePort, LoadBalancer
- NodePort is used to externally communicate on the port
- If our application crashes then the service will automatically spin the image again and restart our pod/service but there is some down time of like 2-6 seconds (depends).
- To overcome this downtime we use Deployment Object

# Nodejs Example:

## Service file

```
apiVersion: v1
kind: Service
metadata:
  name: jokes-api-service
spec:
  selector:
    app: jokes-api
  type: LoadBalancer
  ports:
    - port: 7000
      targetPort: 8000
```

## Deployment file

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jokes-api-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: jokes-api
  template:
    metadata:
      labels:
        app: jokes-api
    spec:
      containers:
        - name: jokes-api-container
          image: prodanish203/jokes-api
          ports:
            - containerPort: 8000
          env:
            - name: MONGO_URI
              valueFrom:
                secretKeyRef:
                  name: jokes-api-secret
                  key: MONGO_URI
            - name: PORT
              valueFrom:
                secretKeyRef:
                  name: jokes-api-secret
                  key: PORT
```

## Secrets file

```
apiVersion: v1
kind: Secret
metadata:
  name: jokes-api-secret
type: Opaque
data:
  MONGO_URI: bW9uZ29kYjovL21vbm9nZGItc2VydmljZS9rOHMtam9rZXM=
  PORT: ODAwMA==
```

### PersistentStorage and PersistentStorageClaim

- Purpose: Provides durable storage that survives pod restarts or rescheduling, essential for stateful applications and data persistence.
- Key Concepts: PersistentVolume (PV) is a piece of storage in the cluster, while PersistentVolumeClaim (PVC) is a request for storage by a user.
- How it works: Admins create PVs, users create PVCs to request storage, and Kubernetes binds PVC to a suitable PV.
- Benefits: Decouples storage provisioning from pod lifecycle, allows for different storage classes, and supports dynamic provisioning in cloud environments.
- Use cases: Ideal for databases, file storage, and shared application data.
- Storage Classes: Define different types of storage with varying performance characteristics, enabling automatic provisioning of PVs.
- Reclaim Policies: Include Retain (keep data when PVC is deleted), Delete (remove data when PVC is deleted), and Recycle (basic scrub and make available again).