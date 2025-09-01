
1. kind   [imp all info](https://iximiuz.com/en/posts/kubernetes-api-structure-and-terminology/ ) DONE
2. Pod Lifecycle
3. class
4. taint
5. Health Checks (Liveness Probe & Readiness Probe)
6. crd
7. resourcequota
8. Etcd and backup, db use full explain 
9.  limits


```
 k rollout history deploy account-validation-dev-dev -n trusthub-dev
 k rollout undo deployment pizza-app
 k rollout undo deployment pizza-app --to-revision=2
```


```


## 🟢 **CPU Values in Kubernetes**

- CPU is measured in **cores**.
    
- **Units you can define:**
    
    - `1` → 1 full CPU core
        
    - `500m` → 0.5 CPU core (**500 milliCPU**)
        
    - `100m` → 0.1 CPU core
        

💡 Rule:

- `1000m = 1 core`
    
- `250m = 0.25 core`
    

✅ Examples:

|Value|Meaning|
|---|---|
|`100m`|0.1 CPU core (1/10th of a CPU)|
|`500m`|0.5 CPU core (half of one core)|
|`2`|2 full CPU cores|

---

## 🔵 **Memory Values in Kubernetes**

- Memory is measured in **bytes**, usually specified in **Mi (Mebibytes)** or **Gi (Gibibytes)**.
    
- **Units you can define:**
    
    - `Ki` → Kibibyte (1024 bytes)
        
    - `Mi` → Mebibyte (1024 Ki ≈ 1 MB)
        
    - `Gi` → Gibibyte (1024 Mi ≈ 1 GB)
        

✅ Examples:

|Value|Meaning|
|---|---|
|`128Mi`|128 Mebibytes (~134 MB)|
|`512Mi`|512 Mebibytes (~0.5 GB)|
|`1Gi`|1 Gibibyte (1024 Mi, ~1 GB)|
|`2Gi`|2 Gibibytes (~2 GB)|

---

## 🟠 **Requests vs Limits with Units**

Here’s your example again in a **table with meanings**:

|Resource|Requests (Minimum)|Meaning|Limits (Maximum)|Meaning|
|---|---|---|---|---|
|**CPU**|`100m`|0.1 CPU core guaranteed|`200m`|Up to 0.2 CPU core allowed|
|**Memory**|`128Mi`|128 MiB RAM guaranteed|`256Mi`|Up to 256 MiB RAM allowed|

---

✅ **Quick Summary:**

- CPU → measured in **cores (milliCPU for fractions)**
    
- Memory → measured in **MiB/GiB**
    
- **Requests** = minimum guaranteed
    
- **Limits** = maximum allowed
    

---

Would you like me to also give you a **ready-made YAML cheat sheet** (table + common CPU/memory values) that you can reuse when defining resources in your deployments?

```