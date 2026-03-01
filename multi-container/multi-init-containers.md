# Multi-Container Pod in Kubernetes
## Sidecar vs Init Container

---

## What is a Multi-Container Pod?

In Kubernetes, a Pod can run multiple containers that:

- Share the same network namespace (same IP address)
- Share storage volumes
- Run on the same node
- Are tightly coupled

This pattern is used when containers need to work closely together.

---

# Sidecar Container

## Definition

A Sidecar container runs alongside the main container for the entire lifecycle of the Pod.

It starts with the main container and stops when the Pod stops.

---

## Key Characteristics

- Runs concurrently with main container
- Shares volumes and network
- Continuous execution
- Supports main container

---

## Common Use Cases

- Log collection
- Monitoring agents
- Service mesh proxy
- Config reloaders
- Reverse proxies

---

## Sidecar Example (Log Collector)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
spec:
  containers:
  - name: main-app
    image: nginx
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx

  - name: log-collector
    image: busybox
    command: ["sh", "-c", "tail -f /var/log/nginx/access.log"]
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx

  volumes:
  - name: shared-logs
    emptyDir: {}
```

Explanation:

- Both containers share the same volume.
- Main container writes logs.
- Sidecar container reads and processes logs.
- Both run simultaneously.

---

# Init Container

## Definition

An Init Container runs before the main container starts.

It must complete successfully before the main container begins execution.

---

## Key Characteristics

- Runs before main container
- Executes sequentially
- Runs only once per Pod start
- Must complete successfully

---

## Common Use Cases

- Wait for database readiness
- Perform database migrations
- Download configuration files
- Set file permissions
- Pre-populate shared volumes

---

## Init Container Example (Wait for Database)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-init
spec:
  initContainers:
  - name: wait-for-db
    image: busybox
    command:
      - sh
      - -c
      - >
        until nc -z db-service 3306;
        do echo waiting for database;
        sleep 2;
        done;

  containers:
  - name: main-app
    image: nginx
```

Explanation:

- Init container checks if database is reachable.
- Only after success, main container starts.
- If init container fails, Pod will not move to Running state.

---

# Sidecar vs Init Container (Comparison)

| Feature | Sidecar | Init Container |
|----------|----------|----------------|
| Runs when | With main container | Before main container |
| Lifecycle | Entire Pod lifecycle | Runs once |
| Continuous | Yes | No |
| Primary purpose | Support main app | Prepare environment |
| Failure effect | Pod may restart | Pod never starts |

---

# Important Interview Points

- Init containers run sequentially.
- Sidecars run concurrently.
- All containers in a Pod share the same IP address.
- They can share volumes.
- If an init container fails, the Pod enters CrashLoopBackOff.
- Init containers can use different images than main containers.
- Init containers can temporarily use elevated permissions.

---

# When to Use What?

Use Sidecar when:
- You need continuous background support.
- You need log shipping or monitoring.
- You use service mesh proxy.

Use Init Container when:
- You need setup before application starts.
- You must wait for dependency readiness.
- You need database migration before app launch.

---

# Summary

- Init Container = Preparation phase
- Sidecar Container = Continuous helper
- Both share network and storage
- Used in production-grade Kubernetes deployments

---

End of Notes