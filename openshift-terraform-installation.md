# ✨ OpenShift Installation with Terraform

Terraform enables Infrastructure as Code (IaC) for provisioning OpenShift Container Platform on cloud platforms like AWS and Azure. This guide covers using Terraform modules for automated OpenShift deployment.

## 🔹 Prerequisites

- Terraform 1.0+
- AWS CLI or Azure CLI configured
- OpenShift pull secret
- SSH key pair
- Cloud provider account with appropriate permissions

## 🔹 AWS Installation

### 📌 Method 1: Using Terraform AWS Provider

1. **Initialize Terraform project:**

   ```bash
   mkdir openshift-aws-terraform
   cd openshift-aws-terraform
   terraform init
   ```

2. **Create main.tf:**

   ```hcl
   terraform {
     required_providers {
       aws = {
         source  = "hashicorp/aws"
         version = "~> 4.0"
       }
     }
   }

   provider "aws" {
     region = var.aws_region
   }

   # VPC Configuration
   module "vpc" {
     source = "terraform-aws-modules/vpc/aws"

     name = "openshift-vpc"
     cidr = "10.0.0.0/16"

     azs             = ["${var.aws_region}a", "${var.aws_region}b", "${var.aws_region}c"]
     private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
     public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

     enable_nat_gateway = true
     enable_vpn_gateway = false

     tags = {
       Terraform = "true"
       Environment = "openshift"
     }
   }

   # Security Groups
   resource "aws_security_group" "openshift" {
     name_prefix = "openshift-"
     vpc_id      = module.vpc.vpc_id

     ingress {
       from_port   = 22
       to_port     = 22
       protocol    = "tcp"
       cidr_blocks = ["0.0.0.0/0"]
     }

     ingress {
       from_port   = 6443
       to_port     = 6443
       protocol    = "tcp"
       cidr_blocks = ["0.0.0.0/0"]
     }

     ingress {
       from_port   = 80
       to_port     = 80
       protocol    = "tcp"
       cidr_blocks = ["0.0.0.0/0"]
     }

     ingress {
       from_port   = 443
       to_port     = 443
       protocol    = "tcp"
       cidr_blocks = ["0.0.0.0/0"]
     }

     egress {
       from_port   = 0
       to_port     = 0
       protocol    = "-1"
       cidr_blocks = ["0.0.0.0/0"]
     }

     tags = {
       Name = "openshift-sg"
     }
   }

   # EC2 Instances for OpenShift
   resource "aws_instance" "master" {
     count = 3

     ami           = data.aws_ami.rhel.id
     instance_type = "m5.xlarge"
     key_name      = var.key_name

     vpc_security_group_ids = [aws_security_group.openshift.id]
     subnet_id              = module.vpc.private_subnets[count.index]

     root_block_device {
       volume_size = 100
       volume_type = "gp3"
     }

     tags = {
       Name = "openshift-master-${count.index + 1}"
       Role = "master"
     }
   }

   resource "aws_instance" "worker" {
     count = 3

     ami           = data.aws_ami.rhel.id
     instance_type = "m5.large"
     key_name      = var.key_name

     vpc_security_group_ids = [aws_security_group.openshift.id]
     subnet_id              = module.vpc.private_subnets[count.index]

     root_block_device {
       volume_size = 50
       volume_type = "gp3"
     }

     tags = {
       Name = "openshift-worker-${count.index + 1}"
       Role = "worker"
     }
   }

   # Load Balancer for API
   resource "aws_lb" "api" {
     name               = "openshift-api-lb"
     internal           = false
     load_balancer_type = "network"
     subnets            = module.vpc.public_subnets

     tags = {
       Name = "openshift-api"
     }
   }

   resource "aws_lb_target_group" "api" {
     name     = "openshift-api-tg"
     port     = 6443
     protocol = "TCP"
     vpc_id   = module.vpc.vpc_id

     health_check {
       enabled = true
       port     = 6443
       protocol = "TCP"
     }
   }

   resource "aws_lb_listener" "api" {
     load_balancer_arn = aws_lb.api.arn
     port              = "6443"
     protocol          = "TCP"

     default_action {
       type             = "forward"
       target_group_arn = aws_lb_target_group.api.arn
     }
   }

   resource "aws_lb_target_group_attachment" "api" {
     count            = 3
     target_group_arn = aws_lb_target_group.api.arn
     target_id        = aws_instance.master[count.index].id
     port             = 6443
   }

   # S3 Bucket for Registry
   resource "aws_s3_bucket" "registry" {
     bucket = "openshift-registry-${random_string.suffix.result}"

     tags = {
       Name = "openshift-registry"
     }
   }

   resource "random_string" "suffix" {
     length  = 8
     special = false
     upper   = false
   }

   # IAM Roles and Policies
   resource "aws_iam_role" "openshift" {
     name = "openshift-role"

     assume_role_policy = jsonencode({
       Version = "2012-10-17"
       Statement = [
         {
           Action = "sts:AssumeRole"
           Effect = "Allow"
           Principal = {
             Service = "ec2.amazonaws.com"
           }
         }
       ]
     })
   }

   resource "aws_iam_role_policy_attachment" "openshift" {
     role       = aws_iam_role.openshift.name
     policy_arn = "arn:aws:iam::aws:policy/AmazonEC2FullAccess"
   }

   # Data sources
   data "aws_ami" "rhel" {
     most_recent = true
     owners      = ["amazon"]

     filter {
       name   = "name"
       values = ["RHEL-8*x86_64*"]
     }
   }
   ```

