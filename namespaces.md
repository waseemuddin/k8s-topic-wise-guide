# Kubernetes Namespace Notes

## What is a Namespace?

A Namespace in Kubernetes is a logical partition inside a cluster used to organize and isolate resources.

It helps in:
- Environment separation (dev, staging, prod)
- Resource management
- Access control (RBAC)
- Avoiding name conflicts

---

## Why Do We Need Namespaces?

Imagine you are managing a Kubernetes cluster for:

- Development team
- Testing team
- Production environment
- Without namespaces:
- All Pods, Services, Deployments will exist in one place.
- Name conflicts can happen.
- Hard to manage access.
- Difficult to apply resource limits.

### With namespaces:
- Each team/environment is isolated.
- Same resource names can exist in different namespaces.
- Resource limits can be applied per namespace.
- Access control can be configured per namespace.

--- 

## Default Namespaces

| Namespace | Purpose |
|------------|----------|
| default | Default namespace |
| kube-system | System components |
| kube-public | Public readable resources |
| kube-node-lease | Node heartbeat tracking |

---

## Basic Commands

### Create Namespace
```bash
kubectl create namespace dev
```

### List Namespaces
```bash
kubectl get namespaces
```

### Delete Namespace
```bash
kubectl delete namespace dev
```

---

## Example: Same Pod Name in Different Namespaces

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: prod
spec:
  containers:
  - name: nginx
    image: nginx
```

This works because namespaces isolate resources.

---

## Resource Quota Example

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    pods: "5"
    requests.cpu: "2"
    requests.memory: 2Gi
```

---

## Deploy in Specific Namespace

```bash
kubectl apply -f deployment.yaml -n staging
```

Or inside YAML:

```yaml
metadata:
  namespace: staging
```

---

## Important Notes

- Namespaces isolate names, not networks.
- Deleting namespace deletes all resources inside it.
- Nodes and PersistentVolumes are NOT namespaced.
- Use NetworkPolicy for traffic isolation.

---

## When to Use Namespaces

✔ Multiple teams  
✔ Dev/Test/Prod separation  
✔ Resource quotas  
✔ RBAC management  

---

End of Notes
