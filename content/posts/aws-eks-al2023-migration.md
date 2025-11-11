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