3. **Create variables.tf:**

   ```hcl
   variable "aws_region" {
     description = "AWS region"
     type        = string
     default     = "us-east-1"
   }

   variable "key_name" {
     description = "SSH key pair name"
     type        = string
   }

   variable "cluster_name" {
     description = "OpenShift cluster name"
     type        = string
     default     = "openshift-cluster"
   }

   variable "openshift_version" {
     description = "OpenShift version"
     type        = string
     default     = "4.12"
   }
   ```

4. **Create outputs.tf:**

   ```hcl
   output "master_ips" {
     description = "Private IPs of master nodes"
     value       = aws_instance.master[*].private_ip
   }

   output "worker_ips" {
     description = "Private IPs of worker nodes"
     value       = aws_instance.worker[*].private_ip
   }

   output "api_lb_dns" {
     description = "DNS name of API load balancer"
     value       = aws_lb.api.dns_name
   }

   output "registry_bucket" {
     description = "S3 bucket for registry storage"
     value       = aws_s3_bucket.registry.bucket
   }
   ```

5. **Deploy infrastructure:**

   ```bash
   terraform plan
   terraform apply
   ```

### 📌 Method 2: OpenShift on AWS with ROSA

For Red Hat OpenShift Service on AWS (ROSA):

```hcl
# ROSA Cluster
resource "awscc_rosa_cluster" "example" {
  cluster_name = "my-rosa-cluster"
  version      = "4.12"

  cloud_region = "us-east-1"

  machine_cidr     = "10.0.0.0/16"
  service_cidr     = "172.30.0.0/16"
  pod_cidr         = "10.128.0.0/14"
  host_prefix      = 23

  # STS configuration
  sts = {
    role_arn         = aws_iam_role.cluster.arn
    support_role_arn = aws_iam_role.support.arn
    instance_iam_roles = {
      master_role_arn = aws_iam_role.master.arn
      worker_role_arn = aws_iam_role.worker.arn
    }
    operator_iam_roles = [
      {
        role_arn = aws_iam_role.operator.arn
        service_account_namespaces = ["openshift-cluster-csi-drivers", "openshift-machine-api"]
        service_account_names = ["aws-ebs-csi-driver-operator", "aws-ebs-csi-driver-controller-sa"]
      }
    ]
  }

  # Network configuration
  aws_subnet_ids = [
    aws_subnet.private_a.id,
    aws_subnet.private_b.id,
    aws_subnet.private_c.id
  ]

  # Security groups
  machine_cidr = "10.0.0.0/16"

  # Worker nodes
  worker_machine_replica_count = 3
  worker_machine_type          = "m5.xlarge"

  # Master nodes
  master_machine_type = "m5.xlarge"

  # Additional configuration
  default_mp_labels = {
    "environment" = "production"
  }

  tags = {
    "Environment" = "production"
    "ManagedBy"   = "terraform"
  }
}
```

