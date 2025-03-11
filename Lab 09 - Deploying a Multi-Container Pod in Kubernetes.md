# Lab 09: Deploying a Multi-Container Pod in Kubernetes

## Objective

- Deploy a pod that contains multiple containers.
- Understand how containers within the same pod share the network namespace and storage.
- Observe how sidecar containers can assist the main application container (e.g., by logging or data processing).

## Using Sidecar Containers in Kubernetes
A `sidecar container` runs alongside the main container in the same pod. It can share resources (like volumes, networking, and environment variables) with the main container to provide auxiliary functions such as:

- Log collection and processing.
- Proxying or service mesh functionalities.
- Dynamic configuration reloads.
- Monitoring.

## Prerequisites

- A running Kubernetes cluster (minikube, kind, or any cloud-based cluster).
- `kubectl` installed and configured.
- Basic knowledge of Kubernetes objects and YAML syntax.

---

## Part 1: Create the Multi-Container Pod YAML

In this exercise, the pod will consist of:
- **Main Container (NGINX):** Serves web content.
- **Sidecar Container (Busybox):** Periodically logs a custom message to a shared volume.

### 1.1 Create the YAML File

Create a file named `multi-container-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
  labels:
    app: multi-container-demo
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
      volumeMounts:
        - name: shared-logs
          mountPath: /usr/share/nginx/html/logs
    - name: sidecar-container
      image: busybox:latest
      command: ["sh", "-c"]
      args:
        - |
          while true; do
            echo "Sidecar log entry at $(date)" >> /logs/sidecar.log;
            sleep 10;
          done
      volumeMounts:
        - name: shared-logs
          mountPath: /logs
  volumes:
    - name: shared-logs
      emptyDir: {}
```

## Part 2: Deploy the Multi-Container Pod
## 2.1 Apply the YAML
Deploy the pod by running:

```bash
kubectl apply -f multi-container-pod.yaml
```
Verify that the pod is running:

```bash
kubectl get pods multi-container-pod
```


### 2.2 Inspect Pod Details
To check the configuration and ensure both containers are running, use:

```bash
kubectl describe pod multi-container-pod
```

## Part 3: Testing the Multi-Container Pod
### 3.1 Check Logs from the Sidecar Container
You can view logs from the sidecar container to confirm it’s writing log entries:

```bash
kubectl logs multi-container-pod -c sidecar-container
```


### 3.2 Access the NGINX Container
To verify that the NGINX container is serving content, you can port-forward the pod:

```bash
kubectl port-forward pod/multi-container-pod 8080:80
```
Then, open your browser and navigate to `http://localhost:8080`. You may not see the log file directly unless you modify the NGINX index file to list files in `/usr/share/nginx/html/logs`, but you can verify that the server is up and running.

### 3.3 Exec into the Pod
To inspect the shared volume, execute a shell in the pod and check the log file:

```bash
kubectl exec -it multi-container-pod -c sidecar-container -- sh
```
Inside the shell, run:

```sh
cat /logs/sidecar.log
```
You should see multiple timestamped log entries confirming that the sidecar container is writing to the shared volume.


## Part 4: Cleanup
After completing the lab, clean up the resources by deleting the pod:

```bash
kubectl delete -f multi-container-pod.yaml
```
