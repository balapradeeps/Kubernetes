# Kubernetes Kustomize: Base and Overlays

This example shares common Kubernetes manifests in `k8s/base` and customizes replica counts and environment configuration in `k8s/overlays/{dev,stg,prod}`.

## Prerequisites
- `kubectl` installed
- Access to a Kubernetes cluster (for applying resources)

## Preview the rendered manifests
Run these commands from the directory containing this README:

```bash
kubectl kustomize k8s/overlays/dev
kubectl kustomize k8s/overlays/stg
kubectl kustomize k8s/overlays/prod
```

## Deploy to an environment
```bash
kubectl apply -k k8s/overlays/dev
kubectl apply -k k8s/overlays/stg
kubectl apply -k k8s/overlays/prod
```

## Check resources
```bash
kubectl get all -n dev
kubectl get all -n stg
kubectl get all -n prod
```

## Remove resources
```bash
kubectl delete -k k8s/overlays/dev
kubectl delete -k k8s/overlays/stg
kubectl delete -k k8s/overlays/prod
```

## Key idea
- `base/kustomization.yaml` lists shared resources.
- Each overlay references `../../base`.
- The `replicas` transformer changes replica counts without editing the base Deployment.
- `configMapGenerator` creates an environment-specific ConfigMap.

Note: the generated ConfigMap is included in the rendered output, but this sample does not wire it into the NGINX or Redis containers. Add `envFrom` or explicit `env` entries to a suitable application container if you want the application to consume those values. The `dev`, `stg`, and `prod` namespaces are created automatically by `kubectl apply -k` when the overlay specifies `namespace`? No—create them first if they do not already exist:

```bash
kubectl create namespace dev
kubectl create namespace stg
kubectl create namespace prod
```
