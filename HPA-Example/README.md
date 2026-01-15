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

