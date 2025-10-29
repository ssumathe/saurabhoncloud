# EKS RBAC per Namespace Example — Isolated Access for Multiple Users

This guide explains how to set up **RBAC in EKS (Amazon Elastic Kubernetes Service)** so that:
- `user-1` can access resources only in namespace `ns-1`
- `user-2` can access resources only in namespace `ns-2`

Both users will log in to a **bastion VM** and run `kubectl` commands from there.

---

## Overview

We'll configure:

1. **AWS IAM** users and map them to Kubernetes identities using the `aws-auth` ConfigMap.
2. **Kubernetes Roles and RoleBindings** for namespace-level access control.
3. **Bastion setup** so each Linux user’s `kubectl` context is limited to their namespace.

---

## Step 1: Create IAM Principals

Create IAM users for each person who will access the cluster:

```bash
aws iam create-user --user-name user1-iam
aws iam create-access-key --user-name user1-iam

aws iam create-user --user-name user2-iam
aws iam create-access-key --user-name user2-iam
```

Attach a minimal IAM policy for EKS authentication:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EKSReadAndGetToken",
      "Effect": "Allow",
      "Action": [
        "eks:ListClusters",
        "eks:DescribeCluster",
        "sts:GetCallerIdentity"
      ],
      "Resource": "*"
    }
  ]
}
```

Each user stores credentials in their Linux home directory:

```bash
~/.aws/credentials
[default]
aws_access_key_id = ABC...
aws_secret_access_key = XYZ...
region = ap-south-1
```

---

## Step 2: Map IAM Users in `aws-auth` ConfigMap

As an EKS admin, edit the `aws-auth` ConfigMap:

```bash
kubectl edit configmap aws-auth -n kube-system
```

Add mappings:

```yaml
mapUsers: |
  - userarn: arn:aws:iam::123456789012:user/user1-iam
    username: user1
    groups:
      - eks-ns-1-group
  - userarn: arn:aws:iam::123456789012:user/user2-iam
    username: user2
    groups:
      - eks-ns-2-group
```

This links AWS IAM identities to Kubernetes users/groups.

---

## Step 3: Create Namespace-Level RBAC

Create namespaces:

```bash
kubectl create namespace ns-1
kubectl create namespace ns-2
```

Then, define a `Role` for `ns-1`:

```yaml
# role-ns-1.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: ns-1-developer
  namespace: ns-1
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "configmaps", "secrets"]
    verbs: ["get", "list", "watch", "create", "delete", "patch", "update"]
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets", "statefulsets"]
    verbs: ["get", "list", "watch", "create", "delete", "patch", "update"]
  - apiGroups: ["batch"]
    resources: ["jobs", "cronjobs"]
    verbs: ["get", "list", "watch", "create", "delete", "patch", "update"]
```

Bind it to the IAM group mapped in `aws-auth`:

```yaml
# rolebinding-ns-1.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: bind-ns-1-developer
  namespace: ns-1
subjects:
  - kind: Group
    name: eks-ns-1-group
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: ns-1-developer
  apiGroup: rbac.authorization.k8s.io
```

Repeat for `ns-2`, changing names and namespace references.

Apply the manifests:

```bash
kubectl apply -f role-ns-1.yaml
kubectl apply -f rolebinding-ns-1.yaml
kubectl apply -f role-ns-2.yaml
kubectl apply -f rolebinding-ns-2.yaml
```

---

## Step 4: Bastion VM Setup

Each Linux user on the bastion should:

1. Install AWS CLI v2 and `kubectl`
2. Add their AWS credentials in `~/.aws/credentials`
3. Run:

```bash
aws eks update-kubeconfig --region ap-south-1 --name my-cluster
```

This generates `~/.kube/config` using the AWS IAM identity.

Now `user-1` can access:

```bash
kubectl get pods -n ns-1   # Works
kubectl get pods -n ns-2   # Forbidden
```

And `user-2` behaves oppositely.

---

## Step 5: Security Recommendations

- Use IAM **roles** with short-lived STS tokens instead of static keys.
- Restrict SSH access to the bastion (consider AWS Session Manager).
- Enable **audit logs** in EKS for accountability.
- Use **AWS SSO / IAM Identity Center** if scaling to more users.

---

## Summary

✅ Each IAM user maps to a specific Kubernetes group via `aws-auth`.  
✅ Each group has namespace-scoped RBAC permissions.  
✅ Each Linux user on the bastion automatically authenticates with AWS IAM and is restricted to their namespace.

This ensures clean, auditable access control per namespace in your EKS cluster.