## 🔹 Azure Installation

### 📌 Method 1: OpenShift on Azure IaaS

1. **Create main.tf for Azure:**

   ```hcl
   terraform {
     required_providers {
       azurerm = {
         source  = "hashicorp/azurerm"
         version = "~> 3.0"
       }
     }
   }

   provider "azurerm" {
     features {}
   }

   # Resource Group
   resource "azurerm_resource_group" "openshift" {
     name     = "openshift-rg"
     location = var.location
   }

   # Virtual Network
   resource "azurerm_virtual_network" "openshift" {
     name                = "openshift-vnet"
     address_space       = ["10.0.0.0/16"]
     location            = azurerm_resource_group.openshift.location
     resource_group_name = azurerm_resource_group.openshift.name
   }

   # Subnets
   resource "azurerm_subnet" "master" {
     name                 = "master-subnet"
     resource_group_name  = azurerm_resource_group.openshift.name
     virtual_network_name = azurerm_virtual_network.openshift.name
     address_prefixes     = ["10.0.1.0/24"]
   }

   resource "azurerm_subnet" "worker" {
     name                 = "worker-subnet"
     resource_group_name  = azurerm_resource_group.openshift.name
     virtual_network_name = azurerm_virtual_network.openshift.name
     address_prefixes     = ["10.0.2.0/24"]
   }

   # Network Security Groups
   resource "azurerm_network_security_group" "openshift" {
     name                = "openshift-nsg"
     location            = azurerm_resource_group.openshift.location
     resource_group_name = azurerm_resource_group.openshift.name

     security_rule {
       name                       = "SSH"
       priority                   = 1001
       direction                  = "Inbound"
       access                     = "Allow"
       protocol                   = "Tcp"
       source_port_range          = "*"
       destination_port_range     = "22"
       source_address_prefix      = "*"
       destination_address_prefix = "*"
     }

     security_rule {
       name                       = "HTTPS"
       priority                   = 1002
       direction                  = "Inbound"
       access                     = "Allow"
       protocol                   = "Tcp"
       source_port_range          = "*"
       destination_port_range     = "443"
       source_address_prefix      = "*"
       destination_address_prefix = "*"
     }

     security_rule {
       name                       = "API"
       priority                   = 1003
       direction                  = "Inbound"
       access                     = "Allow"
       protocol                   = "Tcp"
       source_port_range          = "*"
       destination_port_range     = "6443"
       source_address_prefix      = "*"
       destination_address_prefix = "*"
     }
   }

   # Virtual Machines
   resource "azurerm_linux_virtual_machine" "master" {
     count               = 3
     name                = "openshift-master-${count.index + 1}"
     resource_group_name = azurerm_resource_group.openshift.name
     location            = azurerm_resource_group.openshift.location
     size                = "Standard_D8s_v3"
     admin_username      = "openshift"

     network_interface_ids = [
       azurerm_network_interface.master[count.index].id,
     ]

     admin_ssh_key {
       username   = "openshift"
       public_key = file(var.ssh_public_key)
     }

     os_disk {
       caching              = "ReadWrite"
       storage_account_type = "Premium_LRS"
       disk_size_gb         = 100
     }

     source_image_reference {
       publisher = "RedHat"
       offer     = "RHEL"
       sku       = "8.4"
       version   = "latest"
     }

     tags = {
       Role = "master"
     }
   }

   resource "azurerm_linux_virtual_machine" "worker" {
     count               = 3
     name                = "openshift-worker-${count.index + 1}"
     resource_group_name = azurerm_resource_group.openshift.name
     location            = azurerm_resource_group.openshift.location
     size                = "Standard_D4s_v3"
     admin_username      = "openshift"

     network_interface_ids = [
       azurerm_network_interface.worker[count.index].id,
     ]

     admin_ssh_key {
       username   = "openshift"
       public_key = file(var.ssh_public_key)
     }

     os_disk {
       caching              = "ReadWrite"
       storage_account_type = "Premium_LRS"
       disk_size_gb         = 50
     }

     source_image_reference {
       publisher = "RedHat"
       offer     = "RHEL"
       sku       = "8.4"
       version   = "latest"
     }

     tags = {
       Role = "worker"
     }
   }

   # Network Interfaces
   resource "azurerm_network_interface" "master" {
     count               = 3
     name                = "master-nic-${count.index + 1}"
     location            = azurerm_resource_group.openshift.location
     resource_group_name = azurerm_resource_group.openshift.name

     ip_configuration {
       name                          = "internal"
       subnet_id                     = azurerm_subnet.master.id
       private_ip_address_allocation = "Dynamic"
     }
   }

   resource "azurerm_network_interface" "worker" {
     count               = 3
     name                = "worker-nic-${count.index + 1}"
     location            = azurerm_resource_group.openshift.location
     resource_group_name = azurerm_resource_group.openshift.name

     ip_configuration {
       name                          = "internal"
       subnet_id                     = azurerm_subnet.worker.id
       private_ip_address_allocation = "Dynamic"
     }
   }

   # Load Balancer
   resource "azurerm_lb" "api" {
     name                = "openshift-api-lb"
     location            = azurerm_resource_group.openshift.location
     resource_group_name = azurerm_resource_group.openshift.name
     sku                 = "Standard"

     frontend_ip_configuration {
       name                 = "PublicIPAddress"
       public_ip_address_id = azurerm_public_ip.api.id
     }
   }

   resource "azurerm_public_ip" "api" {
     name                = "openshift-api-pip"
     location            = azurerm_resource_group.openshift.location
     resource_group_name = azurerm_resource_group.openshift.name
     allocation_method   = "Static"
     sku                 = "Standard"
   }

   # Storage Account for Registry
   resource "azurerm_storage_account" "registry" {
     name                     = "openshiftregistry${random_string.suffix.result}"
     resource_group_name      = azurerm_resource_group.openshift.name
     location                 = azurerm_resource_group.openshift.location
     account_tier             = "Standard"
     account_replication_type = "LRS"

     tags = {
       purpose = "openshift-registry"
     }
   }

   resource "random_string" "suffix" {
     length  = 8
     special = false
     upper   = false
   }
   ```

