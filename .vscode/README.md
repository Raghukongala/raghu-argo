# Banking Application - Kubernetes Project

## Overview
Production-ready Kubernetes deployment of a Banking Application on AWS EKS.

## Architecture
- **EKS Cluster**: ekswithavinash (ap-south-1), 2 worker nodes (t3.small)
- **ECR Image**: 796796207873.dkr.ecr.ap-south-1.amazonaws.com/banking-5:latest
- **App Port**: 5000 (Flask)

## Kubernetes Resources

| Resource | Name | Purpose |
|---|---|---|
| Namespace | dev-namespace | Isolate banking app resources |
| LimitRange | dev-limit-range | Enforce CPU/Memory limits per container |
| ConfigMap | banking-config | App configuration (DB host, port, env) |
| Secret | banking-secret | DB credentials (never hardcoded) |
| Deployment | banking-deployment | Stateless app, 3 replicas, RollingUpdate |
| Service | banking-service | LoadBalancer - external access |
| Service | banking-headless | Headless - direct pod access |
| DaemonSet | monitoring-agent | Node exporter on labeled nodes only |
| StatefulSet | banking-stateful | Stateful app with persistent EBS storage |
| Service | banking-stateful-headless | Headless service for StatefulSet |

## Setup

### 1. Create EKS Cluster
```bash
eksctl create cluster --name ekswithavinash --region ap-south-1 \
  --nodegroup-name banking-nodes --node-type t3.small --nodes 2 --managed
```

### 2. Install EBS CSI Driver
```bash
eksctl create addon --name aws-ebs-csi-driver --cluster ekswithavinash \
  --service-account-role-arn arn:aws:iam::796796207873:role/AmazonEKS_EBS_CSI_DriverRole
```

### 3. Label Nodes for DaemonSet
```bash
kubectl label node <node-name> monitor=true environment=production
```

### 4. Create Secret (never store in code)
```bash
kubectl create secret generic banking-secret \
  --from-literal=DB_USERNAME=bankinguser \
  --from-literal=DB_PASSWORD=SecurePass@123 \
  -n dev-namespace
```

### 5. Deploy Application
```bash
kubectl apply -f complete-app.yaml
```

## Key Concepts Demonstrated

- **RollingUpdate**: Zero downtime deployments (maxSurge: 2, maxUnavailable: 1)
- **LimitRange**: Prevents resource abuse per container
- **ConfigMap**: Externalized configuration, no hardcoding
- **Secrets**: Credentials managed outside codebase
- **DaemonSet + NodeSelector**: Monitoring only on labeled nodes
- **StatefulSet + PVC**: Data persists across pod restarts (EBS gp2, 5Gi per pod)
- **SecurityContext (fsGroup)**: Correct file permissions on mounted volumes
- **Headless Service**: Direct pod-to-pod DNS resolution

## Cleanup
```bash
kubectl delete -f complete-app.yaml
eksctl delete cluster --name ekswithavinash --region ap-south-1
```
