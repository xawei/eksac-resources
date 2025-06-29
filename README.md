# EKSAC Resources

This repository contains ArgoCD applications designed to be deployed on a Kubernetes cluster with Crossplane installed. The resources in this repository are primarily **Crossplane Composite Resources (XRs)** that provision and manage AWS infrastructure.

## Overview

This repository serves as a GitOps source for managing AWS infrastructure through Crossplane using ArgoCD. Each directory contains Helm charts that define Crossplane resources for different infrastructure components.

## Prerequisites

- **Kubernetes cluster** with the following components installed:
  - **Crossplane** - For infrastructure provisioning
  - **ArgoCD** - For GitOps deployment
  - **AWS Provider for Crossplane** - Configured with appropriate permissions

## Repository Structure

```
eksac-resources/
├── aws-clusters/          # Helm chart for AWS EKS clusters and networking
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── templates/
│   └── README.md
└── README.md             # This file
```

## What are XR Resources?

The resources in this repository are **Crossplane Composite Resources (XRs)**. These are:

- **High-level abstractions** that define complex infrastructure patterns
- **Composed of multiple cloud resources** managed as a single unit  
- **Declarative** - you specify what you want, Crossplane handles the how
- **Kubernetes-native** - managed through standard kubectl and GitOps workflows

## Usage

### Deploy via ArgoCD

Each directory in this repository can be deployed as an ArgoCD Application. For example, to deploy the AWS clusters:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: aws-clusters
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/xawei/eksac-resources.git
    targetRevision: main
    path: aws-clusters
    helm:
      valueFiles:
        - values.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: eksac
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

### GitOps Workflow

1. **Modify values** in the respective Helm chart's `values.yaml` file
2. **Commit changes** to this repository
3. **ArgoCD automatically syncs** the changes to your cluster
4. **Crossplane provisions** the corresponding AWS infrastructure

## Available Components

### AWS Clusters (`/aws-clusters`)

Helm chart containing:
- **NetworkKcl** - VPC and networking configuration
- **EksClusterKcl** - EKS cluster with managed components

See the [aws-clusters README](./aws-clusters/README.md) for detailed configuration options.
