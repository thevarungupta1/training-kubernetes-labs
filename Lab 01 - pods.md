# Lab 01: Understanding of PODs

## **Objective**
By the end of this lab, you will have a functional Minikube installation running on a Windows machine, allowing you to deploy and manage Kubernetes clusters locally.

---

## **Introduction**
In **Kubernetes**, a **Pod** is the smallest and most basic deployable unit. It represents a single instance of a running process in the cluster. A **Pod** can contain one or more **containers** that share the same network and storage resources.

---


### Key Features of Pods

1. **Multiple Containers in a Pod**
    - A Pod can run multiple tightly coupled containers. These containers share:
        - The same **network namespace** (same IP, ports).
        - **Shared storage volumes**, allowing them to communicate via shared files.

    - Example: A web application and a logging sidecar running in the same Pod.

2. **Lifecycle & Scheduling**
    - Kubernetes schedules Pods on nodes.
    - If a Pod crashes, Kubernetes may restart it or create a new one.
    - Pods are **ephemeral**; they can be replaced by new ones.

3. **Networking**
    - Each Pod gets a **unique IP address** within the cluster.
    - Containers inside the same Pod communicate via `localhost`, but different Pods communicate using **services**.

4. **Storage (Volumes)**
    - Pods can define **volumes** to share storage between containers.
    - **Persistent storage** (like **Persistent Volumes**) allows data to survive Pod restarts.

---

### Pod Types:
1. **Single-Container Pod:**
    - Runs a single container (most common use case).

2. **Multi-Container Pod:**
    - Runs multiple containers that work together (e.g., an app container and a logging sidecar).

3. **Static Pods:**
    - Directly managed by the Kubelet, not through the API server.

4. **Init Containers:**
    - Special containers that run before the main container starts.

---

## **Pod Components**
A Pod consists of the following main components:
1. **Containers**: The applications running inside the pod.
2. **Storage (Volumes)**: Persistent storage shared between containers in the pod.
3. **Networking**: Each pod has a unique IP address within the cluster.
4. **Metadata**: Information such as labels and annotations for identification and management.

---


### How to Create a Pod in Kubernetes
You can create a Pod using a YAML file or directly via the Kubernetes CLI.

**1. Using YAML Definition File**
Create a YAML file (e.g., `pod-example.yaml`):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
    - name: my-container
      image: nginx
      ports:
        - containerPort: 80

```

To deploy this Pod, use the command:
```sh
kubectl apply -f pod-example.yaml
```

To verify:
```sh
kubectl get pods
```

**2. Using the kubectl Command Directly**
You can also create a Pod without a YAML file:

```sh
kubectl run my-pod --image=nginx --port=80
```
Check the Pod status:

```sh
kubectl get pods
```

---

## **Managing Pods**
- **List all Pods:**
  ```sh
  kubectl get pods
  ```
- **Describe a Pod:**
  ```sh
  kubectl describe pod my-pod
  ```
- **Delete a Pod:**
  ```sh
  kubectl delete pod my-pod
  ```

---

## **Pod Lifecycle**
A pod goes through various phases:
1. **Pending**: Waiting for resources to be allocated.
2. **Running**: The pod is successfully scheduled and containers are running.
3. **Succeeded**: All containers have successfully terminated.
4. **Failed**: One or more containers terminated with an error.
5. **Unknown**: The state of the pod cannot be determined.

---

### Best Practices
- Use Deployments or ReplicaSets instead of standalone Pods for better management.
- Use Labels and Selectors to organize and manage your Pods effectively.
- Set Resource Requests and Limits to optimize resource allocation.
---

## **Conclusion**
Pods are fundamental to Kubernetes and serve as the building blocks for deploying and managing containerized applications. Understanding pods is essential for working efficiently with Kubernetes.
