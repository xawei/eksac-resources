# AWS Clusters Helm Chart

This Helm chart deploys AWS EKS clusters and networking resources using Crossplane.

## Overview

This chart creates:
- **NetworkKcl**: Defines the VPC and subnet configuration
- **EksClusterKcl**: Defines the EKS cluster with all necessary components

## Prerequisites

- Crossplane installed and configured
- AWS Provider for Crossplane configured
- ArgoCD installed and configured
- Appropriate AWS permissions

## Installation

### ArgoCD Application (Recommended)

Create an ArgoCD Application to deploy this Helm chart:

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

### ArgoCD Application with Custom Values

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: aws-clusters
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://your-git-repo.com/path/to/repo
    targetRevision: HEAD
    path: aws-clusters
    helm:
      valueFiles:
        - values.yaml
      values: |
        global:
          owner: "my.user@company.com"
          awsAccountId: "123456789012"
        network:
          name: my-custom-network
        cluster:
          name: my-custom-cluster
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

### Direct Helm Installation (Alternative)

If you prefer to install directly with Helm:

```bash
helm install my-aws-cluster ./aws-clusters
```

```bash
helm install my-aws-cluster ./aws-clusters -f custom-values.yaml
```

## ArgoCD Configuration Notes

### Repository Structure
Make sure your git repository contains this Helm chart at the specified path. The typical structure would be:
```
your-repo/
├── aws-clusters/
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── templates/
│   └── README.md
└── ...
```

### Values Override Options
1. **Inline values**: Use the `values` field in the ArgoCD Application spec
2. **Multiple values files**: Reference different values files for different environments
3. **External values**: Store values in separate files within your repository

### Sync Policies
- **Automated sync**: Automatically sync when changes are detected in git
- **Self-healing**: Automatically correct drift between desired and actual state
- **Prune**: Remove resources that are no longer defined in git

## Configuration

The following table lists the configurable parameters and their default values:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `global.namespace` | Kubernetes namespace | `eksac` |
| `global.owner` | Owner of resources | `andyxinan.wei` |
| `global.location` | AWS region | `ap-southeast-1` |
| `global.awsAccountId` | AWS Account ID | `249140151390` |
| `network.name` | Network/VPC name | `andy-network` |
| `network.vpcCidr` | VPC CIDR block | `10.0.0.0/22` |
| `cluster.name` | EKS cluster name | `andy-cluster` |
| `cluster.crossplane.createDefaultAccessEntry` | Create default access entry | `false` |
| `cluster.crossplane.iamRoleName` | Crossplane IAM role name | `CrossPlaneRole` |

## Example Custom Values

### Single Environment
```yaml
global:
  namespace: my-namespace
  owner: "my.user@company.com"
  location: "us-west-2"
  awsAccountId: "123456789012"

network:
  name: my-network
  vpcCidr: "172.16.0.0/16"

cluster:
  name: my-cluster
  components:
    eks:
      version: "1.34"
    ciliumHelm:
      version: "1.18.0"
```

### Environment-Specific Values Files

For multiple environments, create separate values files:

**values-dev.yaml**
```yaml
global:
  namespace: eksac-dev
  owner: "dev.team@company.com"
  location: "ap-southeast-1"

network:
  name: dev-network
  vpcCidr: "10.1.0.0/22"

cluster:
  name: dev-cluster
```

**values-prod.yaml**
```yaml
global:
  namespace: eksac-prod
  owner: "ops.team@company.com"
  location: "ap-southeast-1"

network:
  name: prod-network
  vpcCidr: "10.2.0.0/22"

cluster:
  name: prod-cluster
  components:
    eks:
      version: "1.33"  # Stable version for prod
```

Then reference the appropriate values file in your ArgoCD Application:
```yaml
helm:
  valueFiles:
    - values.yaml
    - values-prod.yaml  # or values-dev.yaml
```

## Uninstallation

### ArgoCD Application

```bash
# Delete the ArgoCD Application
kubectl delete application aws-clusters -n argocd

# Or using ArgoCD CLI
argocd app delete aws-clusters
```

### Direct Helm (if installed via Helm)

```bash
helm uninstall my-aws-cluster
```

## Support

For issues and questions, please contact the maintainer or create an issue in the repository. 