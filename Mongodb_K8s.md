

### **🔹 Corrections & Improvements:**
1. **Fixed MongoDB Secret Encoding**:  
   - The **username and password must be base64-encoded**. I added a command to properly encode them.
  
2. **Added Readiness and Liveness Probes** to MongoDB & MongoExpress:  
   - Helps Kubernetes **detect when a pod is unhealthy** and should be restarted.

3. **Explicitly Defined Volume for MongoDB Data**:  
   - Ensures **data persists** in case the pod restarts.

4. **Updated `kubectl exec` Command**:  
   - Used `mongosh` instead of `mongod` for **modern MongoDB versions**.

---

## **✅ Steps to Install MongoDB and MongoExpress in Kubernetes**
---

### **🔹 1. Prerequisites**
- **Running Kubernetes cluster** (`minikube` or a real cluster).
- **`kubectl` installed** and connected to the cluster.
- **Docker images for MongoDB (`mongo`) and MongoExpress (`mongo-express`)** are available.

---

## **🔹 2. Create Kubernetes Resources**

### **Step 1: Create a Namespace**
```bash
kubectl create namespace deploy
```

---

### **Step 2: Create a ConfigMap for MongoDB**
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-config
  namespace: deploy
data:
  database_url: mongodb-service
```
**Apply the ConfigMap:**
```bash
kubectl apply -f configmap.yaml
```

---

### **Step 3: Create a Secret for MongoDB Credentials**
🔹 **Base64 Encode Username & Password**  
Ensure you **base64-encode** your credentials before storing them in the secret:

```bash
echo -n 'admin' | base64
echo -n 'password123' | base64
```
Copy the **base64-encoded values** into `secret.yaml`:

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
  namespace: deploy
type: Opaque
data:
  root_username: YWRtaW4=    # "admin" encoded
  root_password: cGFzc3dvcmQxMjM=  # "password123" encoded
```

**Apply the Secret:**
```bash
kubectl apply -f secret.yaml
```

---

### **Step 4: Deploy MongoDB**
```yaml
# mongodb-deployment.yaml
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
        livenessProbe:
          exec:
            command: ["mongosh", "--eval", "db.adminCommand('ping')"]
          initialDelaySeconds: 10
          periodSeconds: 5
        readinessProbe:
          exec:
            command: ["mongosh", "--eval", "db.adminCommand('ping')"]
          initialDelaySeconds: 5
          periodSeconds: 5
        volumeMounts:
        - name: mongodb-data
          mountPath: /data/db
      volumes:
      - name: mongodb-data
        emptyDir: {}
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

**Apply the MongoDB Deployment:**
```bash
kubectl apply -f mongodb-deployment.yaml
```

---

### **Step 5: Deploy MongoExpress**
```yaml
# mongoexpress-deployment.yaml
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
        livenessProbe:
          httpGet:
            path: /
            port: 8081
          initialDelaySeconds: 10
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /
            port: 8081
          initialDelaySeconds: 5
          periodSeconds: 5
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

**Apply the MongoExpress Deployment:**
```bash
kubectl apply -f mongoexpress-deployment.yaml
```

---

## **🔹 3. Verify the Deployment**

### **Step 1: Check Pods & Services**
```bash
kubectl get pods -n deploy
kubectl get services -n deploy
```

### **Step 2: Access MongoExpress**
```bash
kubectl get svc mongoexpress-service -n deploy
```
Find the **EXTERNAL-IP** and access it in your browser:
```
http://<EXTERNAL-IP>:8081
```

---

## **🔹 4. Interact with MongoDB**

### **Step 1: Access MongoDB Shell**
Get the MongoDB pod name:
```bash
kubectl get pods -n deploy
```
Exec into the MongoDB pod:
```bash
kubectl exec -it <mongodb-pod-name> -n deploy -- mongosh -u admin -p password123
```

### **Step 2: Create a Database and Collection**
```javascript
use mydatabase
db.createCollection("users")
db.users.insertOne({ name: "Stanley", age: 30 })
```

### **Step 3: Verify in MongoExpress**
Go to MongoExpress UI and check the **"mydatabase"** collection.

---

## **🔹 5. Clean Up Resources**
If you want to delete everything:
```bash
kubectl delete namespace deploy
```

---

## **✅ Summary of Fixes & Improvements**
1. **Fixed Secret Encoding** – Properly base64-encoded username/password.
2. **Added Readiness & Liveness Probes** – Helps Kubernetes restart unhealthy pods.
3. **Ensured MongoDB Data Persists** – Added a volume mount.
4. **Updated MongoDB Exec Command** – Used `mongosh` instead of `mongo`.
5. **Ensured Security Best Practices** – Used secrets for sensitive credentials.

---
