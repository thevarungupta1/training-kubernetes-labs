## Lab 02 - Kubernetes - Implementing fine-grained network policies and restricting traffic between applications

Welcome to the advanced concepts of Kubernetes. Kubernetes network policies enable you to define how pods communicate with each other and with other network endpoints. Implementing detailed network policies helps control traffic flow between applications and enhances the security of the Kubernetes environment.

**Pre-requisites**
- A running kubernetes cluster.
- kubectl installed and configured to interact with the kubernetes cluster.
- Helm installed for managing kubernetes applications.

_________________________________
### Task 1: In this task, you will deploy a sample application.
___________________________________

**Step 1: Create namespaces**

Use the following command to create namespaces.

```run
kubectl create namespace frontend
kubectl create namespace backend
```
**Result:**
The result will show that you have been able to create two namespaces.

--------------------------------------------------

**Step 2: Deploy two simple applications, frontend and backend**

Create a file named `frontend-deployment.yaml` to deploy the frontend application.

```
# frontend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
Create a file named `backend-deployment.yaml` to deploy the frontend application

```
# backend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: api
        image: hashicorp/http-echo
        args:
          - "-text=Hello from backend"
        ports:
        - containerPort: 5678
```
**Result:**
The result will show that you have been able to create a yaml file to deploy frontend and backend applications.

--------------------------------------------------

**Step 3: Apply the deployments**

Run the deploy command to apply the deployment of `frontend` and `backend` application using the yaml file.

```run
kubectl apply -f frontend-deployment.yaml
kubectl apply -f backend-deployment.yaml
```
**Result:**
The result will show that you have been able to apply the deployment.

--------------------------------------------------

**Step 4 Verify the deployments**

Run the following command to verify `frontend` and `backend` deployment.

```run
kubectl get deployment -n frontend
kubectl get deployment -n backend
```

**Result:**
The result will verify the deployments.

--------------------------------------------------

**Step 5: Expose both deployments as ClusterIP services within their namespaces**

Create a file named `frontend-service.yaml` to expose the frontend application.

```
# frontend-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: frontend
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
Create a file named `backend-service.yaml` to expose the backend application.

```
# backend-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: backend
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 5678
      targetPort: 5678
```

**Result:**
The result will show that you have been able to create a yaml file to expose both frontend and backend applications.

--------------------------------------------------

**Step 6: Apply both services**

You need to run the following command to create services.

```run
kubectl apply -f frontend-service.yaml
kubectl apply -f backend-service.yaml
```
**Result:**
The result will show that you have been able to create services.

--------------------------------------------------

**Step 7: Verify services**

You need to run the following command to verify that the services are running.

```run
kubectl get services -n frontend
kubectl get services -n backend
```

**Result:**
The result will show that you have been able to verify the newly created services.

--------------------------------------------------

## **Expected outcome:**
In this challenge, you have developed a deeper understanding of network policies in kubernetes, an essential skill for building secure applications.



_________________________________
### Task 2: In this task, you will learn to implement network policies.
___________________________________

**Step 1: Deny all traffic to the backend namespace by default**

You need to create a file named  `backend-deny-all.yaml` to create network policy which will deny all incoming traffic.

```
# backend-deny-all.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: backend
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress: []
```

**Result:**
The result will show that you have been able to create a yaml file to create a network policy to deny all incoming traffic to backend namespace.

--------------------------------------------------

**Step 2: Apply the network policy**

You need to run the following command to create a policy to block all incoming connection to any pod in the `backend` namespace.

```run
kubectl apply -f backend-deny-all.yaml
```

**Result:**
The result will show that you have applied the network policy.

--------------------------------------------------

**Step 3: Allow traffic only from the frontend namespace to the backend namespace**

You need to create a file named  `backend-policy.yaml` to create a network policy which will allow traffic from the `frontend` namespace to the `backend` namespace.

```
# backend-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: backend
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: frontend
    ports:
    - protocol: TCP
      port: 5678
```

**Result:**
The result will show that you have been able to allow traffic only from the frontend namespace to the backend namespace.

