# EKS Cluster Upgrade and Node AMI Update Workflow

This GitHub Actions workflow automates the process of upgrading an Amazon EKS Kubernetes cluster, updating associated add-ons, and replacing node AMIs in the Auto Scaling Group (ASG) one at a time. The workflow uses a configuration file to manage key variables, ensuring flexibility and ease of updates.

## Prerequisites

- **AWS Account**: Ensure you have access to an AWS account with permissions to manage EKS clusters, EC2 instances, and Auto Scaling Groups.
- **GitHub Repository**: This workflow should be added to a GitHub repository where you can manage your infrastructure as code.
- **GitHub Secrets**: Store your AWS credentials securely in GitHub Secrets:
  - `AWS_ACCESS_KEY_ID`
  - `AWS_SECRET_ACCESS_KEY`

## Configuration File

Create a file named `config.json` in your repository to store the AMI ID, cluster version, plugin versions, and Launch Template ID:

json
{
  "cluster_version": "1.29",
  "ami_id": "ami-12345678",
  "vpc_cni_version": "latest",
  "kube_proxy_version": "latest",
  "launch_template_id": "lt-0abcd1234efgh5678"
}

## Workflow Overview

The workflow consists of several jobs that perform the following actions:

- **Upgrade EKS Kubernetes Version**: Uses `eksctl` to upgrade the Kubernetes version of the EKS cluster.
- **Update EKS Add-ons**: Updates the Amazon VPC CNI and kube-proxy add-ons to ensure compatibility with the new Kubernetes version.
- **Update Node AMIs**: Fetches the latest AMI ID, updates the launch template, and replaces instances in the ASG one at a time.
