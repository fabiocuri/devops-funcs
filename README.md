This repo contains all the code for the exercises of the DevOps bootcamp.

Author: Fabio Curi Paixao 

E-mail: fcuri91@gmail.com

**Exercises 1 and 2**

Set up the provider file providers.tf to specify the AWS version we want to use as provider:

```
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

Set up a var file called dev.tfvars:

```
env_prefix = "dev"
k8s_version = "1.28"
cluster_name = "devops-cluster"
region = "eu-central-1"
```

And the default variables.tf:

```
variable env_prefix {
  default = "test"
}

variable k8s_version {
  default = "1.28"
}

variable cluster_name {
  default = "devops-cluster"
}

variable region {
  default = "eu-central-1"
}
```

Following the course, use the templates and create vpc.tf:

```
terraform {
  backend "s3" {
    bucket = "s3_bucket"
    key    = "state/state.tfstate"
    region  = "eu-central-1"
  }
}

provider "aws" {
  region  = var.region
}

data "aws_availability_zones" "available" {}

locals {
  cluster_name = var.cluster_name 
}

resource "random_string" "suffix" {
  length  = 8
  special = false
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.2.0"

  name                 = "my-vpc"
  cidr                 = "10.0.0.0/16"
  azs                  = data.aws_availability_zones.available.names
  private_subnets      = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets       = ["10.0.4.0/24", "10.0.5.0/24", "10.0.6.0/24"]
  enable_nat_gateway   = true
  single_nat_gateway   = true
  enable_dns_hostnames = true

  tags = {
    "kubernetes.io/cluster/${local.cluster_name}" = "shared"
  }

  public_subnet_tags = {
    "kubernetes.io/cluster/${local.cluster_name}" = "shared"
    "kubernetes.io/role/elb"                      = "1"
  }

  private_subnet_tags = {
    "kubernetes.io/cluster/${local.cluster_name}" = "shared"
    "kubernetes.io/role/internal-elb"             = "1"
  }
}
```

Now, the mySQL file, that will be created inside our EKS cluster:

```
data "aws_eks_cluster" "cluster" {
  name = module.eks.cluster_name
}

data "aws_eks_cluster_auth" "cluster" {
  name = module.eks.cluster_name
}

provider "kubernetes" {
  host                   = data.aws_eks_cluster.cluster.endpoint
  token                  = data.aws_eks_cluster_auth.cluster.token
  cluster_ca_certificate = base64decode(data.aws_eks_cluster.cluster.certificate_authority.0.data)
}

provider "helm" {
  kubernetes {
    host = data.aws_eks_cluster.cluster.endpoint
    token = data.aws_eks_cluster_auth.cluster.token
    cluster_ca_certificate = base64decode(data.aws_eks_cluster.cluster.certificate_authority.0.data)
  }
}

resource "helm_release" "mysql" {
  name       = "my-release"
  repository = "https://charts.bitnami.com/bitnami"
  chart      = "mysql"
  version    = "9.14.0"
  timeout    = "1000" # seconds

  values = [
    "${file("values.yaml")}"
  ]

  set {
    name  = "volumePermissions.enabled" 
    value = true
  }
}
```

and now the eks-cluster.tf file that will create the EKS cluster:

```
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "19.20.0"

  cluster_name                   = var.cluster_name
  cluster_version                = var.k8s_version
  cluster_endpoint_public_access = true

  subnet_ids = module.vpc.private_subnets
  vpc_id     = module.vpc.vpc_id
  tags = {
    environment = "bootcamp"
  }

  cluster_addons = {
    aws-ebs-csi-driver = {}
  } 

  eks_managed_node_groups = {
    nodegroup = {
      use_custom_templates = false
      instance_types       = ["t3.small"]
      node_group_name      = var.env_prefix

      min_size     = 1
      max_size     = 3
      desired_size = 3

      tags = {
        Name = "${var.env_prefix}"
      }   
      iam_role_additional_policies = {
        AmazonEBSCSIDriverPolicy = "arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy"
      }  
    }
  }
  fargate_profiles = {
    profile = {
      name = "my-fargate-profile"
      selectors = [
        {
          namespace = "my-app"
        }
      ]
    }
  }
}
```

And now we can apply

```
terraform apply -var-file="dev.tfvars"
```

**Exercise 3**

Here below is the Jenkinsfile that will provision this new infra.
Here, we suppose that the AWS credentials have already been stored in Jenkins.
Also, the variables start with TF_VAR_ and they will be recognized in Terraform.

```
#!/usr/bin/env groovy

pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
    }
    stages {
        stage('provision cluster') {
            environment {
                TF_VAR_env_prefix = "dev"
                TF_VAR_k8s_version = "1.28"
                TF_VAR_cluster_name = "my-cluster"
                TF_VAR_region = "eu-central-1"
            }
            steps {
                script {
                    sh "terraform init"
                    sh "terraform apply --auto-approve"
                    
                    env.K8S_CLUSTER_URL = sh(
                        script: "terraform output cluster_url",
                        returnStdout: true
                    ).trim()
                    
                    sh "aws eks update-kubeconfig --name ${TF_VAR_cluster_name} --region ${TF_VAR_region}"
                }
            }
        }
    }
}

```
