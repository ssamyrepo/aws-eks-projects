Deploying the RESTful API on AWS EKS (Elastic Kubernetes Service) involves creating an EKS cluster, deploying the application using Kubernetes manifests, and integrating MongoDB Atlas as the database. Below are the step-by-step instructions:



 Step 1: Set Up an EKS Cluster
 
1. Install Prerequisites:
   - Install the AWS CLI:
  
     curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
     unzip awscliv2.zip
     sudo ./aws/install
     
   - Install `kubectl`:
  
     curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
     sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
     
   - Install `eksctl`:
  
     curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
     sudo mv /tmp/eksctl /usr/local/bin
     

2. Create an EKS Cluster:

   - Use `eksctl` to create a cluster:
  
     eksctl create cluster \
       --name rest-api-cluster \
       --region us-west-2 \
       --nodegroup-name rest-api-nodes \
       --node-type t2.micro \
       --nodes 2 \
       --nodes-min 1 \
       --nodes-max 3
     
   - This will create an EKS cluster with 2 worker nodes.

3. Verify the Cluster:

   - Update the kubeconfig:
  
     aws eks update-kubeconfig --name rest-api-cluster --region us-west-2
     
   - Check the nodes:
  
     kubectl get nodes
     

---

 Step 2: Set Up MongoDB
 
1. Use MongoDB Atlas:
   - Sign up for a free account at [MongoDB Atlas](https://www.mongodb.com/cloud/atlas).
   - Create a cluster and database.
   - Whitelist the EKS cluster's public IP range for access.
   - Get the connection string (replace `<username>`, `<password>`, and `<cluster-url>`):
     
     mongodb+srv://<username>:<password>@<cluster-url>/library?retryWrites=true&w=majority
     

---

Step 3: Prepare Kubernetes Manifests
 
1. Create a Namespace:
   - Create a namespace for the application:
  
     kubectl create namespace rest-api
     

2. Create a ConfigMap:

   - Save the following as `configmap.yaml`:
     yaml
     apiVersion: v1
     kind: ConfigMap
     metadata:
       name: rest-api-config
       namespace: rest-api
     data:
       db_host: "<cluster-url>"
       db_username: "<username>"
       db_name: "library"
     

3. Create a Secret:

   - Save the following as `secret.yaml`:
     yaml
     apiVersion: v1
     kind: Secret
     metadata:
       name: rest-api-secret
       namespace: rest-api
     type: Opaque
     stringData:
       db_password: "<password>"
     

4. Create a Deployment:

   - Save the following as `deployment.yaml`:
     yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: rest-api
       namespace: rest-api
     spec:
       replicas: 2
       selector:
         matchLabels:
           app: rest-api
       template:
         metadata:
           labels:
             app: rest-api
         spec:
           containers:
           - name: rest-api
             image: <your-docker-image>
             ports:
             - containerPort: 5000
             env:
             - name: DB_HOST
               valueFrom:
                 configMapKeyRef:
                   name: rest-api-config
                   key: db_host
             - name: DB_USERNAME
               valueFrom:
                 configMapKeyRef:
                   name: rest-api-config
                   key: db_username
             - name: DB_PASSWORD
               valueFrom:
                 secretKeyRef:
                   name: rest-api-secret
                   key: db_password
             - name: DB_NAME
               valueFrom:
                 configMapKeyRef:
                   name: rest-api-config
                   key: db_name
     

5. Create a Service:

   - Save the following as `service.yaml`:
     yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: rest-api
       namespace: rest-api
     spec:
       selector:
         app: rest-api
       ports:
       - port: 80
         targetPort: 5000
       type: LoadBalancer
     

Step 4: Deploy the Application
 
1. Apply the manifests:

   kubectl apply -f configmap.yaml
   kubectl apply -f secret.yaml
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yaml
   

2. Verify the deployment:

   kubectl get all -n rest-api
   

3. Get the LoadBalancer URL:

   kubectl get svc -n rest-api
   
   - Use the `EXTERNAL-IP` from the output to access the API.


Step 5: Test the API
 
1. Access the API:
   - Use the LoadBalancer URL to access the API:
     
     http://<loadbalancer-url>
     

2. Test Endpoints:
   - Use `curl` or Postman to test the API endpoints:
     - Home endpoint:
    
       curl http://<loadbalancer-url>
       
     - Create a book:
    
       curl -X POST http://<loadbalancer-url>/api/v1/books \
         -H "Content-Type: application/json" \
         -d '{"title": "Introduction to RESTful APIs", "author": "John Doe", "pages": 120}'
       
     - Get all books:
    
       curl http://<loadbalancer-url>/api/v1/books
       


Step 6: Scale the Application
 
1. Scale the deployment:

   kubectl scale deployment rest-api --replicas=3 -n rest-api
   

2. Verify scaling:

   kubectl get pods -n rest-api
   

Step 7: Clean Up
 
1. Delete the EKS cluster:

   eksctl delete cluster --name rest-api-cluster --region us-west-2
   

2. Delete the MongoDB Atlas cluster (if no longer needed).
