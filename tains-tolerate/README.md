# Kubernetes Taints and Tolerations

## Overview

**Taints and Tolerations** are Kubernetes features used to control which Pods can be scheduled on specific Nodes.

They work together to ensure that **Pods are not scheduled on inappropriate Nodes** unless explicitly allowed.

* **Taint** → Applied to a Node to repel Pods.
* **Toleration** → Applied to a Pod to allow it to run on a tainted Node.

Think of it as:

```
Node: "Do not schedule Pods here!"
Pod: "I can tolerate that rule, so allow me."
```

---

# Table of Contents

1. What are Taints
2. What are Tolerations
3. Types of Taint Effects
4. Example 1: Basic Taint
5. Example 2: Pod with Toleration
6. Example 3: GPU Node Example
7. Real-world Use Cases
8. Useful Commands
9. Best Practices

---

# 1. What is a Taint?

A **Taint** is applied to a **Node** to prevent Pods from being scheduled on it.

It tells Kubernetes scheduler:

> Do not place Pods on this Node unless they tolerate the taint.

### Taint Structure

```
key=value:effect
```

Example:

```
gpu=true:NoSchedule
```

Meaning:

* Node is reserved for GPU workloads
* Only Pods with matching toleration can run there

---

# 2. What is a Toleration?

A **Toleration** is applied to a **Pod**.

It allows the Pod to **bypass the Node restriction created by a taint**.

Example toleration in a Pod:

```yaml
tolerations:
- key: "gpu"
  operator: "Equal"
  value: "true"
  effect: "NoSchedule"
```

This means the Pod **can run on nodes tainted with `gpu=true:NoSchedule`**.

---

# 3. Taint Effects

Kubernetes supports three types of taint effects.

| Effect           | Meaning                                                 |
| ---------------- | ------------------------------------------------------- |
| NoSchedule       | Pod will NOT be scheduled on the node                   |
| PreferNoSchedule | Kubernetes tries to avoid scheduling but not guaranteed |
| NoExecute        | Pod will be evicted if already running                  |

---

# 4. Example 1: Apply Taint to Node

Apply a taint to a node:

```bash
kubectl taint nodes worker1 gpu=true:NoSchedule
kubectl taint nodes worker2 gpu=true:NoSchedule
```

Check taints on a node:

```bash
kubectl describe node worker1 | grep Taints
kubectl describe node worker2 | grep Taints
```

Now **no Pod will be scheduled on this node** unless it has a matching toleration.

---

# 5. Example 2: Pod with Toleration

Create a Pod that tolerates the taint.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: toleration-pod
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

Apply it:

```bash
kubectl apply -f pod.yaml
```

This Pod **can now run on the tainted node**.

---

# 6. Example 3: GPU Node Example

In production clusters, GPU nodes are usually reserved for AI/ML workloads.

### Add taint to GPU node

```bash
kubectl taint nodes gpu-node gpu=true:NoSchedule
```

### Pod that uses GPU node

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-workload
spec:
  containers:
  - name: cuda-container
    image: nvidia/cuda
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

Now only **GPU workloads can run on this node**.

---

# 7. Real-World Use Cases

### 1. Reserve GPU Nodes

Only AI/ML workloads run on GPU servers.

```
gpu=true:NoSchedule
```

---

### 2. Protect Control Plane Nodes

Prevent regular workloads from running on master nodes.

```
node-role.kubernetes.io/control-plane:NoSchedule
```

---

### 3. Dedicated Database Nodes

Reserve nodes only for database workloads.

```
db=true:NoSchedule
```

---

### 4. Special Hardware Nodes

Nodes with SSD, FPGA, or high memory.

```
high-memory=true:NoSchedule
```

---

# 8. Useful Commands

### Add Taint

```
kubectl taint nodes <node-name> key=value:effect
```

Example:

```
kubectl taint nodes worker1 gpu=true:NoSchedule
```

---

### Remove Taint

```
kubectl taint nodes worker1 gpu=true:NoSchedule-
```

---

### Check Node Taints

```
kubectl describe node <node-name>
```

---

### List Nodes

```
kubectl get nodes
```

---

# 9. Best Practices

✔ Use taints to reserve nodes for special workloads
✔ Combine taints with node labels for better scheduling
✔ Avoid overusing taints in small clusters
✔ Use tolerations carefully to prevent accidental scheduling

---

# Summary

