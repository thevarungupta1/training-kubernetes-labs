# Lab 04: Understanding of Deployment

## **Objective**
By the end of this lab, you will have a functional Minikube installation running on a Windows machine, allowing you to deploy and manage Kubernetes clusters locally.

---

### **Deployment in Kubernetes**
A **Deployment** in Kubernetes is an API object that manages the lifecycle of application pods. It ensures that the desired number of pod replicas are running and can automatically roll out updates or roll back to previous versions when needed.

---

## **Key Features of a Deployment**
1. **Declarative Management**  
   - Instead of manually running and managing containers, you declare the desired state in a YAML or JSON file, and Kubernetes ensures that state is maintained.

2. **Rolling Updates**  
   - Deployments allow you to update your application with zero downtime by gradually replacing old pods with new ones.

3. **Rollback Mechanism**  
   - If an update fails, Kubernetes can automatically roll back to the previous stable version.

4. **Scaling**  
   - You can scale your application up or down easily by changing the `replicas` count in the Deployment spec.

5. **Self-healing**  
   - If a pod crashes or becomes unresponsive, Kubernetes will automatically create a new one.

---

## **Components of a Deployment**
A Kubernetes Deployment consists of:
1. **Pod Template**  
   - Defines how the container(s) should be created (image, ports, environment variables, etc.).
  
2. **ReplicaSet**  
   - Ensures the desired number of pod replicas are running.

3. **Strategy**  
   - Defines how updates should be rolled out (RollingUpdate or Recreate).

---

## **Example Deployment YAML**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3  # Number of pod replicas
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
        image: my-app:latest  # Docker image
        ports:
        - containerPort: 80
```

---

## **Deployment Strategies**
1. **Rolling Update (Default)**
   - Gradually replaces old pods with new ones to avoid downtime.
   - Example configuration:
   ```yaml
   strategy:
     type: RollingUpdate
     rollingUpdate:
       maxUnavailable: 1
       maxSurge: 1
   ```

2. **Recreate**
   - Deletes all existing pods before creating new ones.
   - Used when an application cannot run multiple versions simultaneously.

---

## **Managing a Deployment**
- **Create a Deployment:**
  ```sh
  kubectl apply -f deployment.yaml
  ```

- **View Deployments:**
  ```sh
  kubectl get deployments
  ```

- **Scale Deployment:**
  ```sh
  kubectl scale deployment my-app --replicas=5
  ```

- **Update Deployment:**
  ```sh
  kubectl set image deployment/my-app my-container=my-app:v2
  ```

- **Rollback Deployment:**
  ```sh
  kubectl rollout undo deployment my-app
  ```

---

## **Conclusion**
Deployments in Kubernetes simplify application management by providing automated rollouts, rollbacks, scaling, and self-healing capabilities. By defining the desired state in a YAML file, Kubernetes ensures that your application runs reliably with minimal manual intervention.

