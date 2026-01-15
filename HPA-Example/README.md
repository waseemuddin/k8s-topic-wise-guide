# Kubernetes Metrics Server – Readiness Failure
## Operations, Configuration & Best Practices (README)

---

## 📌 Purpose

This repository/document provides a **practical and production-aware guide** for diagnosing, configuring, and operating **Kubernetes Metrics Server**, with a focus on resolving **Readiness Probe failures (HTTP 500)**.

It is intended to be used as:
- A **GitHub README**
- An **internal DevOps / SRE runbook**
- A **cluster configuration reference**

---

## 📖 Overview

**Metrics Server** is a cluster-wide resource metrics aggregator that collects CPU and memory usage from kubelets.

It is required for:

- `kubectl top nodes`
- `kubectl top pods`
- Horizontal Pod Autoscaler (HPA)
- Resource monitoring and capacity planning

Metrics Server runs in the `kube-system` namespace and communicates directly with kubelets on each node.

---

## ⚠️ Common Problem

### Readiness Probe Failure (HTTP 500)

Typical Kubernetes event:

```
Warning  Unhealthy  Readiness probe failed: HTTP probe failed with statuscode: 500
```

### Symptoms

- Pod status shows `Running` but `READY 0/1`
- Metrics Server logs show certificate or scrape errors
- `kubectl top nodes` / `kubectl top pods` do not work

---

## 🔍 Root Cause

In **self-managed or local Kubernetes clusters**, kubelets usually expose metrics using **self-signed TLS certificates**.

Metrics Server:
- Attempts to validate kubelet TLS certificates
- Fails verification
- Cannot scrape node metrics
- Returns HTTP 500 → Readiness probe fails

---

## 🧩 Affected Environments

| Environment | Impact |
|------------|--------|
| Minikube | ✅ Yes |
| kubeadm | ✅ Yes |
| Bare-metal | ✅ Yes |
| WSL2 | ✅ Yes |
| Docker Desktop Kubernetes | ✅ Yes |
| EKS / GKE / AKS | ❌ No |

---

## 🛠️ Diagnosis Steps

### 1️⃣ Check Pod Status

```bash
kubectl get pods -n kube-system
```

Expected failing state:

```
READY   0/1
STATUS  Running
```

### 2️⃣ Inspect Logs

```bash
kubectl logs -n kube-system deployment/metrics-server
```

Common error messages:

- `x509: certificate signed by unknown authority`
- `failed to scrape node`
- `unable to fetch metrics from kubelet`

---

## ✅ Recommended Fix (Best Practice for Local Clusters)

### When to Apply

✔ Development environments  
✔ Test / QA clusters  
✔ CI/CD Kubernetes labs  
✔ Learning & training setups  

🚫 **Do NOT use in regulated production environments**

---

## ⚙️ Configuration Change

Metrics Server must be configured to:
- Skip kubelet TLS verification
- Use the correct node IP resolution order

### Edit Deployment

```bash
kubectl edit deployment metrics-server -n kube-system
```

### Add These Arguments

```yaml
--kubelet-insecure-tls
--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
```

### Example Configuration

```yaml
containers:
- name: metrics-server
  args:
    - --cert-dir=/tmp
    - --secure-port=10250
    - --kubelet-insecure-tls
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
```

Kubernetes will automatically restart the pod after saving.

---

## 🔁 Validation

### Pod Readiness

```bash
kubectl get pods -n kube-system
```

Expected:

```
READY   1/1
STATUS  Running
```

### Metrics Availability

```bash
kubectl top nodes
kubectl top pods -A
```

If metrics are displayed, the issue is resolved.

---

## 🔐 Security Considerations

### Why TLS Verification Is Disabled

- Local clusters use self-signed certificates
- Managing kubelet CA trust is complex in non-managed clusters
- This configuration prioritizes **operational stability** over strict TLS enforcement

### Risk Assessment

| Risk | Impact |
|------|--------|
| Man-in-the-middle attack | Low (internal cluster network) |
| Unauthorized metrics access | Low |
| Compliance risk in production | High |

---

## 🏭 Production Guidance

For production clusters, use:

- Managed Kubernetes services (EKS, GKE, AKS)
- Proper kubelet CA and certificate configuration
- Full TLS verification enabled

❗ **Never use `--kubelet-insecure-tls` in security-sensitive or regulated production environments**

---

## 📋 Operational Checklist

- [ ] Metrics Server pod is `Ready`
- [ ] `kubectl top nodes` works
- [ ] `kubectl top pods -A` works
- [ ] HPA can retrieve metrics
- [ ] Configuration is documented
- [ ] Environment type validated (dev vs prod)

