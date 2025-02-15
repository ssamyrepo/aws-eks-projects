
### **1. Install Kubernetes Tools**
#### Install `kubectl` (Kubernetes CLI)
```sh
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

#### Install `kubeadm`, `kubelet`, and `kubectl` (if setting up a cluster manually)
```sh
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

---

### **2. Verify Kubernetes Cluster Access**
#### Check Cluster Information
```sh
kubectl cluster-info
```

#### Check Node Status
```sh
kubectl get nodes
```

---

### **3. Deploy a Simple Kubernetes Pod**
#### Create a Pod YAML File (`pod.yaml`)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mycontainer
    image: nginx
    ports:
    - containerPort: 80
```

#### Apply the Pod Configuration
```sh
kubectl apply -f pod.yaml
kubectl get pods
```

---

### **4. Deploy an Application Using Deployments**
#### Create a Deployment YAML File (`deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx
        ports:
        - containerPort: 80
```

#### Apply the Deployment
```sh
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get pods
```

---

### **5. Expose the Deployment as a Service**
#### Expose the Deployment as a NodePort Service
```sh
kubectl expose deployment myapp --type=NodePort --port=80
kubectl get services
```

#### Access the Application
- Get the public IP of the EC2 instance where the Kubernetes node is running.
- Use the NodePort assigned to the service (from `kubectl get services`) to access the application:
  ```sh
  curl http://<EC2_PUBLIC_IP>:<NODE_PORT>
  ```

---

### **6. Scaling and Auto-Scaling**
#### Scale the Deployment Manually
```sh
kubectl scale deployment myapp --replicas=5
kubectl get pods
```

#### Enable Auto-Scaling
```sh
kubectl autoscale deployment myapp --cpu-percent=50 --min=1 --max=10
kubectl get hpa
```

---

### **7. View Logs and Debugging**
#### View Pod Logs
```sh
kubectl logs mypod
```

#### Describe a Pod
```sh
kubectl describe pod mypod
```

#### Execute a Command Inside a Running Pod
```sh
kubectl exec -it mypod -- /bin/bash
```

---

### **8. Cleanup Resources**
#### Delete Pods, Deployments, and Services
```sh
kubectl delete pod mypod
kubectl delete deployment myapp
kubectl delete service myapp
```

---

### **9. Additional Steps for Clarity and Completeness**
#### Check Pod Status
```sh
kubectl get pods -o wide
```

#### Check Service Details
```sh
kubectl describe service myapp
```

#### Check Deployment Status
```sh
kubectl rollout status deployment myapp
```

#### Update Deployment (Rolling Update)
```sh
kubectl set image deployment/myapp myapp=nginx:1.19
kubectl rollout status deployment myapp
```

#### Rollback Deployment
```sh
kubectl rollout undo deployment myapp
kubectl rollout status deployment myapp
```

#### Check Resource Usage
```sh
kubectl top pods
kubectl top nodes
```

#### Delete All Resources in a Namespace
```sh
kubectl delete all --all -n <namespace>
```

---

### **10. Accessing the Cluster Remotely**
If you're managing the Kubernetes cluster from a remote machine:
1. Copy the `kubeconfig` file from the master node to your local machine:
   ```sh
   scp user@<MASTER_NODE_IP>:~/.kube/config ~/.kube/config
   ```
2. Set the `KUBECONFIG` environment variable:
   ```sh
   export KUBECONFIG=~/.kube/config
   ```
3. Verify access:
   ```sh
   kubectl get nodes
   ```

---

### **11. Persistent Storage (Optional)**
If you need persistent storage for your applications, you can use AWS EBS volumes with Kubernetes Persistent Volumes (PVs) and Persistent Volume Claims (PVCs).

#### Create a StorageClass for AWS EBS
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
volumeBindingMode: Immediate
```

#### Create a PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ebs-sc
  resources:
    requests:
      storage: 10Gi
```

#### Apply the StorageClass and PVC
```sh
kubectl apply -f storageclass.yaml
kubectl apply -f pvc.yaml
kubectl get pvc
```
