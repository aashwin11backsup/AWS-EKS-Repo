# Project: AWS EKS Cluster with Terraform

**Provisioning and Managing an Amazon EKS Cluster using Terraform**

---

> **NOTE:**  
> Please **fork this repository** for your own use.  
> Make sure you have **AWS credentials configured** in your CLI (`aws configure`) or in your environment variables before running Terraform.  
> The cluster is provisioned with **private worker nodes** for security, while public subnets are used only for NAT Gateway and Load Balancers.

---

## Overview

This project provisions an **Amazon Elastic Kubernetes Service (EKS) cluster** using **Terraform**.  
It sets up all required AWS infrastructure components, including:

- **VPC, Subnets (public + private), NAT Gateway, Internet Gateway, Route Tables**
- **IAM Roles and Policies** for EKS Control Plane and Worker Nodes
- **EKS Cluster & Managed Node Group** (EC2 instances as worker nodes)
- **Networking configuration** so pods get IPs from the VPC (via CNI plugin)

Once created, you can easily connect to the cluster using `kubectl` and deploy your workloads.

---

## Architecture

- **VPC Setup**

  - 2 Public Subnets → for NAT Gateway + Load Balancers
  - 2 Private Subnets → for Worker Nodes & Pods
  - Internet Gateway, NAT Gateway, and Route Tables

- **EKS Cluster**

  - Control Plane managed by AWS
  - API Server:
    - **Public Access:** Enabled (can connect from your laptop)
    - **Private Access:** Disabled (for this setup)

- **EKS Node Group**

  - Worker Nodes (EC2 instances) in **private subnets**
  - On-Demand `t2.medium` instances (configurable)
  - Autoscaling between **1–5 nodes**

- **IAM Roles**
  - Control Plane Role (to manage the cluster)
  - Worker Node Role (with EKS, CNI, and ECR policies)

---

## Repository Structure

.
├── main.tf # Main Terraform configuration
├── variables.tf # Input variables
├── outputs.tf # Terraform outputs
├── vpc.tf # VPC, subnets, route tables, gateways
├── eks-cluster.tf # EKS cluster resource
├── eks-node-group.tf # Managed node group resource
├── iam.tf # IAM roles and policy attachments
├── provider.tf # AWS provider config
└── README.md # Project documentation

## Steps to Deploy

1. Initialize Terraform

Initialize the working directory containing your Terraform configuration:

```bash
terraform init
```

2. Validate the Code

Check whether the Terraform configuration is syntactically valid:

```bash
terraform validate
```

3. See the Execution Plan

Preview the changes Terraform will make before applying them:

```bash
terraform plan
```

4. Apply the Infrastructure

Apply the configuration and create the infrastructure:

```bash
terraform apply -auto-approve
```

This will create the full VPC, EKS cluster, IAM roles, and node group.

5. Update kubeconfig

Configure your local kubeconfig to connect with the newly created EKS cluster:

```bash
aws eks update-kubeconfig --region <aws-region> --name <eks-cluster-name>
```

Example:

```bash
aws eks update-kubeconfig --region us-east-2 --name staging-eks-demo
```

6. Verify Connection

Confirm that your Kubernetes cluster is accessible and the worker nodes are ready:

```bash
kubectl get nodes
```

You should see the worker nodes listed as Ready.

7. Cleanup

To destroy all resources created by Terraform and avoid ongoing costs:

```bash
terraform destroy -auto-approve
```