2. **Create variables.tf for Azure:**

   ```hcl
   variable "location" {
     description = "Azure region"
     type        = string
     default     = "East US"
   }

   variable "ssh_public_key" {
     description = "Path to SSH public key"
     type        = string
   }

   variable "cluster_name" {
     description = "OpenShift cluster name"
     type        = string
     default     = "openshift-cluster"
   }

   variable "openshift_version" {
     description = "OpenShift version"
     type        = string
     default     = "4.12"
   }
   ```

### 📌 Method 2: Azure Red Hat OpenShift (ARO)

For managed OpenShift service on Azure:

```hcl
resource "azurerm_redhat_openshift_cluster" "example" {
  name                = "example-aro-cluster"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name

  cluster_profile {
    domain      = "example"
    version     = "4.12.25"
    pull_secret = var.pull_secret
  }

  network_profile {
    pod_cidr     = "10.128.0.0/14"
    service_cidr = "172.30.0.0/16"
  }

  master_profile {
    subnet_id = azurerm_subnet.master.id
  }

  worker_profile {
    subnet_id = azurerm_subnet.worker.id
    count     = 3
  }

  api_server_profile {
    visibility = "Public"
  }

  ingress_profile {
    visibility = "Public"
  }

  service_principal {
    client_id     = var.client_id
    client_secret = var.client_secret
  }

  tags = {
    environment = "production"
  }
}
```

## 🔹 Advanced Terraform Configurations

### 📌 Remote State Management

