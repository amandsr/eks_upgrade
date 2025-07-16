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

```json
{
  "cluster_version": "1.29",
  "ami_id": "ami-12345678",
  "vpc_cni_version": "latest",
  "kube_proxy_version": "latest",
  "launch_template_id": "lt-0abcd1234efgh5678"
}
```

## Workflow Overview

The workflow consists of several jobs that perform the following actions:

- **Upgrade EKS Kubernetes Version**: Uses `eksctl` to upgrade the Kubernetes version of the EKS cluster.
- **Update EKS Add-ons**: Updates the Amazon VPC CNI and kube-proxy add-ons to ensure compatibility with the new Kubernetes version.
- **Update Node AMIs**: Fetches the latest AMI ID, updates the launch template, and replaces instances in the ASG one at a time.

## Detailed Steps
**1. Upgrade EKS Kubernetes Version**

Setup eksctl: Installs eksctl on the runner to manage EKS clusters.
Load Configuration: Reads values from config.json to set environment variables.
Upgrade Cluster: Executes the eksctl upgrade cluster command using the specified cluster version.
Validation: Uses kubectl version --short to validate the upgrade.

**2. Update EKS Add-ons**
Update Amazon VPC CNI Add-on: Uses eksctl utils update-cni to update the VPC CNI add-on.
Update kube-proxy Add-on: Uses eksctl utils update-kube-proxy to update the kube-proxy add-on.
Validation: Checks the deployment status using kubectl get deployment -n kube-system.

**3. Update Node AMIs**
Fetch Latest AMI ID: Uses AWS CLI to retrieve the latest AMI ID from config.json.
Update Launch Template: Creates a new version of the launch template using the specified AMI ID and Launch Template ID.
Update Auto Scaling Group: Adjusts the desired capacity to replace instances one at a time, validates each new instance, and ensures the termination policy is set to OldestInstance.

## Validation and Rollback
- **Validation:** Each step includes validation checks to ensure successful execution. For instance, the AMI of new instances is validated against the expected AMI ID.
- **Rollback for Add-ons Update:** If an add-on update fails, redeploy the previous version using eksctl with the old version specified.
- **Rollback for Node AMI Update:** If a node AMI update fails, revert the launch template to the previous version and adjust the ASG to use the previous launch template version.

## Usage
- **Create Workflow File:** Add the workflow file (eks-upgrade.yml) to the .github/workflows directory in your repository.
- **Configure Secrets:** Ensure AWS credentials are stored in GitHub Secrets.
- **Run Workflow:** Navigate to the "Actions" tab in your GitHub repository and manually trigger the workflow using workflow_dispatch.

## Notes
- **Sleep Time:** Adjust the sleep time in the ASG update loop based on your environment's instance launch times.
- **Manual Rollback:** In case of failures, manual rollback may be required, especially for Kubernetes version downgrades.

## Conclusion
This workflow provides a comprehensive solution for automating EKS cluster upgrades and node AMI updates, ensuring that each step is validated and instances are replaced safely. By using a configuration file, you can easily manage and update key variables, enhancing flexibility and maintainability.
