# Lab 03: Understanding of ReplicaSet

## **Objective**
By the end of this lab, you will have a functional Minikube installation running on a Windows machine, allowing you to deploy and manage Kubernetes clusters locally.

---

### **ReplicaSet and ReplicationController in Kubernetes**  

In Kubernetes, **ReplicaSet** and **ReplicationController** are controllers that ensure a specified number of pod replicas are running at all times. They help maintain the desired state of applications by replacing failed or terminated pods.  

---

## **1. ReplicationController** (Deprecated)  
A **ReplicationController** ensures that a specific number of pod replicas are running at all times. If a pod fails or is terminated, it creates a new pod to maintain the defined number of replicas.  

### **Key Features:**  
- Ensures a specific number of identical pod replicas are running.  
- Supports self-healing by replacing failed pods.  
- Performs rolling updates by updating pods gradually.  

### **ReplicationController Example:**  
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-app-rc
spec:
  replicas: 3
  selector:
    app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: my-app:latest
        ports:
        - containerPort: 80
```

### **Why is ReplicationController Deprecated?**  
ReplicationController has been largely replaced by **ReplicaSet** because ReplicaSet provides better features and efficiency. ReplicaSet is part of the **apps/v1** API, whereas ReplicationController belongs to the older **v1** API.

---

## **2. ReplicaSet (Recommended Controller)**  
A **ReplicaSet** is an improved version of ReplicationController and is used to ensure the desired number of pod replicas are running. It introduces label selectors with set-based expressions for more flexible pod selection.  

### **Key Features:**  
- Ensures a specified number of pods are running at all times.  
- Uses **set-based label selectors** for more flexibility.  
- Can be used by a **Deployment** for rolling updates and rollbacks.  

### **ReplicaSet Example:**  
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-app-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: my-app:latest
        ports:
        - containerPort: 80
```

### **Differences Between ReplicaSet and ReplicationController:**  

| Feature             | ReplicationController | ReplicaSet |
|---------------------|----------------------|------------|
| API Version        | v1                    | apps/v1    |
| Selector Type      | Equality-based only   | Supports both equality-based and set-based selectors |
| Usage Recommendation | Deprecated, use ReplicaSet instead | Recommended for managing pods directly |
| Rolling Updates   | Not supported         | Used by Deployments for rolling updates |

---

## **3. Managing ReplicaSets**  

- **Create a ReplicaSet:**  
  ```sh
  kubectl apply -f replicaset.yaml
  ```

- **View all ReplicaSets:**  
  ```sh
  kubectl get replicasets
  ```

- **Scale a ReplicaSet:**  
  ```sh
  kubectl scale --replicas=5 rs/my-app-rs
  ```

- **Delete a ReplicaSet:**  
  ```sh
  kubectl delete replicaset my-app-rs
  ```

---

## **Conclusion**  
- **ReplicationController** was an earlier method to maintain pod replicas but is now **deprecated**.  
- **ReplicaSet** is the improved version and is widely used today.  
- In most cases, **Deployments** are preferred over ReplicaSets because they provide additional features like rolling updates and rollbacks.  


