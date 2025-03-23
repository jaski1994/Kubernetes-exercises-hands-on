# Kubernetes Basics Cheat Sheet

## 1. Start Minikube
```sh
minikube start
```

## 2. Working with Pods
Pods are the smallest deployable units in Kubernetes, representing a single instance of a running process.
```sh
kubectl run --help
kubectl run nginx --image=nginx
kubectl describe pod nginx
kubectl get pods -o wide

# Generate YAML for Redis pod
kubectl run redis --image=redis --dry-run=client -o yaml
kubectl run redis --image=redis --dry-run=client -o yaml > redis.yaml

# Create pod from YAML
kubectl create -f redis.yaml

# Apply changes (if there are modifications)
kubectl apply -f redis.yaml

# Execute a command inside a running pod
kubectl exec -it <pod_name> -- curl -I http://localhost
```

## 3. Working with ReplicaSets
ReplicaSets ensure a specified number of identical Pods are running at all times.
```sh
kubectl get replicaset
kubectl describe replicaset <replicaset_name>
kubectl explain replicaset
```

### Creating a ReplicaSet via YAML
1. Generate a deployment YAML
```sh
kubectl create deployment example-replicaset --image=nginx:1.25 --replicas=3 --dry-run=client -o yaml
```
2. Modify the YAML file to specify `ReplicaSet`:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: example-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
```
3. Apply the configuration:
```sh
kubectl apply -f example-replicaset.yaml
```

### ReplicaSet Commands
```sh
kubectl edit rs <replicaset_name>
kubectl scale rs <replicaset_name> --replicas=5
```

### Dry Run Modes
- `--dry-run=client`: Validates client-side, ensuring syntax correctness.
- `--dry-run=server`: Validates against the live cluster without applying changes.

## 4. Working with Deployments
Deployments provide declarative updates for Pods and ReplicaSets, allowing for easy scaling and rolling updates.
```sh
kubectl get deploy
kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
kubectl create deployment example-deployment --image=nginx:1.25 --replicas=3
kubectl create deployment example-deployment --image=nginx:1.25 --replicas=3 --dry-run=client -o yaml
```

---
This cheat sheet provides quick Kubernetes commands for Pods, ReplicaSets, and Deployments.
