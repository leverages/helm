# PostgreSQL Helm Chart

This Helm chart deploys PostgreSQL database on Kubernetes with support for both local and cloud deployments.

## Features

- Uses official PostgreSQL Docker image from Docker Hub
- Supports two deployment modes:
  - **Local**: StatefulSet + Service + Secret for local deployment
  - **Cloud**: Deployment + PV + PVC + Service (LoadBalancer) + Secret for cloud deployment
- Configurable resources, persistence, and security settings
- Health checks and probes for reliability

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+

## Installing the Chart

### Local Deployment

```bash
helm install my-postgresql ./charts/postgresql \
  --set deploymentType=local \
  --set postgresql.auth.password=mypassword
```

### Cloud Deployment

```bash
helm install my-postgresql ./charts/postgresql \
  --set deploymentType=cloud \
  --set postgresql.auth.password=mypassword \
  --set cloud.service.type=LoadBalancer
```

## Configuration

The following table lists the configurable parameters and their default values.

### Global Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `deploymentType` | Deployment type: "local" or "cloud" | `"local"` |

### PostgreSQL Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `postgresql.image.repository` | PostgreSQL image repository | `postgres` |
| `postgresql.image.tag` | PostgreSQL image tag | `"16"` |
| `postgresql.image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `postgresql.auth.username` | PostgreSQL username | `postgres` |
| `postgresql.auth.password` | PostgreSQL password | `changeme` |
| `postgresql.auth.database` | PostgreSQL database name | `postgres` |
| `postgresql.auth.existingSecret` | Use existing secret for credentials | `""` |

### Local Deployment Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `local.replicaCount` | Number of replicas | `1` |
| `local.service.type` | Service type | `ClusterIP` |
| `local.service.port` | Service port | `5432` |
| `local.persistence.enabled` | Enable persistence | `true` |
| `local.persistence.size` | Storage size | `8Gi` |
| `local.persistence.storageClass` | Storage class | `""` |
| `local.resources.limits.cpu` | CPU limit | `500m` |
| `local.resources.limits.memory` | Memory limit | `512Mi` |

### Cloud Deployment Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `cloud.replicaCount` | Number of replicas | `1` |
| `cloud.service.type` | Service type | `LoadBalancer` |
| `cloud.service.port` | Service port | `5432` |
| `cloud.persistence.enabled` | Enable persistence | `true` |
| `cloud.persistence.size` | Storage size | `20Gi` |
| `cloud.persistence.storageClass` | Storage class | `""` |
| `cloud.persistence.persistentVolume.enabled` | Create PV | `true` |
| `cloud.persistence.persistentVolume.capacity` | PV capacity | `20Gi` |
| `cloud.resources.limits.cpu` | CPU limit | `1000m` |
| `cloud.resources.limits.memory` | Memory limit | `1Gi` |

## Examples

### Using with existing secret

```bash
kubectl create secret generic my-postgres-secret \
  --from-literal=POSTGRES_USER=myuser \
  --from-literal=POSTGRES_PASSWORD=mypassword \
  --from-literal=POSTGRES_DB=mydatabase

helm install my-postgresql ./charts/postgresql \
  --set postgresql.auth.existingSecret=my-postgres-secret
```

### Cloud deployment with specific storage class

```bash
helm install my-postgresql ./charts/postgresql \
  --set deploymentType=cloud \
  --set cloud.persistence.storageClass=fast-ssd \
  --set postgresql.auth.password=securepassword
```

## Connecting to PostgreSQL

After installation, you can connect to PostgreSQL using:

```bash
# Get the service name
kubectl get svc -l "app.kubernetes.io/name=postgresql"

# Forward port for local access
kubectl port-forward svc/my-postgresql 5432:5432

# Connect using psql
psql -h localhost -U postgres -d postgres
```

## Uninstalling the Chart

```bash
helm uninstall my-postgresql
```

## License

This chart is licensed under the Apache License 2.0.
