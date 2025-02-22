# Lab 01: Installation of Kubernetes using Minikube

## **Objective**
By the end of this lab, you will have a functional Minikube installation running on a Windows machine, allowing you to deploy and manage Kubernetes clusters locally.

---

## **Prerequisites**
1. **Windows 10/11** (64-bit)
2. **Hardware Requirements:**
   - Minimum 2 CPU cores
   - 4GB RAM (8GB recommended)
   - 20GB free disk space
3. **Enable Virtualization:**
   - Check if virtualization is enabled in the BIOS.
   - You can verify by running this command in **Command Prompt**:
     ```powershell
     systeminfo | find "Virtualization"
     ```

---

## **Step 1: Install Prerequisites**
### **1.1 Install Windows Package Manager (winget)**
- Winget is pre-installed in Windows 10 (2004+) and Windows 11.
- If not installed, get it from the Microsoft Store.

### **1.2 Install a Hypervisor**
Minikube supports the following drivers on Windows:
- **Hyper-V** (Available in Windows Pro, Enterprise, Education)
- **VirtualBox** (Recommended for Windows Home)

#### **Enable Hyper-V (For Windows Pro/Enterprise)**
1. Open PowerShell as Administrator and run:
   ```powershell
   dism.exe /Online /Enable-Feature:Microsoft-Hyper-V /All
   ```
2. Restart your system.

#### **Install VirtualBox (For Windows Home)**
1. Download VirtualBox from: [https://www.virtualbox.org](https://www.virtualbox.org)
2. Install it with default settings.

---

## **Step 2: Install Minikube**
### **2.1 Using Winget**
Run the following command in **PowerShell**:
```powershell
winget install minikube
```

### **2.2 Using Installer (Alternative)**
1. Download Minikube: [https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)
2. Install and add it to the system **PATH**.

### **Verify Installation**
Run:
```powershell
minikube version
```
It should display the installed version.

---

## **Step 3: Install kubectl (Kubernetes CLI)**
### **3.1 Using Winget**
```powershell
winget install kubectl
```
### **3.2 Verify kubectl Installation**
```powershell
kubectl version --client
```
It should display the **client version**.

---

## **Step 4: Start Minikube Cluster**
### **4.1 Start Minikube**

- If using **Docker**:
  ```powershell
  minikube start --driver=docker
  ```

- If using **Hyper-V**:
  ```powershell
  minikube start --driver=hyperv
  ```
- If using **VirtualBox**:
  ```powershell
  minikube start --driver=virtualbox
  ```

### **4.2 Verify Cluster Status**
```powershell
minikube status
```

---

## **Step 5: Deploy and Test Kubernetes**
### **5.1 Deploy a Sample Application**
```powershell
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4
```
### **5.2 Expose the Application**
```powershell
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```

### **5.3 Get Service Details**
```powershell
kubectl get services
```
Note the **NodePort** assigned.

### **5.4 Access the Application**
Run:
```powershell
minikube service hello-minikube
```
This will open the application in your browser.

---

## **Step 6: Stop and Delete Minikube (Optional)**
- To stop Minikube:
  ```powershell
  minikube stop
  ```
- To delete the Minikube cluster:
  ```powershell
  minikube delete
  ```

---

## **Conclusion**
You have successfully installed Minikube on Windows and deployed a Kubernetes application. You can now experiment with Kubernetes locally!
