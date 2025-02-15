### **1. Install Prerequisites**
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

### **2. Create an ECS Cluster**
#### Create an ECS Cluster
```sh
aws ecs create-cluster --cluster-name my-cluster
```

#### Verify Cluster Creation
```sh
aws ecs describe-clusters --cluster my-cluster
```

---

### **3. Create an ECS Task Definition**
#### Create a Task Definition JSON File (`task-definition.json`)
```json
{
  "family": "my-task",
  "networkMode": "awsvpc",
  "containerDefinitions": [
    {
      "name": "my-container",
      "image": "nginx",
      "cpu": 256,
      "memory": 512,
      "essential": true,
      "portMappings": [
        {
          "containerPort": 80,
          "hostPort": 80
        }
      ]
    }
  ],
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512"
}
```

#### Register the Task Definition
```sh
aws ecs register-task-definition --cli-input-json file://task-definition.json
```

#### Verify Task Definition
```sh
aws ecs describe-task-definition --task-definition my-task
```

---

### **4. Run the ECS Task**
#### Run the Task
```sh
aws ecs run-task --cluster my-cluster --task-definition my-task --launch-type FARGATE --network-configuration "awsvpcConfiguration={subnets=[subnet-xxxxxxx],securityGroups=[sg-xxxxxxx],assignPublicIp=ENABLED}"
```
- Replace `subnet-xxxxxxx` and `sg-xxxxxxx` with your subnet and security group IDs.

#### Verify Task Status
```sh
aws ecs describe-tasks --cluster my-cluster --tasks <task-id>
```

---

### **5. Create an ECS Service**
#### Create a Service
```sh
aws ecs create-service --cluster my-cluster --service-name my-service --task-definition my-task --desired-count 2 --launch-type FARGATE --network-configuration "awsvpcConfiguration={subnets=[subnet-xxxxxxx],securityGroups=[sg-xxxxxxx],assignPublicIp=ENABLED}"
```

#### Verify Service Status
```sh
aws ecs describe-services --cluster my-cluster --services my-service
```

---

### **6. Expose the Service**
#### Create a Load Balancer
```sh
aws elbv2 create-load-balancer --name my-lb --subnets subnet-xxxxxxx --security-groups sg-xxxxxxx
```

#### Create a Target Group
```sh
aws elbv2 create-target-group --name my-tg --protocol HTTP --port 80 --vpc-id vpc-xxxxxxx
```

#### Register Targets
```sh
aws elbv2 register-targets --target-group-arn <target-group-arn> --targets Id=<instance-id>,Port=80
```

#### Create a Listener
```sh
aws elbv2 create-listener --load-balancer-arn <load-balancer-arn> --protocol HTTP --port 80 --default-actions Type=forward,TargetGroupArn=<target-group-arn>
```

#### Access the Application
- Use the DNS name of the Load Balancer to access your application:
  ```sh
  curl http://<LOAD_BALANCER_DNS>
  ```

---

### **7. Scaling and Auto-Scaling**
#### Update Service Desired Count
```sh
aws ecs update-service --cluster my-cluster --service my-service --desired-count 5
```

#### Enable Auto-Scaling
- Create an Auto Scaling policy for your ECS service using the AWS Management Console or CLI.

---

### **8. View Logs and Debugging**
#### View Logs in CloudWatch
- ECS tasks automatically send logs to Amazon CloudWatch. Use the CloudWatch console to view logs.

#### Describe Tasks
```sh
aws ecs describe-tasks --cluster my-cluster --tasks <task-id>
```

#### Execute a Command Inside a Running Container
- Use AWS Systems Manager Session Manager to connect to the container.

---

### **9. Cleanup Resources**
#### Delete Service
```sh
aws ecs delete-service --cluster my-cluster --service my-service --force
```

#### Deregister Task Definition
```sh
aws ecs deregister-task-definition --task-definition my-task
```

#### Delete Cluster
```sh
aws ecs delete-cluster --cluster my-cluster
```

#### Delete Load Balancer
```sh
aws elbv2 delete-load-balancer --load-balancer-arn <load-balancer-arn>
```

---

### **10. Additional Steps for Clarity and Completeness**
#### Check Service Status
```sh
aws ecs describe-services --cluster my-cluster --services my-service
```

#### Update Service
```sh
aws ecs update-service --cluster my-cluster --service my-service --task-definition my-task
```

#### Check Task Status
```sh
aws ecs describe-tasks --cluster my-cluster --tasks <task-id>
```

---

### **11. Persistent Storage (Optional)**
ECS supports persistent storage using **Amazon EFS** (Elastic File System).

#### Create an EFS File System
```sh
aws efs create-file-system --creation-token my-efs
```

#### Mount EFS in Task Definition
- Add a volume and mount point in your task definition to use EFS.

---

### **12. Ingress (Optional)**
To expose your application using an **Application Load Balancer (ALB)**, follow the steps in **Step 6** to create a Load Balancer and Target Group.

---
