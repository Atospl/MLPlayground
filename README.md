# ML Playground

MLflow deployment on Kubernetes.

## Prerequisites

- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Helm](https://helm.sh/docs/intro/install/)
- [minikube](https://minikube.sigs.k8s.io/docs/start/) (local only)

## Minikube (local)

```bash
# Start cluster
minikube start

# Add Helm repo
helm repo add community-charts https://community-charts.github.io/helm-charts
helm repo update

# Create namespace and artifact PVC
kubectl create namespace mlflow
kubectl apply -f k8s/mlflow-artifacts-pvc.yaml

# Deploy MLflow (with bundled PostgreSQL)
helm install mlflow community-charts/mlflow \
  --namespace mlflow \
  -f k8s/mlflow-values.yaml

# Access the UI
kubectl port-forward -n mlflow svc/mlflow 8080:80
# open http://localhost:8080
```

## External cluster

Switch your kubectl/Helm context to the target cluster:

```bash
# List available contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <CONTEXT_NAME>

# Verify
kubectl config current-context
```

Then follow the same steps as above with two changes:

1. **Use a managed database** instead of the in-cluster PostgreSQL.
   Override the relevant values at install time:

   ```bash
   helm install mlflow community-charts/mlflow \
     --namespace mlflow \
     -f k8s/mlflow-values.yaml \
     --set postgresql.enabled=false \
     --set backendStore.postgres.enabled=true \
     --set backendStore.postgres.host=<DB_HOST> \
     --set backendStore.postgres.port=5432 \
     --set backendStore.postgres.database=mlflow \
     --set backendStore.postgres.user=<DB_USER> \
     --set backendStore.postgres.password=<DB_PASSWORD>
   ```

2. **Switch the service type** if you need external access
   (e.g. `--set service.type=LoadBalancer`) or set up an Ingress.