---

## 📄 Documentation Metadata

- **Audience:** DevOps / SRE / Platform Engineers
- **Document Type:** GitHub README / Operational Runbook
- **Scope:** Kubernetes Metrics Server
- **Recommended Location:** Repository root as `README.md`

---

## ✅ Summary

This README provides a **safe, repeatable, and well-documented approach** to fixing Metrics Server readiness issues in local and self-managed Kubernetes clusters while maintaining clear security boundaries for production usage.


---

# Horizontal Pod Autoscaler (HPA) — Practical Guide (README)

This section documents how to configure and use **Kubernetes Horizontal Pod Autoscaler (HPA)** using a clear, production-aligned example. It is written to be used directly as a **GitHub README.md** for learning, labs, and internal platform documentation.

---

## 📌 What is HPA?

**Horizontal Pod Autoscaler (HPA)** automatically scales the number of pod replicas in a Deployment, ReplicaSet, or StatefulSet based on observed metrics.

Most commonly used metrics:
- CPU utilization
- Memory utilization
- Custom / external metrics (advanced)

⚠️ HPA **requires Metrics Server** to be installed and healthy.

---

## 🧱 HPA Architecture Overview

```
User Traffic
     ↓
Service
     ↓
Deployment  ←── HPA Controller
     ↓               ↑
Pods  ←── Metrics Server ←── Kubelet
```

---

## ✅ Prerequisites

Before using HPA, verify metrics availability:

```bash
kubectl top nodes
kubectl top pods
```

If both commands return metrics, HPA is ready to use.

---

## 🚀 Example: CPU-based HPA (Recommended Start)

This example demonstrates automatic scaling of an **NGINX Deployment** based on CPU utilization.

---

## 1️⃣ Create Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-hpa-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-hpa
  template:
    metadata:
      labels:
        app: nginx-hpa
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
```

Apply:

```bash
kubectl apply -f deployment.yaml
```

⚠️ **CPU requests are mandatory for HPA**

---

## 2️⃣ Create Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-hpa-svc
spec:
  selector:
    app: nginx-hpa
  ports:
  - port: 80
    targetPort: 80
```

Apply:

```bash
kubectl apply -f service.yaml
```

---

## 3️⃣ Create HPA Resource

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-hpa-demo
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

Apply:

```bash
kubectl apply -f hpa.yaml
```

---

## 📊 Verify HPA Status

```bash
kubectl get hpa
```

Example output:

```
NAME        REFERENCE                  TARGETS   MINPODS   MAXPODS   REPLICAS
nginx-hpa  Deployment/nginx-hpa-demo  10%/50%   1         5         1
```

---

## 🔥 Generate Load (Trigger Scaling)

### Option 1: BusyBox Load Generator

```bash
kubectl run -i --tty load-generator --image=busybox -- /bin/sh
```

Inside the pod:

```sh
while true; do wget -q -O- http://nginx-hpa-svc; done
```

---

## 👀 Observe Auto-Scaling

```bash
kubectl get hpa -w
```

Pods will scale automatically:

```
1 → 2 → 3 → 4
```

Check pods:

```bash
kubectl get pods
```

---

## 📉 Scale Down Behavior

When traffic stops:
- CPU usage decreases
- HPA waits stabilization window
- Pods scale down automatically

This prevents frequent scaling fluctuations.

---

## ❌ Common Pitfalls

### Missing CPU Requests

Error:
```
failed to get CPU utilization
```

Fix:
```yaml
resources:
  requests:
    cpu: "100m"
```

---

### Metrics Server Not Available

Symptoms:
- HPA shows `unknown`

Fix:
- Ensure Metrics Server is running and ready

---

## 🏭 Production Best Practices

| Area | Recommendation |
|-----|---------------|
| Metric Type | CPU first, Memory carefully |
| minReplicas | ≥ 2 |
| maxReplicas | Load-tested |
| Resource Requests | Mandatory |
| Load Testing | Required |
| Monitoring | Prometheus + Alerts |

---

## 🧠 Key Takeaways

- Metrics Server is mandatory
- CPU-based HPA is the safest starting point
- Always define resource requests
- HPA works best for stateless workloads
- Test scaling behavior before production

---

## 📄 Documentation Metadata

- **Audience:** DevOps / SRE / Platform Engineers
- **Document Type:** GitHub README
- **Scope:** Kubernetes Horizontal Pod Autoscaler (HPA)

---

## ✅ Summary

This README provides a **clear, reproducible, and production-aware introduction** to Kubernetes HPA using a real, working example that can be directly applied in development and learning environments.

