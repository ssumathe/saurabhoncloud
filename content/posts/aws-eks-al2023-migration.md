---
title: "Migrating EKS Nodes from Amazon Linux 2 to Amazon Linux 2023 (AL2023)"
date: 2025-11-11T10:00:00+05:30
draft: false
description: "Learn how to migrate your EKS worker nodes from Amazon Linux 2 (AL2) to Amazon Linux 2023 (AL2023), set up custom launch templates, and handle IMDSv2 and IRSA configurations."
slug: "eks-migration-amazon-linux-2023"
keywords: ["EKS", "Amazon Linux 2023", "AL2023", "IRSA", "IMDSv2", "AWS Launch Template", "EKS Migration"]
tags: ["AWS", "EKS", "DevOps", "Amazon Linux 2023", "Kubernetes"]
categories: ["AWS", "EKS", "CloudOps"]
cover:
  image: "/images/aws-eks-al2023.png"
  alt: "EKS Amazon Linux 2023 Migration"
  caption: "Migrating EKS worker nodes from AL2 to AL2023"
---

## 🚀 AWS Launch Template with Amazon Linux 2023 Standard EKS Images

With **Amazon EKS 1.33 and later**, AWS has deprecated **Amazon Linux 2 (AL2)** EKS AMIs and made **Amazon Linux 2023 (AL2023)** the **default and mandatory base image**.  
This means all new EKS clusters and node groups must use AL2023 moving forward.

If you’re still using AL2-based worker nodes, now is the time to **migrate to AL2023** and reschedule your workloads to new nodes.

This guide walks you through the migration process, highlights **important configuration changes**, and shares **real-world fixes** for issues we faced during the transition.

---

## ⚙️ Key Features of Amazon Linux 2023 (AL2023)

###  1. IMDSv2 by Default
AL2023 enables **Instance Metadata Service v2 (IMDSv2)** — a more secure, token-based version of IMDS that prevents SSRF (Server-Side Request Forgery) attacks and unauthorized metadata access.

Unlike AL2, **pods in AL2023 nodes cannot directly access the EC2 instance’s IAM role credentials** via IMDS.

This introduces a stronger security boundary:
- Pods **cannot use node IAM roles directly**.
- AWS recommends using **IRSA (IAM Role for Service Account)** for pod-level IAM permissions.

---

##  Why You Must Use IRSA (IAM Roles for Service Accounts)

With AL2023:
- The EC2 IAM role assigned to the node is no longer accessible from pods.
- The recommended approach is to **assign IAM roles directly to pods** via **IRSA**.

This ensures **least privilege** and isolates permissions per pod.

Here’s how to annotate a service account with an IAM role:

```bash
kubectl annotate serviceaccount <SERVICE_ACCOUNT_NAME> \
  -n <NAMESPACE> \
  eks.amazonaws.com/role-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:role/<IAM_ROLE_NAME> \
  --overwrite
```
Then, restart your pods to pick up the new IAM role.

💡 Example:
If your pod needs access to an S3 bucket, attach an IAM role with the required S3 permissions to the service account.
Add this service account to the IAM role’s trust policy in AWS IAM.

⚙️ Recommended Configuration — Hop Count and Launch Templates

AWS recommends setting IMDS hop count = 2 for EKS nodes running AL2023.
To do this, you must use a custom launch template when creating your EKS node group.

🧾 You can also use the launch template to customize:

Storage type (gp2 → gp3)

Encryption settings

Network and security configurations

🧱 Example: Custom Launch Template with User Data

During our migration, we faced an issue where the EKS node group didn’t become active and threw the error:

failed to join the kubectl cluster

The fix was to include a bootstrap user_data script inside the launch template.
This ensures that the node registers properly with the EKS control plane during provisioning.

Below is the minimum required user_data content:
---
MIME-Version: 1.0
Content-Type: multipart/mixed; boundary="//"

--//
Content-Type: application/node.eks.aws

---
apiVersion: node.eks.aws/v1alpha1
kind: NodeConfig
spec:
  cluster:
    apiServerEndpoint: https://88C1DFBF289C6EXXXXXXE6F2D95A00.gr7.ap-south-1.eks.amazonaws.com
    certificateAuthority: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS
    cidr: 172.20.0.0/17
    name: eks-cdp1-p-ap-south-1-1
--//--
---

Once this template is configured, create or update your EKS managed node group using it.

Now scale up your new AL2023 node pool to the requried count once all the nodes are Ready scale down the AL2 node pool.

Summary of actions:

Create a custom launch template with hop count 2 and bootstrap user_data.

Use AL2023 standard EKS AMI.

Migrate workloads and ensure pods use IRSA-based service accounts.

Decommission AL2 node groups after validation.

If you encounter issues or need help debugging EKS node initialization, feel free to reach out at
📧 hello.saurabhoncloud@gmail.com
