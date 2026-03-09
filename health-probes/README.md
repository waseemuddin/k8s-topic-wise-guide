# Kubernetes Health Probes

This repository explains **Health Probes in Kubernetes** with simple explanations and practical examples.

Health probes help Kubernetes check whether an application is **running properly**, **ready to receive traffic**, or **still starting**.

---

# 📌 Table of Contents

1. What are Health Probes?
2. Types of Health Probes
3. Liveness Probe
4. Readiness Probe
5. Startup Probe
6. Real World Use Case
7. Commands to Test
8. Best Practices

---

# 1️⃣ What are Health Probes?

Health Probes are **automatic health checks** used by Kubernetes to monitor containers.

They help Kubernetes determine:

* If the container is **alive**
* If the container is **ready to accept traffic**
* If the container has **successfully started**

If a problem is detected, Kubernetes can **restart the container** or **stop sending traffic** to it.

---

# 2️⃣ Types of Health Probes

Kubernetes provides three types of probes:

| Probe Type      | Purpose                                            |
| --------------- | -------------------------------------------------- |
| Liveness Probe  | Checks if the container is still running           |
| Readiness Probe | Checks if the container is ready to serve requests |
| Startup Probe   | Checks if the application has started successfully |

---

# 3️⃣ Liveness Probe

The **Liveness Probe** checks whether the application is still running.

If the probe fails, Kubernetes **restarts the container automatically**.

## Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80

    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
```

### How it works

* Kubernetes sends an HTTP request to `/`
* If the container fails to respond several times
* Kubernetes **restarts the container**

---

# 4️⃣ Readiness Probe

The **Readiness Probe** checks whether the container is ready to handle user requests.

If the probe fails:

* Kubernetes **removes the pod from the Service**
* Traffic will **not be sent** to that pod

## Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80

    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```

### How it works

* Kubernetes checks the application periodically
* If the app is not ready, the pod **will not receive traffic**

---

# 5️⃣ Startup Probe

The **Startup Probe** is used for applications that take **longer to start**.

It prevents Kubernetes from restarting containers during the startup phase.

## Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startup-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80

    startupProbe:
      httpGet:
        path: /
        port: 80
      failureThreshold: 30
      periodSeconds: 10
```

### How it works

* Kubernetes allows the application extra time to start
* If the probe keeps failing after the threshold, the container will restart

---

# 6️⃣ Real World Use Case

Example: A microservice application

| Component              | Probe Used           |
| ---------------------- | -------------------- |
| API Server             | Liveness + Readiness |
| Database Connection    | Readiness            |
| Large Java Application | Startup Probe        |

This ensures the application remains **stable and self-healing**.

---

# 7️⃣ Useful Commands

### Create Pod

```bash
kubectl apply -f pod.yaml
```

### Check Pod Status

```bash
kubectl get pods
```

### Describe Pod

```bash
kubectl describe pod <pod-name>
```

### Check Logs

```bash
kubectl logs <pod-name>
```

---

# 8️⃣ Best Practices

✔ Use **Readiness Probe** for applications receiving traffic
✔ Use **Liveness Probe** to recover from application crashes
✔ Use **Startup Probe** for slow-starting applications
✔ Avoid very short probe intervals to reduce overhead

---

# 📚 Summary

Health Probes make Kubernetes applications **self-healing and reliable**.

* **Liveness Probe** → Restart container if it crashes
* **Readiness Probe** → Control traffic to pods
* **Startup Probe** → Handle slow-starting applications

Using these probes ensures that applications stay **healthy and available in production environments**.

---

⭐ If you find this helpful, consider giving the repository a star.

