# Environment-Specific Values Files

This directory contains environment-specific values files for the microservices demo application. Each environment (dev, staging, prod) has its own set of values files that override the default values in the Helm charts.

## Directory Structure

```
environments/
├── dev/
│   ├── backend-values.yaml
│   ├── database-values.yaml
│   ├── frontend-values.yaml
│   ├── redis-values.yaml
│   └── worker-values.yaml
├── staging/
│   ├── backend-values.yaml
│   ├── database-values.yaml
│   ├── frontend-values.yaml
│   ├── redis-values.yaml
│   └── worker-values.yaml
└── prod/
    ├── backend-values.yaml
    ├── database-values.yaml
    ├── frontend-values.yaml
    ├── redis-values.yaml
    └── worker-values.yaml
```

## Environment Configurations

### Development (dev)
- **Replicas**: 1 per service
- **Resources**: Minimal (50-100m CPU, 64-128Mi memory)
- **Features**: Debug mode enabled, detailed logging, development tools
- **Storage**: Small persistent volumes (1Gi for database)
- **Security**: Basic security settings

### Staging (staging)
- **Replicas**: 2 per service
- **Resources**: Medium (100-200m CPU, 128-256Mi memory)
- **Features**: Production-like settings, monitoring enabled
- **Storage**: Medium persistent volumes (5Gi for database)
- **Security**: Enhanced security settings

### Production (prod)
- **Replicas**: 3 per service
- **Resources**: High (200-500m CPU, 256-512Mi memory)
- **Features**: Full production settings, comprehensive monitoring
- **Storage**: Large persistent volumes (20Gi for database)
- **Security**: Maximum security settings, SSL enabled

## Key Differences by Environment

### Backend Service
- **Dev**: Debug logging, development features
- **Staging**: Production-like with monitoring
- **Prod**: Full security, rate limiting, CORS

### Database Service
- **Dev**: Small storage, no backups
- **Staging**: Medium storage, backups enabled
- **Prod**: Large storage, SSL enabled, optimized settings

### Frontend Service
- **Dev**: Hot reload, source maps, dev tools
- **Staging**: Production build, compression, caching
- **Prod**: Full security headers, CSP, HSTS

### Redis Service
- **Dev**: No persistence, debug logging
- **Staging**: Persistence enabled, monitoring
- **Prod**: Full persistence, security settings

### Worker Service
- **Dev**: Single worker, debug mode
- **Staging**: Multiple workers, production settings
- **Prod**: High concurrency, comprehensive monitoring

## Usage

These values files are automatically used by ArgoCD when deploying applications. The ApplicationSet configuration references these files using the pattern:

```yaml
helm:
  valueFiles:
    - "values.yaml"
    - "../../environments/{environment}/{service}-values.yaml"
```

## Maintenance

When updating environment-specific configurations:

1. Update the appropriate values file in the correct environment directory
2. Commit and push changes to the repository
3. ArgoCD will automatically detect changes and sync applications
4. Monitor application health after deployment

## Troubleshooting

If applications fail to deploy:

1. Check that the values files exist in the correct paths
2. Verify YAML syntax is correct
3. Ensure all required values are provided
4. Check ArgoCD application events for specific errors