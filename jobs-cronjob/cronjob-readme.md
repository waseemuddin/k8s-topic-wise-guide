# Kubernetes CronJob Example

## 📌 What is a CronJob?

A CronJob in Kubernetes is used to run Jobs on a scheduled time, similar to a Linux cron.

It automatically creates a Job based on the defined schedule.

---

## 🎯 Why Use CronJob?

CronJob is used for:

- Daily database backups
- Scheduled report generation
- Log cleanup tasks
- Sending notification emails
- Periodic health checks

---

## ⏰ Cron Schedule Format

Cron format:
| | | | |
| | | | └── Day of week (0-6)
| | | └──── Month (1-12)
| | └────── Day of month (1-31)
| └──────── Hour (0-23)
└────────── Minute (0-59)



Example:

- `"*/5 * * * *"` → Every 5 minutes
- `"0 0 * * *"` → Every day at midnight
- `"0 2 * * 0"` → Every Sunday at 2 AM

---

## 🧪 Example: Simple CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: simple-cronjob
spec:
  schedule: "*/1 * * * *"   # Runs every 1 minute
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            command: ["echo", "Hello from Kubernetes CronJob"]
          restartPolicy: Never
