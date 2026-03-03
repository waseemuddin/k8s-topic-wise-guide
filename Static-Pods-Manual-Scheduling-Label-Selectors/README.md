# Kubernetes Core Concepts
## Static Pods, Manual Scheduling, Labels & Selectors

This repository explains important Kubernetes concepts with practical examples.

---

# 📌 Table of Contents

1. Static Pods
2. Manual Scheduling
3. Labels
4. Selectors
5. Real-World Use Cases
6. Commands Reference
7. Best Practices

---

# 1️⃣ Static Pods

## What are Static Pods?

Static Pods are Pods managed directly by the kubelet instead of the Kubernetes API Server.

They are created by placing manifest files inside:

/etc/kubernetes/manifests/

The kubelet automatically detects the file and creates the Pod.

---

## Why Use Static Pods?

- Used for Kubernetes control plane components
- Ensures critical services always run
- Useful when API server is not available

Examples in kubeadm clusters:
- kube-apiserver
- kube-controller-manager
- kube-scheduler

---

## Example: Static Pod

Create a file:

/etc/kubernetes/manifests/nginx-static.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-static
spec:
  containers:
  - name: nginx
    image: nginx
