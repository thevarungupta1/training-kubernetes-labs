# Lab 08: Deploying a Stateful Database Using PVCs in Kubernetes

## Objective

- Deploy a stateful MySQL database in Kubernetes using a StatefulSet.
- Use Persistent Volume Claims (PVCs) to ensure data persists across pod restarts.
- Understand the role of a headless service in managing stateful applications.

## Prerequisites

- A running Kubernetes cluster (minikube, kind, or any cloud-based cluster).
- `kubectl` installed and configured.
- Basic knowledge of Kubernetes objects (Service, StatefulSet, PVC, etc.).
- Familiarity with YAML syntax.

---


## Part 1: Create Persistent Volume
### 1.1: Create a file named `mysql-pv.yaml` to define a persistent volume.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-data
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: "/data/db"
```


Apply the persistent volume
```bash
kubectl apply -f mysql-pv.yaml
```

### 1.2: Create a Persistent Volume Claim
Create a file named `mysql-pvc.yaml` to define a persistent volume claim.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi

```
Apply the persistent volume claim
```bash
kubectl apply -f mysql-pvc.yaml
```

Verify the persistent volume claim creation Check if the persistent volume claim is bound to the persistent volume
```bash
kubectl get pv
kubectl get pvc
```


## Part 2: Create a Headless Service

A headless service is required to manage the network identity for the pods in the StatefulSet. Create a file named `mysql-headless-svc.yaml` with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  ports:
    - port: 3306
      name: mysql
  clusterIP: None  # Marks this service as headless
  selector:
    app: mysql
```

Apply the service using:

```bash
kubectl apply -f mysql-headless-svc.yaml
```

Verify the service creation:

```bash
kubectl get svc mysql
```

### Part 3: Create a StatefulSet for MySQL
The StatefulSet will manage MySQL pods and automatically provision a PVC for each replica using `volumeClaimTemplates`.

Create a file named `mysql-statefulset.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  serviceName: "mysql"
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      terminationGracePeriodSeconds: 10
      containers:
        - name: mysql
          image: mysql:5.7
          ports:
            - containerPort: 3306
              name: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "my-secret-pw"
            - name: MYSQL_DATABASE
              value: "exampledb"
          volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql
  volumeClaimTemplates:
    - metadata:
        name: mysql-persistent-storage
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 1Gi
        storageClassName: ""  # Use default storage class or specify one as needed
```
Apply the StatefulSet using:

```bash
kubectl apply -f mysql-statefulset.yaml
```
Check the status of the StatefulSet and its pods:

```bash
kubectl get statefulset mysql
kubectl get pods -l app=mysql
```

Also, verify the PVCs created for your StatefulSet:

```bash
kubectl get pvc
```

##Part 4: Testing the Deployment
### 4.1 Connect to the MySQL Pod
Once the pod is running, you can connect to the MySQL instance to verify data persistence. For example, start a bash session in the MySQL pod:

```bash
kubectl exec -it mysql-0 -- bash
```

Then, log into MySQL:

```bash
mysql -uroot -p
```

Enter the password (my-secret-pw). Once logged in, you can create a table or insert some test data:

```sql
USE exampledb;
CREATE TABLE test (id INT PRIMARY KEY, data VARCHAR(50));
INSERT INTO test VALUES (1, 'Hello Kubernetes!');
SELECT * FROM test;
```
Exit the MySQL shell and then the pod's shell.

### 4.2 Verify Data Persistence
To test persistence, delete the MySQL pod and let the StatefulSet recreate it:

```bash
kubectl delete pod mysql-0
```
Wait for the pod to be recreated:

```bash
kubectl get pods -l app=mysql
```

Connect to the new pod instance and check that your test data still exists:

```bash
kubectl exec -it mysql-0 -- bash
mysql -uroot -p
```
Run:

```sql
Copy
USE exampledb;
SELECT * FROM test;
```

Your test data should be intact, confirming that the PVC preserved the data.

### Part 5: Cleanup
When you have finished the lab, clean up the resources:

```bash
kubectl delete -f mysql-statefulset.yaml
kubectl delete -f mysql-headless-svc.yaml
```

Depending on your cluster configuration, you may also want to delete the PVCs manually if they aren’t automatically reclaimed:

```bash
kubectl get pvc
kubectl delete pvc <pvc-name>
```