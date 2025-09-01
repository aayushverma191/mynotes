
## What is Deployment?

- A Deployment manages a set of Pods to run an application workload, usually one that doesn't maintain state.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
```

- A Kubernetes Deployment is a higher-level API resource used to manage stateless applications by defining the desired state for a set of identical pods. It ensures the specified number of replicas are running and automatically replaces failed pods. Deployments also support rolling updates and rollbacks, making them ideal for controlled application updates and version management.


![[Pasted image 20250901154759.png]]


## Why do we use Deployment?

- To keep the application always available.
- To scale the number of pods (increase/decrease).
- To update app versions without downtime.
- To roll back easily if something goes wrong.

---

## Workflow

1. **You create a Deployment YAML** (specifies how many replicas, which image, etc.).
2. **Deployment creates a ReplicaSet.**
3. **ReplicaSet creates Pods.**
4. **Kubernetes Scheduler** assigns Pods to nodes.
5. If Pods die → ReplicaSet ensures they are recreated.

# Deployment Commands in Kubernetes

### Basic Commands

```bash

# Create deployment from YAML

kubectl apply -f deployment.yaml

# Delete deployment

kubectl delete -f deployment.yaml

kubectl delete deployment <deployment-name>

# List all deployments

kubectl get deployments

# Get detailed info about a deployment

kubectl describe deployment <deployment-name>

```

---
### Pods & ReplicaSets from Deployment


```bash
# Get all pods created by deployment

kubectl get pods -l app=<label-name>

# Get ReplicaSets controlled by a deployment

kubectl get rs

# Describe ReplicaSet

kubectl describe rs <replicaset-name>

```

---
### Scaling

```bash

# Scale deployment manually

kubectl scale deployment <deployment-name> --replicas=5

# Check rollout status after scaling

kubectl rollout status deployment <deployment-name>

# Autoscale deployment (Horizontal Pod Autoscaler)

kubectl autoscale deployment <deployment-name> --min=2 --max=10 --cpu-percent=80

```

  
---
### Updating Deployments


```bash

# Update container image

kubectl set image deployment/<deployment-name> <container-name>=<new-image>

# Example: update nginx version

kubectl set image deployment/nginx-deployment nginx=nginx:1.27

# Check rollout status

kubectl rollout status deployment/nginx-deployment

# Pause deployment (pause updates)

kubectl rollout pause deployment/nginx-deployment

# Resume deployment

kubectl rollout resume deployment/nginx-deployment

```

---

### Rollback & History

```bash

# View deployment rollout history
kubectl rollout history deployment/<deployment-name>
  
# View specific revision details
kubectl rollout history deployment/<deployment-name> --revision=2
  

# Rollback to previous revision
kubectl rollout undo deployment/<deployment-name>


# Rollback to a specific revision
kubectl rollout undo deployment/<deployment-name> --to-revision=3

```


---

### Debugging


```bash

# Show events related to deployment
kubectl describe deployment <deployment-name>

# Get logs of a specific pod
kubectl logs <pod-name>

# Execute into a running pod (shell access)
kubectl exec -it <pod-name> -- /bin/bash

```

---
### YAML from Running Deployment

```bash
# Export running deployment as YAML
kubectl get deployment <deployment-name> -o yaml > export.yaml

```




