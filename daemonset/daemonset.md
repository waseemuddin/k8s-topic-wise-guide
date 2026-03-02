# Kubernetes DaemonSet Example

## 📌 What is DaemonSet?

A DaemonSet ensures that a copy of a Pod runs on every node in a Kubernetes cluster.

When a new node joins the cluster, the Pod is automatically deployed.
When a node is removed, the Pod is deleted.

---

## 🎯 Why Use DaemonSet?

DaemonSet is mainly used for:

- Log collection agents
- Monitoring agents
- Networking plugins
- Security agents

These services need to run once per node.

---

## 📂 Example: Nginx DaemonSet

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-daemonset
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
