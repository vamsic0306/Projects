# Infrastructure as Code (IaC) Automation Suite

## Project Overview
This project demonstrates a complete Infrastructure as Code implementation using Terraform, AWS, and GitHub Actions for automated deployments.

## Architecture
```
Infrastructure
├── AWS VPC
│   ├── Public Subnets
│   └── Private Subnets
├── EKS Cluster
├── RDS Database
└── Monitoring Stack
```

## Prerequisites
- AWS Account
- GitHub Account
- Terraform installed (v1.0+)
- AWS CLI configured
- kubectl installed

## Step-by-Step Implementation Guide

### 1. Initial Setup
```bash
# Clone the repository
git clone <your-repo>
cd infrastructure-automation

# Initialize Terraform
cd terraform
terraform init
```

### 2. AWS Infrastructure Setup
1. Configure AWS Provider:
```hcl
# terraform/provider.tf
provider "aws" {
  region = "us-west-2"
}
```

2. Create VPC Module:
```hcl
# terraform/modules/vpc/main.tf
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-west-2a", "us-west-2b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
}
```

### 3. EKS Cluster Deployment
```hcl
# terraform/modules/eks/main.tf
module "eks" {
  source = "terraform-aws-modules/eks/aws"
  
  cluster_name    = "my-cluster"
  cluster_version = "1.24"
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets
}
```

### 4. Monitoring Setup
1. Deploy Prometheus:
```bash
# monitoring/prometheus/deploy.sh
kubectl apply -f prometheus-operator.yaml
kubectl apply -f prometheus-cluster-role.yaml
```

2. Configure Grafana:
```bash
# monitoring/grafana/deploy.sh
kubectl apply -f grafana-deployment.yaml
kubectl apply -f grafana-service.yaml
```

### 5. CI/CD Pipeline Setup
1. Create GitHub Actions workflow:
```yaml
# .github/workflows/terraform.yml
name: 'Terraform'

on:
  push:
    branches:
    - main
  pull_request:

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v1
    
    - name: Terraform Init
      run: terraform init
    
    - name: Terraform Plan
      run: terraform plan
    
    - name: Terraform Apply
      if: github.ref == 'refs/heads/main'
      run: terraform apply -auto-approve
```

## Testing
1. Infrastructure Validation:
```bash
# Run Terraform plan
terraform plan

# Validate AWS resources
aws eks list-clusters
aws ec2 describe-vpcs
```

2. Monitoring Validation:
```bash
# Check Prometheus status
kubectl get pods -n monitoring

# Access Grafana dashboard
kubectl port-forward svc/grafana 3000:3000 -n monitoring
```

## Best Practices
- Use remote state storage (S3 + DynamoDB)
- Implement proper tagging strategy
- Enable encryption at rest
- Use proper IAM roles and policies
- Implement proper backup strategies

## Troubleshooting
Common issues and solutions:

1. EKS Cluster Access:
```bash
aws eks update-kubeconfig --name cluster-name --region region
```

2. Terraform State Issues:
```bash
terraform state list
terraform state show resource_name
```

## Security Considerations
- Enable VPC Flow Logs
- Implement Network ACLs
- Use Security Groups
- Enable AWS GuardDuty
- Implement AWS Config Rules

## Cost Optimization
- Use Spot Instances where applicable
- Implement auto-scaling
- Use proper instance sizing
- Enable detailed billing
- Set up cost alerts