```hcl
terraform {
  backend "s3" {
    bucket = "openshift-terraform-state"
    key    = "openshift/terraform.tfstate"
    region = "us-east-1"
  }
}

# Or for Azure
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state"
    storage_account_name = "terraformstate"
    container_name       = "tfstate"
    key                  = "openshift.terraform.tfstate"
  }
}
```

### 📌 Modules for Reusability

Create reusable modules:

```hcl
module "openshift_cluster" {
  source = "./modules/openshift"

  cluster_name    = "production-cluster"
  aws_region      = "us-east-1"
  master_count    = 3
  worker_count    = 6
  instance_type   = "m5.xlarge"
  openshift_version = "4.12"

  providers = {
    aws = aws
  }
}
```

### 📌 Multi-Environment Deployment

```hcl
# Environment-specific variables
variable "environment" {
  description = "Environment name"
  type        = string
}

locals {
  cluster_name = "${var.environment}-openshift"
  tags = {
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}

# Conditional resource creation
resource "aws_instance" "bastion" {
  count = var.environment == "production" ? 1 : 0

  # ... bastion configuration
}
```

## 🔹 Troubleshooting Terraform Deployments

### 📌 Common Issues

1. **Provider authentication failures:**
   - Verify AWS credentials or Azure service principal
   - Check IAM permissions
   - Validate subscription/region access

2. **Resource quota exceeded:**
   - Check service limits
   - Request quota increases
   - Use smaller instance types

3. **Network connectivity issues:**
   - Verify security group rules
   - Check subnet configurations
   - Validate route tables

4. **State lock conflicts:**
   - Use `terraform force-unlock` (carefully)
   - Check for running deployments
   - Implement proper locking mechanisms

### 📌 Debugging Techniques

- **Enable debug logging:**

  ```bash
  export TF_LOG=DEBUG
  terraform apply
  ```

- **Validate configuration:**

  ```bash
  terraform validate
  terraform plan
  ```

- **Check resource state:**

  ```bash
  terraform state list
  terraform state show <resource>
  ```

- **Refresh state:**

  ```bash
  terraform refresh
  ```

## 🔹 Best Practices

### 📌 State Management

- Use remote state backends
- Enable state locking
- Regular state backups
- Use workspaces for environments

### 📌 Security

- Store sensitive data in vaults
- Use least-privilege IAM roles
- Encrypt state files
- Regular credential rotation

### 📌 Version Control

- Store Terraform code in Git
- Use semantic versioning
- Implement code reviews
- Document changes

### 📌 Cost Optimization

- Use spot instances where appropriate
- Implement auto-scaling
- Regular resource cleanup
- Monitor cloud costs

## 🔹 Integration with CI/CD

### 📌 GitHub Actions Example

```yaml
name: 'Terraform'

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2

    - name: Terraform Init
      run: terraform init

    - name: Terraform Plan
      run: terraform plan

    - name: Terraform Apply
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
      run: terraform apply -auto-approve
```

### 📌 Integration with OpenShift GitOps

```yaml
# ArgoCD Application for Terraform
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: openshift-infrastructure
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/openshift-terraform
    targetRevision: HEAD
    path: terraform
  destination:
    server: https://kubernetes.default.svc
    namespace: openshift-infrastructure
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## 🔹 Monitoring and Compliance

### 📌 Cost Monitoring

```hcl
# AWS Cost Allocation Tags
resource "aws_cost_allocation_tag" "openshift" {
  tag_key = "openshift-cluster"
  status  = "Active"
}

# Azure Cost Management
resource "azurerm_monitor_metric_alert" "budget" {
  name                = "budget-alert"
  resource_group_name = azurerm_resource_group.openshift.name
  scopes              = [azurerm_resource_group.openshift.id]

  criteria {
    metric_namespace = "Microsoft.Consumption/budgets"
    metric_name      = "Cost"
    aggregation      = "Total"
    operator         = "GreaterThan"
    threshold        = 1000
  }
}
```

### 📌 Compliance and Governance

- Implement resource tagging standards
- Use policy-as-code tools (Open Policy Agent, AWS Config, Azure Policy)
- Regular compliance audits
- Automated remediation workflows
