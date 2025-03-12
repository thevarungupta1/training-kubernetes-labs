# Lab 10: Deploying a Multi-Pod Application in Kubernetes

## Objective

- Deploy a multi-tier application in Kubernetes with multiple pods.
- Create separate Deployments and Services for the backend API and frontend web server.
- Test inter-pod communication and external access to the application.

## Prerequisites

- A running Kubernetes cluster (minikube, kind, or any cloud-based cluster).
- `kubectl` installed and configured.
- Basic knowledge of Kubernetes objects and YAML syntax.

---

## Part 1: Deploy the Backend API

### 1.1 Create the Backend Deployment

Create a file named `backend-deployment.yaml` with the following content. This deployment uses a simple API container (for example, a Python Flask app container or a simple HTTP echo container).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  labels:
    app: multiapp
    tier: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: multiapp
      tier: backend
  template:
    metadata:
      labels:
        app: multiapp
        tier: backend
    spec:
      containers:
      - name: backend
        image: hashicorp/http-echo
        args:
          - "-text=Hello from the backend"
        ports:
        - containerPort: 5678
```

### 1.2 Create the Backend Service
Create a file named `backend-service.yaml` with the following content to expose the backend pods within the cluster:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  labels:
    app: multiapp
    tier: backend
spec:
  selector:
    app: multiapp
    tier: backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5678
```

### 1.3 Deploy the Backend
Apply the backend resources:

```bash
kubectl apply -f backend-deployment.yaml
kubectl apply -f backend-service.yaml
```
Verify the backend deployment and service:

```bash
kubectl get deployments
kubectl get pods -l app=multiapp,tier=backend
kubectl get svc backend-service
```

## Part 2: Deploy the Frontend Application
### 2.1 Create the Frontend Deployment
Create a file named `frontend-deployment.yaml` with the following content. The frontend uses an image that makes HTTP requests to the backend service. For this lab, we’ll use an NGINX container with a custom index.html that includes a call to the backend service. (For a real-world scenario, you might deploy a more advanced app.)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: multiapp
    tier: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: multiapp
      tier: frontend
  template:
    metadata:
      labels:
        app: multiapp
        tier: frontend
    spec:
      containers:
      - name: frontend
        image: nginx:alpine
        ports:
        - containerPort: 80
        volumeMounts:
          - name: custom-index
            mountPath: /usr/share/nginx/html
      volumes:
      - name: custom-index
        configMap:
          name: frontend-index
```

### 2.2 Create a ConfigMap for the Frontend
Create a file named `frontend-configmap.yaml` with the following content. This ConfigMap contains a custom index.html that calls the backend API. Adjust the content as needed:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-index
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
      <title>Multi-Pod Application</title>
    </head>
    <body>
      <h1>Welcome to the Frontend</h1>
      <p>Fetching message from the backend...</p>
      <div id="backend-message"></div>
      <script>
        // Call the backend API via the internal service name
        fetch('http://backend-service')
          .then(response => response.text())
          .then(data => {
            document.getElementById('backend-message').innerText = data;
          })
          .catch(error => {
            document.getElementById('backend-message').innerText = 'Error: ' + error;
          });
      </script>
    </body>
    </html>
```

### 2.3 Create the Frontend Service
Create a file named `frontend-service.yaml` with the following content to expose the frontend pods outside the cluster:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  labels:
    app: multiapp
    tier: frontend
spec:
  type: NodePort  # Change to LoadBalancer if using a cloud provider
  selector:
    app: multiapp
    tier: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### 2.4 Deploy the Frontend
Apply the frontend resources:

```bash
kubectl apply -f frontend-configmap.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
```
Verify the frontend deployment and service:

```bash
kubectl get deployments
kubectl get pods -l app=multiapp,tier=frontend
kubectl get svc frontend-service
```


## Part 3: Testing the Multi-Pod Application
### 3.1 Access the Frontend
If you are using minikube, run the following command to open the frontend in your browser:

```bash
minikube service frontend-service
```
Alternatively, if using NodePort, get the node IP and port:


```bash
kubectl get svc frontend-service
```
Then, open your browser to `http://<node-ip>:<node-port>`.

### 3.2 Verify the Application
- You should see the frontend webpage displaying a welcome message.
- The page will fetch a message from the backend service ("Hello from the backend") and display it on the page.
- You can verify that multiple pods are running for both the backend and frontend by checking their replica sets.


### Part 4: Cleanup
After completing the lab, clean up the resources:

```bash
kubectl delete -f frontend-service.yaml
kubectl delete -f frontend-deployment.yaml
kubectl delete -f frontend-configmap.yaml
kubectl delete -f backend-service.yaml
kubectl delete -f backend-deployment.yaml
```