

- **`kind:-`** the type of resource (e.g., `Pod`, `Deployment`, `Service`)

`kind` tells Kubernetes what **type of object** you are creating.

---

### Major Categories of `kind` in Kubernetes

We can group them into **types of objects**:

1. **Workload Resources** (things that actually run containers)
    
	- `Pod`
        
    - `Deployment`
        
    - `ReplicaSet`
        
    - `StatefulSet`
        
    - `DaemonSet`
        
    - `Job`
        
    - `CronJob`
        
2. **Service & Networking Resources**
    
    - `Service`
        
    - `Ingress`
        
    - `Endpoint`
        
    - `EndpointSlice`
        
    - `NetworkPolicy`
        
3. **Configuration & Storage Resources**
    
    - `ConfigMap`
        
    - `Secret`
        
    - `PersistentVolume` (PV)
        
    - `PersistentVolumeClaim` (PVC)
        
    - `StorageClass`
        
4. **Cluster & Namespace Resources**
    
    - `Namespace`
        
    - `Node`
        
    - `ResourceQuota`
        
    - `LimitRange`
        
5. **Access Control & Security** [link](https://k21academy.com/docker-kubernetes/rbac-role-based-access-control/)
    
    - `ServiceAccount`
        
    - `Role`
        
    - `RoleBinding`
        
    - `ClusterRole`
        
    - `ClusterRoleBinding`
        
6. **Custom Resources**
    
    - `CustomResourceDefinition` (CRD) → lets you create your **own kinds**
        
    - Any CRDs installed by operators (e.g., `KafkaCluster`, `Prometheus`, etc.)
        


![[Pasted image 20250831143243.png]]