| Component  | Purpose                                  |
| ---------- | ---------------------------------------- |
| Taint      | Restricts Pods from scheduling on a Node |
| Toleration | Allows Pods to run on a tainted Node     |

In simple terms:

```
Taint = Restriction on Node
Toleration = Permission for Pod
```

---

# References

Kubernetes Official Documentation:
https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/


#------------------------------------------------------------------------------------------------------------#

#### Kubernetes NodeSelector ####

#------------------------------------------------------------------------------------------------------------#


## Overview

**NodeSelector** is the simplest way to control **which Node a Pod should run on** in a Kubernetes cluster.

It works using **labels on Nodes**.

Kubernetes scheduler will only place the Pod on a Node that **matches the specified label**.

### Simple Idea

```text
Node Label → color=blue
Pod NodeSelector → color=blue
```

Result:

```text
Pod runs only on Node with label color=blue
```

---

# Table of Contents

1. What is NodeSelector
2. Why NodeSelector is Used
3. Step 1: Add Label to Node
4. Step 2: Create Pod Using NodeSelector
5. Example 1: Basic Example
6. Example 2: GPU Node Scheduling
7. Example 3: Database Node Scheduling
8. Useful Commands
9. Best Practices

---

# 1. What is NodeSelector?

**NodeSelector** is a field in the Pod specification that tells Kubernetes:

> "Schedule this Pod only on Nodes that have these labels."

Example:

```yaml
nodeSelector:
  disktype: ssd
```

This means the Pod will only run on nodes labeled:

```text
disktype=ssd
```

---

# 2. Why NodeSelector is Used

NodeSelector helps schedule workloads on specific nodes based on hardware or role.

Common use cases:

| Use Case          | Example                            |
| ----------------- | ---------------------------------- |
| GPU workloads     | Run ML workloads only on GPU nodes |
| Database nodes    | Run DB pods on dedicated servers   |
| SSD storage       | Schedule storage-heavy apps        |
| High-memory nodes | Run memory intensive apps          |

---

# 3. Step 1: Add Label to Node

First label the node.

Command:

```bash
kubectl label nodes worker1 disktype=ssd
```

Verify label:

```bash
kubectl get nodes --show-labels
```

Example output:

```text
worker1   Ready   disktype=ssd
worker2   Ready   disktype=hdd
```

---

# 4. Step 2: Create Pod Using NodeSelector

Create a Pod that selects the labeled node.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    disktype: ssd
```

Apply the Pod:

```bash
kubectl apply -f pod.yaml
```

Now Kubernetes schedules this Pod **only on nodes labeled `disktype=ssd`**.

---

# 5. Example 1: Basic NodeSelector

Label node:

```bash
kubectl label nodes worker1 environment=production
```

Pod YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: prod-app
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    environment: production
```

Result:

```text
Pod runs only on worker1
```

---

# 6. Example 2: GPU Node Scheduling

GPU nodes are often labeled like this:

```bash
kubectl label nodes gpu-node gpu=true
```

Pod YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-app
spec:
  containers:
  - name: cuda-container
    image: nvidia/cuda
  nodeSelector:
    gpu: "true"
```

Now the Pod **runs only on GPU nodes**.

---

# 7. Example 3: Database Node Scheduling

Label database node:

```bash
kubectl label nodes db-node database=true
```

Pod YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
spec:
  containers:
  - name: mysql
    image: mysql
  nodeSelector:
    database: "true"
```

This ensures **database workloads run only on database nodes**.

---

# 8. Useful Commands

### Label a Node

```bash
kubectl label nodes <node-name> key=value
```

Example:

```bash
kubectl label nodes worker1 disktype=ssd
```

---

### View Node Labels

```bash
kubectl get nodes --show-labels
```

---

### Remove Label

```bash
kubectl label nodes worker1 disktype-
```

---

### Check Pod Node Placement

```bash
kubectl get pods -o wide
```

---

# 9. Best Practices

✔ Use meaningful labels for nodes
✔ Combine NodeSelector with **Taints & Tolerations** for better control
✔ Avoid too many node labels in small clusters
✔ Use **Node Affinity** for more advanced scheduling

---

# Summary

| Feature      | Description                              |
| ------------ | ---------------------------------------- |
| Node Label   | Key-value pair assigned to nodes         |
| NodeSelector | Pod rule to select nodes based on labels |

Simple logic:

```text
Node Label → environment=production
Pod NodeSelector → environment=production
Result → Pod runs on that node
```

---

# References

Official Kubernetes Documentation

https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

