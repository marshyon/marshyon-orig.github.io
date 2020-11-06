---
title: "Deploying Kubernetes Clusters to Cloud Providers"
date: 2020-11-06T14:10:58+01:00
draft: false
---

K8s in any cloud platform is essentially just K8s, in that that it is still fundamentally Kubernetes 'under the hood'. However there are different ways in which to deploy K8s in AWS for example as opposed to Azure, Google cloud and others.

When exposing services to the internet or rather, network sources outside that of the cluster, there are differences brought about by 
the way the prevailing service integrates with said exposure. Azure, Amazon and Google all have their own 'Ingress' load balancer
integrations. We are not entirely limited to the the stock provision from each provider. For example Nginx offers an alternative Ingress load balancer solution that runs as a service within the cluster as do those provided by the big 3 and others. So we have lots of options however in the simplest form, each provider has an Ingress and integration layer that creates a load balancer outside of the cluster that is specific to each cloud platform.

### Prerequisites for K8s and Terraform

__kubectl__

```
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv kubectl /usr/local/bin
```

__terraform__

This is only required if you want to use terraform (ofcourse)

download terraform zip file from https://www.terraform.io/downloads.html

```
unzip /mnt/c/Users/jon/Downloads/terraform_0.13.5_linux_amd64.zip
sudo mv terraform /usr/local/bin/terraform
``` 


## AWS

### Prerequisites for AWS

I've found that implementing K8s using command line tools such as Terraform using Linux to do so gives me best results but this is my opinion having worked on Linux for > 10 years and being entirely happy with bash which most K8s howtos use. Windows WSL is an adequate Linux environment for this so Windows users are in no way out of options. Mac users are all cool, so they can handle anything. If your using Linux as your core OS there nees nothing more to be said.

___aws and eksctl___

example of aws cli install :

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

To allow aws cli to function you need to run `aws configiure` which will ask for a key, secret which is the minimum required to function. 
A default location and output format ( for example eu-west-2 and json ).

Get an access key / secret key pair from the AWS console under `IAM` and search ofr 'your security credentials'.

example eksctl install : 

```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```

for the latest instructions see https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html

___aws-iam-authenticator___

Only needed if you want to use Terraform to build the cluster. Without this Terraform builds will fail. Terraform, I believe specifically the aws plugin that terraform installs needs to have this but I could be mistaken. 

Here is an example install :

```
curl -o aws-iam-authenticator https://amazon-eks.s3.us-west-2.amazonaws.com/1.18.8/2020-09-18/bin/linux/amd64/aws-iam-authenticator 
chmod +x ./aws-iam-authenticator
sudo mv aws-iam-authenticator /usr/local/bin
 ```

for the latest installation see [https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html)

### List Clusters in AWS

We need an easy way to know if there are clusters already created in our account. Heres a quick way to get that information :

```
aws eks list-clusters
{
    "clusters": []
}
```

In this example, there are no clusters. This is good as at this point nothing is deployed 


### Create an AWS K8s cluster with eksctl

a cluster in AWS can be created with a single command :

```

eksctl create cluster \
  --name myk8s-cluster \
  --node-type t2.micro \
  --nodes 3 \
  --nodes-min 3 \
  --nodes-max 5 \
  --region eu-west-2
```

This can take some time to complete. Often over 10 minutes so be prepared to wait.

### Delete the AWS K8s cluster with eksctl

```
eksctl delete cluster --name myk8s-cluster --region eu-west-2
```


### Create an AWS K8s with Terraform

with a file called for example `main.tf`

```
provider "aws" {
  region = "eu-west-2"
}

data "aws_eks_cluster" "cluster" {
  name = module.eks.cluster_id
}

data "aws_eks_cluster_auth" "cluster" {
  name = module.eks.cluster_id
}

provider "kubernetes" {
  host                   = data.aws_eks_cluster.cluster.endpoint
  cluster_ca_certificate = base64decode(data.aws_eks_cluster.cluster.certificate_authority.0.data)
  token                  = data.aws_eks_cluster_auth.cluster.token
  load_config_file       = false
  version                = "~> 1.11"
}

data "aws_availability_zones" "available" {
}

locals {
  cluster_name = "my-cluster"
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "2.47.0"

  name                 = "k8s-vpc"
  cidr                 = "172.16.0.0/16"
  azs                  = data.aws_availability_zones.available.names
  private_subnets      = ["172.16.1.0/24", "172.16.2.0/24", "172.16.3.0/24"]
  public_subnets       = ["172.16.4.0/24", "172.16.5.0/24", "172.16.6.0/24"]
  enable_nat_gateway   = true
  single_nat_gateway   = true
  enable_dns_hostnames = true

  public_subnet_tags = {
    "kubernetes.io/cluster/${local.cluster_name}" = "shared"
    "kubernetes.io/role/elb"                      = "1"
  }

  private_subnet_tags = {
    "kubernetes.io/cluster/${local.cluster_name}" = "shared"
    "kubernetes.io/role/internal-elb"             = "1"
  }
}

module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "12.2.0"

  cluster_name    = local.cluster_name
  cluster_version = "1.17"
  subnets         = module.vpc.private_subnets

  vpc_id = module.vpc.vpc_id

  node_groups = {
    first = {
      desired_capacity = 4
      max_capacity     = 10
      min_capacity     = 3

      instance_type = "t2.micro"
    }
  }

  write_kubeconfig   = true
  config_output_path = "./"
}
```

from within the directory that contains `main.tf` :

```
terraform init
terraform plan
terraform apply
```

if successful, the above will create a credentials file within the same directory which may be used to then authenticate against the K8s
cluster :

```
export KUBECONFIG="${PWD}/kubeconfig_my-cluster"
kubectl get pods --all-namespaces
```

and this should firstly create the environment variable `KUBECONFIG`, recognised by `kubectl` to contain the location of the credentials file and secondly
list all pods in all namespaces

### Delete the AWS K8s cluster with Terraform

To delete the K8s cluster created with Terraform, in the same directory as `terraform apply` command was executed :

```
terraform destroy
```

## Google Cloud

### Prerequisites for Google Cloud

follow installation instructions to install `gcloud` command line interface at :

https://cloud.google.com/sdk/docs/install

### Initialize Google Cloud Environment

From the Google Cloud console, create a project then initialize Google Cloud command line environment to use it with :

```
gcloud init
```

### Create cluster with gcloud

```
gcloud container clusters create kubia --num-nodes 3 --machine-type e2-small
```

### Delete cluster with gcloud

```
gcloud container clusters delete kubia
```

# References

https://learnk8s.io/terraform-eks

https://www.manning.com/books/kubernetes-in-action
