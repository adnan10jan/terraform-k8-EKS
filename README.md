# DevOps Technical Assessment: Private EKS Cluster

This repository contains the deliverables for the DevOps Technical Assessment, focusing on provisioning a private-only Amazon EKS cluster and deploying a sample application stack.

## Deliverables

1. **Terraform Source Code**: Located in the `terraform/` directory.
2. **Kubernetes YAML Files**: Located in the `kubernetes/` directory.
3. **SSH Access Document**: `ssh-access-strategy.md` located in the root directory.

## Part A: Terraform Infrastructure

The Terraform code provisions a completely private AWS architecture.
- **VPC**: A custom VPC (`10.0.0.0/16`) with private subnets across 2 Availability Zones.
- **NAT Gateway**: Configured in public subnets to provide outbound internet access to the private worker nodes.
- **EKS Cluster**: Configured with `endpoint_private_access = true` and `endpoint_public_access = false`. 
- **Security**: Least-privilege IAM roles are used. The cluster security group strictly limits ingress traffic to the VPC CIDR blocks.

### How to Deploy
Ensure you have the AWS CLI configured with your credentials, then run:
```bash
cd terraform
terraform init
terraform plan
terraform apply
```

## Part B: Kubernetes Deployment

The application stack consists of **Tomcat** and **HAProxy** acting as a frontend load balancer. 
- **Tomcat**: Deployed with a minimum of 2 replicas and exposed internally via a `ClusterIP` service.
- **HAProxy**: Configured via a `ConfigMap`. It routes traffic to the Tomcat service using native Kubernetes DNS service discovery. It is exposed via a `NodePort` service, strictly adhering to the constraint of avoiding public `LoadBalancer` services.

### How to Deploy
Since the EKS cluster is private-only, you must apply these manifests from within the VPC (e.g., via AWS CloudShell, a Bastion Host, or SSM Session Manager on a worker node).
```bash
kubectl apply -f kubernetes/
```

## Part C: Secure SSH Access

Please refer to the `ssh-access-strategy.md` file in the root directory for a detailed explanation of how to securely access the private worker nodes using AWS Systems Manager (SSM) Session Manager without requiring public IPs, inbound SSH ports, or SSH keys.
