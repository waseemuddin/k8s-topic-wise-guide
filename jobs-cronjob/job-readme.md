# Kubernetes Job and CronJob Example

## 📌 What is Job?

A Kubernetes Job runs a task once and ensures it completes successfully.

Used for:
- Database migrations
- One-time backups
- Batch processing
- Script execution

---

## 🧪 Example: Simple Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: simple-job
spec:
  backoffLimit: 3
  template:
    spec:
      containers:
      - name: hello
        image: busybox
        command: ["echo", "Hello Kubernetes Job"]
      restartPolicy: Never

