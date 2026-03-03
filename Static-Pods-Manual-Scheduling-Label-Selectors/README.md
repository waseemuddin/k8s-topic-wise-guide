# Kubernetes Core Concepts

## Static Pods, Manual Scheduling, Labels & Selectors

This repository explains important Kubernetes concepts with practical
examples.

------------------------------------------------------------------------

# 📌 Table of Contents

1.  Static Pods
2.  Manual Scheduling
3.  Labels
4.  Selectors
5.  Real-World Use Cases
6.  Commands Reference
7.  Best Practices

------------------------------------------------------------------------

# 1️⃣ Static Pods

## What are Static Pods?

Static Pods are Pods managed directly by the kubelet instead of the
Kubernetes API Server.

They are created by placing manifest files inside:

/etc/kubernetes/manifests/

The kubelet automatically detects the file and creates the Pod.

------------------------------------------------------------------------

## Why Use Static Pods?

-   Used for Kubernetes control plane components
-   Ensures critical services always run
-   Useful when API server is not available

Examples in kubeadm clusters: - kube-apiserver -
kube-controller-manager - kube-scheduler

------------------------------------------------------------------------

## Example: Static Pod

Create a file:

/etc/kubernetes/manifests/nginx-static.yaml

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-static
spec:
  containers:
  - name: nginx
    image: nginx
```

------------------------------------------------------------------------

# 2️⃣ Manual Scheduling

## What is Manual Scheduling?

Normally, Kubernetes Scheduler assigns Pods automatically.

Manual scheduling allows you to assign a Pod to a specific node using:

nodeName field

------------------------------------------------------------------------

## Why Use Manual Scheduling?

-   Debugging
-   Testing
-   Running workload on GPU node
-   Special hardware requirements

------------------------------------------------------------------------

## Example: Manual Scheduling

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: manual-pod
spec:
  nodeName: worker-node1
  containers:
  - name: nginx
    image: nginx
```

This Pod will run only on worker-node1.

------------------------------------------------------------------------

# 3️⃣ Labels

## What are Labels?

Labels are key-value pairs attached to Kubernetes objects.

They are used to organize and group resources.

------------------------------------------------------------------------

## Example:

``` yaml
metadata:
  labels:
    app: frontend
    env: production
```

------------------------------------------------------------------------

## Why Use Labels?

-   Organize resources
-   Select Pods for Services
-   Manage Deployments
-   Environment separation (dev, staging, prod)

------------------------------------------------------------------------

## Add Label to Node

``` bash
kubectl label nodes worker-node1 disk=ssd
```

------------------------------------------------------------------------

# 4️⃣ Selectors

## What are Selectors?

Selectors are used to filter resources based on labels.

They connect:

-   Service → Pods
-   Deployment → Pods
-   ReplicaSet → Pods

------------------------------------------------------------------------

## Example: Service Using Selector

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
```

The Service routes traffic to Pods with label:

app=frontend

------------------------------------------------------------------------

# 5️⃣ Real-World Use Cases

  Scenario                     Concept Used
  ---------------------------- -------------------
  Control plane components     Static Pods
  Run Pod on specific node     Manual Scheduling
  Separate dev and prod apps   Labels
  Connect Service to Pods      Selectors
