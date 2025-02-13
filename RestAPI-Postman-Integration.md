Here’s a step-by-step guide to deploy the RESTful API on **AWS EKS** using **Terraform**, including steps to install all prerequisite software and set up MongoDB on EKS. This guide is written in **GitHub Markdown format** for easy readability.

---

# Deploying a RESTful API on AWS EKS with Terraform

This guide provides step-by-step instructions to deploy a RESTful API on **AWS EKS** using **Terraform**. It includes setting up the EKS cluster, deploying the application, and integrating MongoDB.

---

## Prerequisites

### 1. **Install Required Tools**
1. **Install Terraform**:
   - Download and install Terraform from the [official website](https://www.terraform.io/downloads.html).
   - Verify the installation:
     ```bash
     terraform --version
     ```

2. **Install AWS CLI**:
   - Follow the instructions [here](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).

3. **Install `kubectl`**:
   - Follow the instructions [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

4. **Install `eksctl`**:
   - Follow the instructions [here](https://eksctl.io/introduction/#installation).

5. **Install Docker**:
   - Follow the instructions [here](https://docs.docker.com/get-docker/).

---

## Step 1: Set Up MongoDB

### 1. **Create a MongoDB Atlas Cluster**
   - Sign up for a free account at [MongoDB Atlas](https://www.mongodb.com/cloud/atlas).
   - Create a cluster and database.
   - Whitelist the EKS cluster's public IP range for access.
   - Get the connection string (replace `<username>`, `<password>`, and `<cluster-url>`):
     ```
     mongodb+srv://<username>:<password>@<cluster-url>/library?retryWrites=true&w=majority
     ```

---

## Step 2: Set Up Terraform Configuration

### 1. **Create a Terraform Directory**
   ```bash
   mkdir terraform-eks-rest-api
   cd terraform-eks-rest-api
   ```

### 2. **Add Terraform Files**
   - Create the following files in the directory.

---

### `main.tf`
This file defines the EKS cluster and related resources.

```hcl
provider "aws" {
  region = "us-west-2"
}

module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 18.0"

  cluster_name    = "rest-api-cluster"
  cluster_version = "1.22"

  vpc_id          = module.vpc.vpc_id
  subnet_ids      = module.vpc.private_subnets

  eks_managed_node_groups = {
    rest-api-nodes = {
      min_size     = 1
      max_size     = 3
      desired_size = 2

      instance_types = ["t2.micro"]
    }
  }
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 3.0"

  name = "rest-api-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-west-2a", "us-west-2b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true
}

output "cluster_name" {
  value = module.eks.cluster_name
}

output "kubeconfig" {
  value = module.eks.kubeconfig
}
```

---

### `kubernetes.tf`
This file defines the Kubernetes resources (ConfigMap, Secret, Deployment, and Service).

```hcl
provider "kubernetes" {
  host                   = module.eks.cluster_endpoint
  cluster_ca_certificate = base64decode(module.eks.cluster_certificate_authority_data)
  token                  = data.aws_eks_cluster_auth.cluster.token
}

data "aws_eks_cluster_auth" "cluster" {
  name = module.eks.cluster_name
}

resource "kubernetes_namespace" "rest_api" {
  metadata {
    name = "rest-api"
  }
}

resource "kubernetes_config_map" "rest_api_config" {
  metadata {
    name      = "rest-api-config"
    namespace = kubernetes_namespace.rest_api.metadata[0].name
  }

  data = {
    db_host     = var.db_host
    db_username = var.db_username
    db_name     = var.db_name
  }
}

resource "kubernetes_secret" "rest_api_secret" {
  metadata {
    name      = "rest-api-secret"
    namespace = kubernetes_namespace.rest_api.metadata[0].name
  }

  data = {
    db_password = var.db_password
  }

  type = "Opaque"
}

resource "kubernetes_deployment" "rest_api" {
  metadata {
    name      = "rest-api"
    namespace = kubernetes_namespace.rest_api.metadata[0].name
  }

  spec {
    replicas = 2

    selector {
      match_labels = {
        app = "rest-api"
      }
    }

    template {
      metadata {
        labels = {
          app = "rest-api"
        }
      }

      spec {
        container {
          name  = "rest-api"
          image = var.docker_image

          port {
            container_port = 5000
          }

          env {
            name = "DB_HOST"
            value_from {
              config_map_key_ref {
                name = kubernetes_config_map.rest_api_config.metadata[0].name
                key  = "db_host"
              }
            }
          }

          env {
            name = "DB_USERNAME"
            value_from {
              config_map_key_ref {
                name = kubernetes_config_map.rest_api_config.metadata[0].name
                key  = "db_username"
              }
            }
          }

          env {
            name = "DB_PASSWORD"
            value_from {
              secret_key_ref {
                name = kubernetes_secret.rest_api_secret.metadata[0].name
                key  = "db_password"
              }
            }
          }

          env {
            name = "DB_NAME"
            value_from {
              config_map_key_ref {
                name = kubernetes_config_map.rest_api_config.metadata[0].name
                key  = "db_name"
              }
            }
          }
        }
      }
    }
  }
}

resource "kubernetes_service" "rest_api" {
  metadata {
    name      = "rest-api"
    namespace = kubernetes_namespace.rest_api.metadata[0].name
  }

  spec {
    selector = {
      app = "rest-api"
    }

    port {
      port        = 80
      target_port = 5000
    }

    type = "LoadBalancer"
  }
}
```

---

### `variables.tf`
Define input variables for the Terraform configuration.

```hcl
variable "db_host" {
  description = "MongoDB Atlas host"
  type        = string
}

variable "db_username" {
  description = "MongoDB Atlas username"
  type        = string
}

variable "db_password" {
  description = "MongoDB Atlas password"
  type        = string
  sensitive   = true
}

variable "db_name" {
  description = "MongoDB database name"
  type        = string
}

variable "docker_image" {
  description = "Docker image for the REST API"
  type        = string
  default     = "your-docker-image"
}
```

---

### `outputs.tf`
Define outputs for the Terraform configuration.

```hcl
output "load_balancer_url" {
  description = "URL of the LoadBalancer"
  value       = kubernetes_service.rest_api.status[0].load_balancer[0].ingress[0].hostname
}
```

---

## Step 3: Initialize and Apply Terraform

### 1. **Initialize Terraform**
   ```bash
   terraform init
   ```

### 2. **Apply Terraform Configuration**
   ```bash
   terraform apply
   ```
   - Provide the required variables when prompted:
     - `db_host`: MongoDB Atlas host.
     - `db_username`: MongoDB Atlas username.
     - `db_password`: MongoDB Atlas password.
     - `db_name`: MongoDB database name.
     - `docker_image`: Docker image for the REST API.

### 3. **Verify the Deployment**
   - Get the LoadBalancer URL:
     ```bash
     terraform output load_balancer_url
     ```
   - Test the API using the LoadBalancer URL.

---

## Step 4: Clean Up

### 1. **Destroy Resources**
   - To clean up all resources created by Terraform:
     ```bash
     terraform destroy
     ```

---

This guide automates the deployment of the RESTful API on AWS EKS using Terraform. Let me know if you need further assistance!