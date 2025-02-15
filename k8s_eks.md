
### **1. Install Prerequisites**
#### Install `kubectl` (Kubernetes CLI)
```sh
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

#### Install `eksctl` (EKS CLI)
```sh
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

#### Install `awscli` (AWS CLI)
```sh
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

#### Configure AWS CLI
```sh
aws configure
```
- Provide your AWS Access Key, Secret Key, Region, and Output format.

---

### **2. Create an EKS Cluster**
#### Create an EKS Cluster Using `eksctl`
```sh
eksctl create cluster --name my-cluster --region us-west-2 --nodegroup-name my-nodes --node-type t3.medium --nodes 3
```
- Replace `my-cluster`, `us-west-2`, `my-nodes`, and `t3.medium` with your desired cluster name, region, node group name, and instance type.

#### Verify Cluster Creation
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
#### Expose the Deployment as a LoadBalancer Service
```sh
kubectl expose deployment myapp --type=LoadBalancer --port=80
kubectl get services
```

#### Access the Application
- Wait for the LoadBalancer to provision and get the external IP or DNS name:
  ```sh
  kubectl get services
  ```
- Access the application using the LoadBalancer's external DNS name or IP:
  ```sh
  curl http://<LOADBALANCER_DNS_NAME>
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

#### Delete the EKS Cluster
```sh
eksctl delete cluster --name my-cluster --region us-west-2
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

### **10. Persistent Storage (Optional)**
AWS EKS integrates with **Amazon EBS** and **Amazon EFS** for persistent storage.

#### Create a StorageClass for EBS
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

---

### **11. Ingress (Optional)**
To expose your application using an **Ingress** resource, you can use the **AWS Load Balancer Controller**.

#### Install the AWS Load Balancer Controller
Follow the [official guide](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html) to install the AWS Load Balancer Controller.

#### Create an Ingress Resource
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp
            port:
              number: 80
```

#### Apply the Ingress
```sh
kubectl apply -f ingress.yaml
kubectl get ingress
```
