# Lab 07: Persisting Data in Kubernetes Pods

## Objective

- Learn how to provision persistent storage in Kubernetes.
- Create a Persistent Volume (PV) and a Persistent Volume Claim (PVC).
- Mount the persistent storage in a pod to persist application data.
- Understand how data persists even if pods are recreated.

## Prerequisites

- A running Kubernetes cluster (minikube, kind, or any cloud-based cluster).
- `kubectl` installed and configured.
- Basic knowledge of Kubernetes objects and YAML syntax.

---

## Part 1: Create a Persistent Volume (PV)

### 1.1 Create a PV YAML

Create a file named `pv.yaml` with the following content. This example uses a hostPath volume for demonstration. In production, use cloud provider or network storage solutions.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: demo-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /tmp/demo-pv  # This path exists on the node
```

## Part 2: Create a Persistent Volume Claim (PVC)
### 2.1 Create a PVC YAML

Create a file named `pvc.yaml` with the following content:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### 2.2 Apply the PVC
Run the following command to create the Persistent Volume Claim:

```bash
kubectl apply -f pvc.yaml
```
Verify the PVC creation:


```bash
kubectl get pvc demo-pvc
```


##Part 3: Create a Pod That Uses the PVC
### 3.1 Create a Pod YAML
Create a file named `pod-persist.yaml` with the following content. This pod mounts the PVC as a volume and writes data to a file. Later, you can delete and re-create the pod to see that the data persists.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: persistent-pod
spec:
  containers:
    - name: demo-container
      image: busybox
      command:
        - sh
        - -c
        - |
          # Write data to a file in the mounted volume
          echo "Persisted Data: $(date)" > /mnt/data/persistent.txt
          # Keep the container running
          sleep 3600
      volumeMounts:
        - name: demo-storage
          mountPath: /mnt/data
  volumes:
    - name: demo-storage
      persistentVolumeClaim:
        claimName: demo-pvc
```

### 3.2 Apply the Pod
Deploy the pod with:

```bash
kubectl apply -f pod-persist.yaml
```
Check the pod status:

```bash
kubectl get pods persistent-pod
```

### 3.3 Verify Data Persistence
After the pod is running, execute a shell in the pod to verify the persisted file:

```bash
kubectl exec -it persistent-pod -- sh
```
Inside the pod, run:

```sh
cat /mnt/data/persistent.txt
```
You should see the timestamp and data written earlier.

### Part 4: Test Data Persistence Across Pod Restarts
**1. Delete the Pod**
Delete the pod while keeping the PVC and PV intact:

```bash
kubectl delete -f pod-persist.yaml
```

**2. Recreate the Pod**
Reapply the pod manifest:

```bash
kubectl apply -f pod-persist.yaml
```

**3.Verify the Data Again**
Once the pod is running, access it and check `/mnt/data/persistent.txt` again to confirm the file still exists.

### Part 5: Cleanup
When you have finished the lab, clean up all resources:

```bash
kubectl delete -f pod-persist.yaml
kubectl delete -f pvc.yaml
kubectl delete -f pv.yaml
```