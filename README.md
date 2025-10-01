# Microservices Demo with ArgoCD

This repository contains a microservices demo application split into individual Helm charts for ArgoCD deployment.

## Structure

```
microservices-demo/
├── charts/                    # Individual microservice charts
│   ├── frontend/             # Frontend service chart
│   ├── backend/              # Backend API chart  
│   ├── database/             # PostgreSQL database chart
│   ├── redis/                # Redis cache chart
│   └── worker/               # Background worker chart
├── argocd/                   # ArgoCD application manifests
│   ├── applications/         # Individual application manifests
│   └── environments/         # Environment-specific configs
└── umbrella/                 # Umbrella chart (optional)
```

## Microservices

- **Frontend**: Nginx serving static content
- **Backend**: Node.js API service
- **Database**: PostgreSQL database
- **Redis**: Redis cache
- **Worker**: Background processing service

## ArgoCD Deployment

Each microservice is deployed as a separate ArgoCD application, allowing for:
- Independent scaling and updates
- Service-specific configurations
- Environment-specific deployments
- Granular RBAC and monitoring

## Usage

Deploy individual services or use the ApplicationSet to deploy all services across environments.# microservices-demo
