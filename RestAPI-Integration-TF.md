# Automating RESTful API Deployment on AWS EKS with Terraform

## Overview
This guide provides step-by-step instructions for automating the deployment of a RESTful API on AWS Elastic Kubernetes Service (EKS) using Terraform. The deployment includes provisioning an EKS cluster, deploying the API, and integrating with MongoDB Atlas.

## Prerequisites
Before you begin, ensure you have the following installed:

- [Terraform](https://www.terraform.io/downloads.html)
- [AWS CLI](https://aws.amazon.com/cli/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [Helm](https://helm.sh/docs/intro/install/)
- MongoDB Atlas account and API key

## Project Structure
```
📂 eks-restapi-terraform
├── 📄 main.tf            # Main Terraform configuration file
├── 📄 variables.tf       # Variables for customization
├── 📄 outputs.tf         # Terraform output definitions
├── 📄 eks.tf             # EKS cluster definition
├── 📄 rds.tf             # MongoDB Atlas integration (optional)
├── 📂 modules
│   ├── 📂 eks            # EKS module
│   ├── 📂 networking     # VPC, subnets, and security groups
│   ├── 📂 api            # API Kubernetes deployment
├── 📂 manifests         # Kubernetes manifests
│   ├── 📄 deployment.yaml
│   ├── 📄 service.yaml
│   ├── 📄 ingress.yaml
└── 📄 README.md         # Documentation
```

## Steps to Deploy

### 1. Clone the Repository
```sh
git clone https://github.com/yourusername/eks-restapi-terraform.git
cd eks-restapi-terraform
```

### 2. Configure AWS Credentials
Ensure your AWS credentials are configured using:
```sh
aws configure
```

### 3. Initialize Terraform
```sh
terraform init
```

### 4. Plan the Deployment
```sh
terraform plan
```

### 5. Apply the Deployment
```sh
terraform apply -auto-approve
```

### 6. Configure `kubectl`
Retrieve the kubeconfig to access the cluster:
```sh
aws eks update-kubeconfig --name my-cluster --region us-east-1
```

### 7. Deploy the API
Apply the Kubernetes manifests:
```sh
kubectl apply -f manifests/
```

### 8. Verify Deployment
Check the running pods:
```sh
kubectl get pods -n my-namespace
```

Check service endpoints:
```sh
kubectl get svc -n my-namespace
```

### 9. Clean Up Resources
To destroy the EKS cluster and all associated resources:
```sh
terraform destroy -auto-approve
```

## Conclusion
This setup automates the deployment of a RESTful API on AWS EKS using Terraform, providing a scalable and manageable cloud-native solution integrated with MongoDB Atlas. Further enhancements can include CI/CD pipelines, monitoring, and security configurations.

## License
This project is licensed under the MIT License - see the LICENSE file for details.
