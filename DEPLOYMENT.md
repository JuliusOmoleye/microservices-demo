# Deployment Guide

## Overview

This microservices demo provides multiple deployment options for ArgoCD:

1. **Individual Applications**: Deploy each service separately
2. **ApplicationSet**: Deploy all services across multiple environments
3. **Environment-specific**: Deploy to specific environments (dev/staging/prod)

## Prerequisites

- Kubernetes cluster with ArgoCD installed
- ArgoCD CLI (optional)
- kubectl access to the cluster

## Deployment Options

### Option 1: Individual Applications

Deploy each microservice as a separate ArgoCD application:

```bash
# Apply individual applications
kubectl apply -f argocd/applications/

# Or apply them one by one
kubectl apply -f argocd/applications/database.yaml
kubectl apply -f argocd/applications/redis.yaml
kubectl apply -f argocd/applications/backend.yaml
kubectl apply -f argocd/applications/frontend.yaml
kubectl apply -f argocd/applications/worker.yaml
```

### Option 2: ApplicationSet (Recommended)

Deploy all services across multiple environments using ApplicationSet:

```bash
# Deploy the ApplicationSet
kubectl apply -f argocd/applicationset.yaml

# This will create applications for all services in all environments:
# - frontend-dev, backend-dev, database-dev, redis-dev, worker-dev
# - frontend-staging, backend-staging, database-staging, redis-staging, worker-staging  
# - frontend-prod, backend-prod, database-prod, redis-prod, worker-prod
```

### Option 3: Environment-specific Deployment

Deploy to a specific environment by modifying the ApplicationSet generators:

```yaml
# Edit applicationset.yaml to deploy only to dev environment
generators:
- matrix:
    generators:
    - list:
        elements:
        - service: frontend
        - service: backend
        - service: database
        - service: redis
        - service: worker
    - list:
        elements:
        - environment: dev
          namespace: microservices-demo-dev
          repoURL: https://github.com/example/microservices-demo
          targetRevision: HEAD
```

## Verification

### Check Application Status

```bash
# List all applications
kubectl get applications -n argocd

# Check specific application
kubectl get application frontend -n argocd -o yaml

# Using ArgoCD CLI
argocd app list
argocd app get frontend
```

### Check Deployed Resources

```bash
# Check all resources in dev environment
kubectl get all -n microservices-demo-dev

# Check specific service
kubectl get pods -n microservices-demo-dev -l app.kubernetes.io/name=frontend

# Check services
kubectl get svc -n microservices-demo-dev
```

### Test Connectivity

```bash
# Port forward to frontend
kubectl port-forward -n microservices-demo-dev svc/frontend 8080:80

# Port forward to backend
kubectl port-forward -n microservices-demo-dev svc/backend 3000:3000

# Test backend health
curl http://localhost:3000/health
```

## Environment Configuration

### Development Environment
- **Namespace**: `microservices-demo-dev`
- **Resources**: Minimal (for development)
- **Persistence**: Disabled (uses emptyDir)
- **Replicas**: 1 per service
- **Autoscaling**: Disabled

### Staging Environment
- **Namespace**: `microservices-demo-staging`
- **Resources**: Medium (for testing)
- **Persistence**: Enabled with moderate storage
- **Replicas**: 2 per service
- **Autoscaling**: Enabled for frontend

### Production Environment
- **Namespace**: `microservices-demo-prod`
- **Resources**: High (for production workloads)
- **Persistence**: Enabled with large storage and fast SSDs
- **Replicas**: 3 per service
- **Autoscaling**: Enabled with conservative thresholds
- **TLS**: Enabled for ingress

## Customization

### Modify Environment Values

Edit the environment-specific values files:

```bash
# Edit dev environment frontend configuration
vi argocd/environments/dev/frontend-values.yaml

# Edit production database configuration
vi argocd/environments/prod/database-values.yaml
```

### Add New Environment

1. Create new environment directory:
```bash
mkdir -p argocd/environments/test
```

2. Create values files for each service:
```bash
cp argocd/environments/dev/* argocd/environments/test/
```

3. Update ApplicationSet to include new environment:
```yaml
- environment: test
  namespace: microservices-demo-test
  repoURL: https://github.com/example/microservices-demo
  targetRevision: HEAD
```

## Troubleshooting

### Application Sync Issues

```bash
# Check application status
argocd app get frontend-dev

# Force sync
argocd app sync frontend-dev

# Check events
kubectl get events -n microservices-demo-dev --sort-by='.lastTimestamp'
```

### Resource Issues

```bash
# Check pod logs
kubectl logs -n microservices-demo-dev deployment/frontend

# Check resource usage
kubectl top pods -n microservices-demo-dev

# Describe problematic pods
kubectl describe pod -n microservices-demo-dev -l app.kubernetes.io/name=backend
```

### Service Discovery Issues

```bash
# Check service endpoints
kubectl get endpoints -n microservices-demo-dev

# Test service connectivity from within cluster
kubectl run test-pod --rm -i --tty --image=busybox -- /bin/sh
# Inside the pod:
# nslookup backend.microservices-demo-dev.svc.cluster.local
# wget -qO- http://backend.microservices-demo-dev.svc.cluster.local:3000/health
```

## Cleanup

### Remove Specific Environment

```bash
# Delete all applications for dev environment
kubectl delete applications -n argocd -l environment=dev

# Delete namespace
kubectl delete namespace microservices-demo-dev
```

### Remove All Applications

```bash
# Delete ApplicationSet (this will remove all generated applications)
kubectl delete -f argocd/applicationset.yaml

# Or delete individual applications
kubectl delete -f argocd/applications/

# Delete namespaces
kubectl delete namespace microservices-demo-dev
kubectl delete namespace microservices-demo-staging
kubectl delete namespace microservices-demo-prod
```