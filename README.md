# eks

# Description

This repository is used to learn EKS

# Prerequisites

1. IAM user (I already create eksadmin IAM user) and IAM permission for eks
2. kubectl
3. eksctl

# Steps

1. Create VPC for EKS with [amazon-eks-vpc-sample.yaml](amazon-eks-vpc-sample.yaml)
2. Create cluster with [eks-cluster.yaml](eks-cluster.yaml)
3. Create deployment with [hello-deployment.yaml](hello-deployment.yaml)
4. (Optional) Create kubernetes dashboard. The documentation in reference number 3
5. Access service with load balancer

# References:

1. https://docs.aws.amazon.com/eks/latest/userguide/creating-a-vpc.html
2. https://github.com/weaveworks/eksctl/blob/main/examples/04-existing-vpc.yaml
3. https://docs.aws.amazon.com/eks/latest/userguide/dashboard-tutorial.html
