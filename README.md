### Terraform - EKS Cluster Setup Documentation

**Prerequisites**

Before you begin, make sure you have the following installed and configured on your local machine:

**Terraform:** Version 5.0 or higher.

**AWS CLI:** Installed and configured with your AWS credentials.

**kubectl:** Kubernetes command-line tool to manage the EKS cluster.


**Step-by-Step Guide**

**Step 1:** Create an IAM User.

**Step2:** Attach the following AWS Managed Policies.
```
AmazonEC2FullAccess
AmazonEKSClusterPolicy
AmazonEKSComputePolicy
AmazonEKSWorkerNodePolicy
```

**Step 2:** Attach Inline Policies.

EKS Policy
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
      
        "eks:CreateCluster",
        "eks:DescribeCluster",
        "eks:UpdateClusterConfig",
        "eks:UpdateClusterVersion",
        "eks:DeleteCluster",
        "eks:TagResource",
        "eks:ListClusters",
        "eks:DescribeUpdate",
        "eks:ListUpdates",
        "eks:CreateNodegroup",
        "eks:DescribeNodegroup",
        "eks:UpdateNodegroupConfig",
        "eks:DeleteNodegroup",
        "eks:DescribeAddonVersions",
        "eks:CreateAddon",
        "eks:DeleteAddon",
        "eks:DescribeAddon",
        "eks:TagResource",
        "eks:CreateAccessEntry",
        "eks:ListAddons",
        "eks:UpdateAddon",
        "eks:DescribeAddonVersions",
        "eks:DescribeAddon",
        "eks:ListAddons",
        "eks:CreateAddon",
        "eks:UpdateAddon",
        "eks:DeleteAddon",
        "eks:DescribeAccessEntry",
        "eks:CreateAccessEntry",
        "eks:DeleteAccessEntry",
        "eks:AssociateAccessPolicy",
        "eks:DisassociateAccessPolicy",
        "eks:ListAssociatedAccessPolicies"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "eks:ListAccessEntries",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "eks:ListAssociatedAccessPolicies",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "kms:DeleteAlias",
      "Resource": "arn:aws:kms:ap-south-1:626635408130:alias/eks/my-cluster"
    },
    {
      "Effect": "Allow",
      "Action": "logs:DeleteLogGroup",
      "Resource": "arn:aws:logs:ap-south-1:626635408130:log-group:/aws/eks/my-cluster/cluster:log-stream:*"
    }
  ]
}
```

Logs Policy

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
      
        "kms:CreateKey",
        "kms:DescribeKey",
        "kms:ListAliases",
        "kms:ListKeys",
        "kms:TagResource",
        "kms:CreateAlias",
        "logs:CreateLogGroup",
        "logs:DescribeLogGroups",
        "logs:PutRetentionPolicy",
        "logs:TagResource",
        "logs:ListTagsForResource"
      ],
      "Resource": "*"
    }
  ]
}
```
**Step 3:** Create Access Key and Secret Key.

**Step 4:** Connect to AWS Using AWS CLI.
```
aws configure
```
Enter your Access Key,
Secret Key,
default region.


**Step 5:** Create a main.tf file and add the following configuration for creating the EKS cluster.

main.tf
```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

module "vpc" {
  source = "terraform-aws-modules/vpc/aws"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = [ "ap-south-1a", "ap-south-1b", "ap-south-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true

  tags = {
    Terraform   = "true"
    Environment = "prod"
  }
}

module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = "my-cluster"
  cluster_version = "1.29"

  cluster_endpoint_public_access  = true

  cluster_addons = {
    coredns = {
      most_recent = true
    }
    kube-proxy = {
      most_recent = true
    }
    vpc-cni = {
      most_recent = true
    }
  }

  vpc_id                   = module.vpc.vpc_id
  control_plane_subnet_ids = module.vpc.public_subnets
  subnet_ids               = module.vpc.private_subnets

  eks_managed_node_group_defaults = {
    instance_types = ["t3.medium"]
  }

  eks_managed_node_groups = {
    example = {
      min_size     = 1
      max_size     = 2
      desired_size = 1

      instance_types = ["t3.medium"]
    }
  }

  enable_cluster_creator_admin_permissions = true
  tags = {
    Environment = "prod"
    Terraform   = "true"
  }
}
```

**Step 6:** Apply below commands to create cluster.

```
terraform init
```
```
terraform plan
```
```
terraform validate
```
```
terraform apply
```
**Update kubeconfig**
```
aws eks update-kubeconfig --name my-cluster --region ap-south-1
```
**Get nodes using kubectl commands**
```
kubectl get nodes
```
