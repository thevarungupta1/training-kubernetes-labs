# Lab 05: Understanding of Services

## **Objective**
By the end of this lab, you will have a functional Minikube installation running on a Windows machine, allowing you to deploy and manage Kubernetes clusters locally.

---
### **Services in Kubernetes**
A **Service** in Kubernetes is an abstraction that provides network access to a set of pods. Since pods are ephemeral and can be replaced, their IP addresses can change. A Service ensures that communication with a group of pods remains stable, even as individual pod instances come and go.

---

## **Key Features of a Service**
1. **Stable Network Endpoint**  
   - Services provide a single IP address and DNS name to access a group of pods.
   
2. **Load Balancing**  
   - Distributes traffic across multiple pod replicas to ensure high availability.

3. **Service Discovery**  
   - Kubernetes automatically assigns a DNS name to each Service, making it easy for applications to find and communicate with each other.

4. **Decoupling**  
   - Services abstract the underlying pod instances, allowing pods to be replaced without breaking connectivity.

---

## **Types of Services**
Kubernetes supports different types of Services based on use cases:

1. **ClusterIP (Default)**
   - Exposes the Service inside the cluster only.
   - Used for internal communication between pods.
   - Example:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-service
   spec:
     selector:
       app: my-app
     ports:
       - protocol: TCP
         port: 80
         targetPort: 8080
   ```

2. **NodePort**
   - Exposes the Service on a static port on each node.
   - Accessible externally via `<NodeIP>:<NodePort>`.
   - Example:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-service
   spec:
     type: NodePort
     selector:
       app: my-app
     ports:
       - protocol: TCP
         port: 80
         targetPort: 8080
         nodePort: 30000
   ```

3. **LoadBalancer**
   - Uses a cloud provider’s external load balancer to route traffic to the service.
   - Used for internet-facing applications.
   - Example:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-service
   spec:
     type: LoadBalancer
     selector:
       app: my-app
     ports:
       - protocol: TCP
         port: 80
         targetPort: 8080
   ```

4. **ExternalName**
   - Maps a Kubernetes Service to an external DNS name.
   - Used for integrating with external services.
   - Example:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-service
   spec:
     type: ExternalName
     externalName: example.com
   ```

---

## **Managing Services**
- **List Services:**  
  ```sh
  kubectl get services
  ```
- **Describe a Service:**  
  ```sh
  kubectl describe service my-service
  ```
- **Delete a Service:**  
  ```sh
  kubectl delete service my-service
  ```

---

## **Conclusion**
Services in Kubernetes provide a stable way to expose and manage access to applications running in pods. Whether for internal communication, external access, or load balancing, Kubernetes Services ensure reliable networking for containerized applications.

Would you like any modifications or additions? 🚀