# Lab 06: Using ConfigMap and Secret in Kubernetes

## **Objective**
- Learn how to create a ConfigMap and a Secret using YAML manifests.
- Understand how to use these objects in a Kubernetes pod.
- Explore ways to update configurations without rebuilding container images.

### **Prerequisites**
- A running Kubernetes cluster (minikube, kind, or any cloud-based cluster).
- `kubectl` installed and configured.
- Basic knowledge of YAML and Kubernetes concepts.

### Part 1: Create and Apply a ConfigMap
### 1.1 Create a ConfigMap YAML
Create a file named `app-config.yaml` with the following content:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: "production"
  APP_DEBUG: "false"
  WELCOME_MESSAGE: "Welcome to our Kubernetes lab!"
```
This ConfigMap holds non-sensitive configuration data for the application.

### 1.2 Apply the ConfigMap
Run the following command to create the ConfigMap in your cluster:

```
kubectl apply -f app-config.yaml
```
Verify that the ConfigMap was created:

```
kubectl get configmap app-config -o yaml
```

### Part 2: Create and Apply a Secret
### 2.1 Create a Secret YAML
Create a file named `app-secret.yaml` with the following content. Note that Kubernetes Secrets require the data to be base64-encoded. For this lab, you can pre-encode values or let Kubernetes handle it if you use `kubectl create secret` command. For demonstration purposes, we’ll provide pre-encoded values for two keys:

**Tip: You can generate base64 encoded values using:**

```bash
echo -n 'your_value' | base64
```
Here’s an example YAML:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  DB_USERNAME: YWRtaW4=        # base64 for "admin"
  DB_PASSWORD: c2VjdXJlcGFzcw==  # base64 for "securepass"
```

### 2.2 Apply the Secret
Run the following command to create the Secret:

```bash
kubectl apply -f app-secret.yaml
```

Verify the Secret with:
```bash
kubectl get secret app-secret -o yaml
```

### Part 3: Create a Pod that Consumes the ConfigMap and Secret
### 3.1 Create a Pod YAML
Create a file named pod-consume.yaml with the following content. This pod will demonstrate two methods of consuming the data:

- Environment Variables: Pull in data from ConfigMap and Secret as environment variables.
- Volume Mounts: (Optional) Mount ConfigMap or Secret data as files inside the container.
For simplicity, this example uses environment variables:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-secret-demo
spec:
  containers:
    - name: demo-container
      image: busybox
      command: [ "sh", "-c", "echo 'App Mode: ' $APP_MODE; echo 'App Debug: ' $APP_DEBUG; echo 'Welcome: ' $WELCOME_MESSAGE; echo 'DB Username: ' $DB_USERNAME; echo 'DB Password: ' $DB_PASSWORD; sleep 3600" ]
      env:
        # From ConfigMap
        - name: APP_MODE
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_MODE
        - name: APP_DEBUG
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_DEBUG
        - name: WELCOME_MESSAGE
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: WELCOME_MESSAGE
        # From Secret
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_USERNAME
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_PASSWORD
```

### 3.2 Apply the Pod

Run the following command:

```bash
kubectl apply -f pod-consume.yaml
```

Check the pod status until it is in the Running state:

```bash
kubectl get pods config-secret-demo
```


### 3.3 Inspect the Pod Logs
Once the pod is running, inspect its logs to see the environment variables in action:

```bash
kubectl logs config-secret-demo
```
You should see output similar to:

```
App Mode: production
App Debug: false
Welcome: Welcome to our Kubernetes lab!
DB Username: admin
DB Password: securepass
```


### Part 4: Cleanup
After completing the lab, clean up the created resources:

```bash
kubectl delete -f pod-consume.yaml
kubectl delete -f app-config.yaml
kubectl delete -f app-secret.yaml
```