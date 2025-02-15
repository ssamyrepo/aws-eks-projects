
### **1. Install Kubernetes Tools**
#### Install `kubectl` (Kubernetes CLI)
```sh
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

#### Install Minikube (Local Kubernetes Cluster)
```sh
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube-linux-amd64
sudo mv minikube-linux-amd64 /usr/local/bin/minikube
minikube version
```

#### Start Minikube Cluster
```sh
minikube start
kubectl cluster-info
kubectl get nodes
```

---

### **2. Deploy a Simple Kubernetes Pod**
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

### **3. Deploy an Application Using Deployments**
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

### **4. Expose the Deployment as a Service**
```sh
kubectl expose deployment myapp --type=NodePort --port=80
kubectl get services
```

#### Get the Minikube Service URL
```sh
minikube service myapp --url
```

---

### **5. Scaling and Auto-Scaling**
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

### **6. View Logs and Debugging**
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

### **7. Cleanup Resources**
#### Delete Pods, Deployments, and Services
```sh
kubectl delete pod mypod
kubectl delete deployment myapp
kubectl delete service myapp
```

#### Stop Minikube
```sh
minikube stop
```

---

### **8. Additional Steps for Clarity and Completeness**
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

#### Check Minikube Status
```sh
minikube status
```

#### Delete Minikube Cluster
```sh
minikube delete
```
