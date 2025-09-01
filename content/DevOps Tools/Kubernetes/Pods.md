## What is the Pods ?

- Pods are the smallest deployable units in Kubernetes. A Pod represents a single instance of a running process or a group of tightly coupled processes running together on a single node. While containers are commonly used within Pods, Pods can also include multiple containers that share resources and network connectivity.

- The following is an example of a Pod which consists of a container running the image `nginx:1.10.2`.
##### Yaml file for single container pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.10.2
    ports:
    - containerPort: 80
```

[check](https://k21academy.com/docker-kubernetes/kubernetes-pods-for-beginners/)

## Pods with multiple containers

### **1. Single-container Pod**

- Most common use case.
- A Pod has only **one container inside it**.
- You can think of the Pod as just a **box around one container**.
- Kubernetes does not directly manage containers, it manages Pods.

 **Example:**
- Running a simple Nginx web server → one Pod = one Nginx container.

**When to use:**
- When your app is simple and doesn’t need any helper containers.

### **Creating Single Container Pod:**

We have two ways:

**1. Imperative way** –

```bash
$ kubectl run <name of pod> -- image=<name of the image from registry>
```

**2. Declarative way –**

**If you are creating a YAML file to run a single container in a pod, follow the YAML above.** [link](#yaml-file-for-single-container-pod)

---

### **2. Multi-container Pod**

- A Pod can also have **multiple containers inside it**.
- These containers are **tightly linked** and need to work together.
- They **share storage, network, and resources** inside the Pod.
- They act as **one single unit** from Kubernetes’ perspective.

**Example:**
- One container is a **web server** serving static files.
- Another container (sidecar) **keeps updating those files** in the shared volume.
- Both live in the same Pod and help each other.

**When to use:**
- When two (or more) containers **must work closely together** and cannot function well if separated into different Pods.

> When we create a multi-container pod, we only use the declarative way. 

#### Yaml file for multi container pod

```yaml
 apiVersion: v1
 kind: Pod
 metadata:
   name: example-pod
 spec:
   containers:
     - name: web
       image: nginx:1.10.2
       ports:
        - containerPort: 80
       imagePullPolicy: Always
     - name: cache
       image: redis:8.2
       Ports:
        - containerPort: 6379
       imagePullPolicy: Always
```

![[Pasted image 20250831140931.png]]


## **Pod Best Practice**

- It is preferred to use “one container per pod” since running many containers as a part of a single Pod can negatively impact the performance of your Kubernetes workers.

- If we are using multi-container pods then we’re likely to violate the “one process per container” principle. This is important because with multiple processes in the same container, it is harder to troubleshoot the container as logs from different processes will be mixed together, and becomes harder to manage the processes lifecycle.

-  For example, running both the front-end server and the backend server for your service in a single Pod with two containers would not be recommended, and instead should be run as separate Pods.

---
## Problem Statement for **Single Container Pod**

> _"I want to run a single application/service in isolation, with clear separation of concerns and independent scaling."_

- When you deploy a **microservice** (e.g., Nginx web server, Redis cache, PostgreSQL DB, API service).
- The container has **everything it needs** inside itself (binary, runtime, dependencies).
- No need for tight coupling with other containers.
- Easier to scale (Kubernetes can horizontally scale the Pod).
- Useful when each service should **fail, scale, and update independently**.

Example Problems:

- Run a standalone **Nginx** server to serve static content.
- Run a **Redis** instance for caching in one Pod.
- Run a **Python API service** container that listens on a port.

---

## Problem Statement for **Multi-Container Pod**

> _"I want multiple tightly coupled processes that need to share storage/network and must run on the same lifecycle."_

- You need **helper/sidecar/adapter** containers to work together with a main container.
- Containers **share the same Pod network & storage volumes**, so they can communicate easily.
- All containers start, stop, and restart together (tied lifecycle).
- Great for **log shipping, proxying, or data transformations** alongside the main app.

Example Problems:

- A **web server + log shipper**: Nginx serving traffic, Fluentd shipping logs from shared volume.
- A **main app + sidecar**: Application container plus Envoy/Linkerd proxy for service mesh.
- A **data processor + helper**: A script container that transforms data before the main app consumes it.
- A **Git sync sidecar**: One container runs the web app, another sidecar keeps config files in sync from Git.

---

**Rule of Thumb**:

- If containers can run independently → **Single container per Pod**.
- If containers are **tightly coupled and must share resources** → **Multi-container Pod**.

---

# Init Containers

- Init Containers are special containers that run before the main application containers in a pod start.
- Init containers always run to completion.
- Each init container must complete successfully before the next one starts.



![[Pasted image 20250831173912.png]]

#### Init containers in use

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

- **_MySQL Initialization:_** _The init container uses a MySQL image to handle initialization tasks._
- **_Script Execution:_** _The_ `_command_` _field specifies the command to execute the_ `_init.sql_` _script within the MySQL image._
- **_Volume Mounting:_** _The_ `_volumeMounts_` _field is employed to mount a shared volume that contains the_ `_init.sql_` _script._
- **_Shared Volume Specification:_** _The_ `_volumes_` _field is used to define and specify the shared volume containing the initialization script._

---

## Real Use Cases

1. **Database readiness check**
* Init container waits until the DB is up before starting the app.
* Example: Run a curl or nc command to check DB port before app launches.

2. **Config/Secret injection**
* Init container downloads configs from Git, S3, or Vault, and puts them into a shared volume for the main app.

3. **Permission/Filesystem setup**
* Init container sets correct permissions on a volume (so the main app runs with non-root user safely).

4. **Dependency setup**
* Init container installs binaries or fetches dependencies needed by the app.

---

## Pros

* **Guarantees order** → Setup happens before app starts.
* **Lightweight** → Each init container does just one job, then exits.
* **Isolation** → Init logic is separate from the main app.
* **Flexibility** → Can run different images (bash, curl, alpine) just for init.

---

## Cons

* **Startup delay** → Pod won’t start until all init containers finish.
* **Debugging** → If init container fails, the Pod won’t start at all.
* **Not reusable for running tasks** → They only run once at startup, not continuously.

---

## Sidecar Containers:

- _Sidecar Containers are additional containers that run alongside the main application container within the same pod_.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: alpine:latest
          command: ['sh', '-c', 'while true; do echo "logging" >> /opt/logs.txt; sleep 1; done']
          volumeMounts:
            - name: data
              mountPath: /opt
      initContainers:
        - name: logshipper
          image: alpine:latest
          restartPolicy: Always
          command: ['sh', '-c', 'tail -F /opt/logs.txt']
          volumeMounts:
            - name: data
              mountPath: /opt
      volumes:
        - name: data
          emptyDir: {}
```

### Use Cases:

- **Logging and Monitoring:-**  A logging sidecar container collecting and forwarding logs to a centralized logging system.

- **Security Operations:-** A sidecar container handling encryption/decryption tasks to secure communication for the main application.

- **Data Synchronization:-** A sidecar container responsible for synchronizing data with an external service, ensuring consistency.
---
### Lifecycle init , main and sidecar Container :

- **Init Container** → Setup that runs before the application starts.
    
- **Main Container** → The main work of your application.
    
- **Sidecar Container** → A helper that supports your application.

 ![[Pasted image 20250831174025.png]]