--------------------------------------------------

**Step 3: Apply the policy**

You need to run the following command to create a policy which allows only traffic from pods in the `frontend` namespace to access pods in the `backend` namespace.

```run
kubectl apply -f backend-policy.yaml
```

**Result:**
The result will show that you have been able to create a network policy to allow only traffic from pods in the `frontend` namespace.

--------------------------------------------------

**Step 4: Verify network policies**

You need to run the following command to verify network policies.

```run
kubectl get networkpolicy -n backend
```

**Result**
The result will show that you have been able to verify the network policy.

--------------------------------------------------

## **Expected outcome:**
In this challenge, you have developed a deeper understanding network policies in kubernetes, an essential skill for building secure applications.



_________________________________
### Task 3: In this task, you will learn to test network policies.
___________________________________


**Step 1: Verify the list of pods**

You need to run the following command to view the list of running pods.

```run
kubectl get pod -n frontend
kubectl get pod -n backend
```

**Result:**
The result will display a list of running pods.

--------------------------------------------------


**Step 2: Test the policy**

Run the following comamnd to create a pod inside the frontend namespace and using the curl command try to access the backend service.

- **Successful Test:**  You should able to access the backend service

```
# replace <frontend-pod-name> with actual fontend running pod
kubectl exec -n frontend <frontend-pod-name> -- curl backend.backend.svc.cluster.local:5678
```

Run the following command to create a pod in different namespace (e.g., default) and using the curl command try to access the backend service.

- **Failed Test:** Your access should be blocked.
```run
kubectl run -n default test --image=busybox --rm -it -- curl backend.backend.svc.cluster.local:5678
```

**Result:**
The result will show that access from the frontend to the backend is allowed, but access from a different namespace will be denied.

--------------------------------------------------

## **Expected outcome:**
In this challenge, you gained a deeper understanding of network policies in Kubernetes, an essential skill for building secure applications.



_________________________________
### Task 4: In this task, you will learn to monitor events related to network policies and applications that will help in troubleshooting.
___________________________________


**Step 1: View resource events**

Use the below mentioned command to view events related to your network policies and applications, which can help in troubleshooting

```run
kubectl get events
```
**Result:**
The result will show a list of events related to the network policies and applications.

--------------------------------------------------

**Step 2: Debugging network policies**

Run the following command to describe the network policy if you encounter issues with the network policies

```run
kubectl describe networkpolicy deny-all -n backend
kubectl describe networkpolicy allow-frontend -n backend
```

**Result:**
The result will display the details of the network policies.

--------------------------------------------------

**Step 3: Check logs of related components**

Run the following command to check the logs if the custom resource interacts with controllers or services.

```run
kubectl get pods -n backend
```

```
# write pod name which log you want to check
kubectl logs <pod-name>
```

**Result:**
The result will display the logs of pods.

--------------------------------------------------

## **Expected outcome:**
In this challenge, you gathered a deeper understanding of network policies in Kubernetes, an essential skill for building secure applications.



_________________________________
### Task 5: In this task, you will learn to clean up the resources.
___________________________________


**Step 1: Delete the deployments and services**

Run the following command to remove the deployed sample applications.
```run
kubectl delete -f frontend-deployment.yaml
kubectl delete -f backend-deployment.yaml
kubectl delete -f frontend-service.yaml
kubectl delete -f backend-service.yaml
```

**Result:**
The result will show that the deployments and services have been deleted.

--------------------------------------------------

**Step 2: Delete the network policies**

Run the following command to remove the network policies

```run
kubectl delete -f backend-deny-all.yaml
kubectl delete -f backend-policy.yaml
```
**Result:**
The result will show that the network polices have been deleted.

--------------------------------------------------


**Step 3: Delete the namespaces**

Run the following command to remove the namespaces.

```run
kubectl delete namespace frontend
kubectl delete namespace backend
```

**Result:**
The result will show that the namespaces have been deleted.

--------------------------------------------------

## **Expected outcome:**
In this challenge, you have gathered a deeper understanding of the network policies in Kubernetes, which is an essential skill for building secure applications.
