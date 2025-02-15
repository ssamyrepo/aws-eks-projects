
---

### **Steps to Install MongoDB and MongoExpress in Kubernetes**

#### **1. Prerequisites**
- Ensure you have a running Kubernetes cluster.
- Install `kubectl` and configure it to access your cluster.
- Docker images for MongoDB and MongoExpress are available on Docker Hub.

---

#### **2. Create Kubernetes Resources**

##### **Step 1: Create a Namespace**
Create a namespace to isolate your resources:
```bash
kubectl create namespace deploy
```

##### **Step 2: Create a ConfigMap**
Create a `configmap.yaml` file to store non-sensitive data (e.g., MongoDB connection URL):
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-config
  namespace: deploy
data:
  database_url: mongodb-service
```

Apply the ConfigMap:
```bash
kubectl apply -f configmap.yaml
```

##### **Step 3: Create a Secret**
Create a `secret.yaml` file to store sensitive data (e.g., MongoDB username and password):
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
  namespace: deploy
type: Opaque
data:
  root_username: dXNlcm5hbWU=  # base64 encoded "username"
  root_password: cGFzc3dvcmQ=  # base64 encoded "password"
```

Apply the Secret:
```bash
kubectl apply -f secret.yaml
```

##### **Step 4: Deploy MongoDB**
Create a `mongodb-deployment.yaml` file for MongoDB deployment and service:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb
  namespace: deploy
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: root_username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: root_password
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
  namespace: deploy
spec:
  selector:
    app: mongodb
  ports:
  - port: 27017
    targetPort: 27017
  type: ClusterIP
```

Apply the MongoDB deployment and service:
```bash
kubectl apply -f mongodb-deployment.yaml
```

##### **Step 5: Deploy MongoExpress**
Create a `mongoexpress-deployment.yaml` file for MongoExpress deployment and service:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongoexpress
  namespace: deploy
  labels:
    app: mongoexpress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongoexpress
  template:
    metadata:
      labels:
        app: mongoexpress
    spec:
      containers:
      - name: mongoexpress
        image: mongo-express
        ports:
        - containerPort: 8081
        env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: root_username
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: root_password
        - name: ME_CONFIG_MONGODB_SERVER
          valueFrom:
            configMapKeyRef:
              name: mongodb-config
              key: database_url
---
apiVersion: v1
kind: Service
metadata:
  name: mongoexpress-service
  namespace: deploy
spec:
  selector:
    app: mongoexpress
  ports:
  - port: 8081
    targetPort: 8081
  type: LoadBalancer
```

Apply the MongoExpress deployment and service:
```bash
kubectl apply -f mongoexpress-deployment.yaml
```

---

#### **3. Verify the Deployment**

##### **Step 1: Check Pods and Services**
Verify that the pods and services are running:
```bash
kubectl get pods -n deploy
kubectl get services -n deploy
```

##### **Step 2: Access MongoExpress**
Get the external IP of the MongoExpress service:
```bash
kubectl get svc mongoexpress-service -n deploy
```
Open a browser and navigate to `http://<EXTERNAL_IP>:8081`.

---

#### **4. Interact with MongoDB**

##### **Step 1: Access MongoDB Shell**
Exec into the MongoDB pod:
```bash
kubectl exec -it <mongodb-pod-name> -n deploy -- mongosh -u username -p password
```

##### **Step 2: Create a Database and Collection**
Inside the MongoDB shell:
```bash
use mongodemo
db.createCollection("demo")
db.demo.insertOne({ age: 24, height: 160 })
```

##### **Step 3: Verify Data in MongoExpress**
Refresh the MongoExpress UI to see the newly created database and collection.

---

#### **5. Clean Up**
Delete all resources in the namespace:
```bash
kubectl delete namespace deploy
```

---

### **Summary**
1. Created a namespace (`deploy`).
2. Created a ConfigMap and Secret for MongoDB configuration.
3. Deployed MongoDB and MongoExpress using Kubernetes manifests.
4. Accessed MongoExpress via the external IP.
5. Interacted with MongoDB using the shell and verified data in MongoExpress.
6. Cleaned up resources.

This process ensures MongoDB and MongoExpress are deployed and accessible in a Kubernetes cluster.
