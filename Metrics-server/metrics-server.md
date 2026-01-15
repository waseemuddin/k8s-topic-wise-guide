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

This README provides a **safe, repeatable, and well-documented approach** to fixing Metrics Server readiness issues in local and self-managed Kubernetes clusters while maintaining clear security boundaries for p