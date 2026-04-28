# Jobs & CronJobs

## Jobs

A Job creates one or more pods and ensures they **run to completion** (exit code 0). Unlike Deployments, a Job does NOT restart a pod that succeeds — it's a one-time task.

### Use Cases
- Database migrations
- Batch processing (image resizing, report generation)
- Sending a one-time email blast
- Data import/export

### Basic Job
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-calculator
spec:
  template:
    spec:
      containers:
        - name: pi
          image: perl
          command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never     # Required for Jobs — must be Never or OnFailure
  backoffLimit: 4              # Retry up to 4 times on failure
```

### Parallel Jobs

Run multiple pods in parallel for batch processing:

```yaml
spec:
  completions: 10        # Total number of pods that need to succeed
  parallelism: 3         # How many pods run at the same time
```

| Setting | Meaning |
|---------|---------|
| `completions: 10` | Job completes when 10 pods have succeeded |
| `parallelism: 3` | Run 3 pods at a time |
| `backoffLimit: 4` | Max retries before marking the Job as failed |
| `activeDeadlineSeconds: 300` | Kill the Job if it runs longer than 5 minutes |

### Job Lifecycle

```
Job created → Pod created → Pod runs → Pod succeeds → Job complete
                                     → Pod fails → Retry (up to backoffLimit)
```

After a Job completes, the pods remain in `Completed` state so you can read their logs. They're NOT automatically cleaned up (use `ttlSecondsAfterFinished` for auto-cleanup).

```yaml
spec:
  ttlSecondsAfterFinished: 3600    # Delete Job + pods 1 hour after completion
```

---

## CronJobs

A CronJob creates Jobs on a **schedule**, using cron syntax.

### Use Cases
- Nightly database backups
- Hourly health checks
- Clean up old data every week
- Generate daily reports

### CronJob YAML
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-backup
spec:
  schedule: "0 2 * * *"           # Run at 2:00 AM every day
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: backup
              image: postgres:15
              command: ["pg_dump", "-h", "db-service", "-U", "admin", "mydb"]
          restartPolicy: OnFailure
  successfulJobsHistoryLimit: 3    # Keep last 3 successful Jobs
  failedJobsHistoryLimit: 1        # Keep last 1 failed Job
```

### Cron Schedule Syntax

```
┌───────────── minute (0-59)
│ ┌───────────── hour (0-23)
│ │ ┌───────────── day of month (1-31)
│ │ │ ┌───────────── month (1-12)
│ │ │ │ ┌───────────── day of week (0-6, Sun=0)
│ │ │ │ │
* * * * *
```

| Schedule | Meaning |
|----------|---------|
| `*/5 * * * *` | Every 5 minutes |
| `0 * * * *` | Every hour at :00 |
| `0 2 * * *` | Every day at 2:00 AM |
| `0 0 * * 0` | Every Sunday at midnight |
| `0 9 1 * *` | 1st of every month at 9 AM |

### Concurrency Policy

What happens if a new CronJob fires while the previous one is still running?

```yaml
spec:
  concurrencyPolicy: Forbid     # Skip the new run
```

| Policy | Behavior |
|--------|----------|
| `Allow` (default) | Multiple Jobs can run simultaneously |
| `Forbid` | Skip the new Job if previous is still running |
| `Replace` | Kill the running Job and start a new one |

## Key Commands

```bash
# Jobs
kubectl get jobs
kubectl describe job <name>
kubectl logs job/<name>
kubectl delete job <name>

# CronJobs
kubectl get cronjobs
kubectl describe cronjob <name>
kubectl create job manual-run --from=cronjob/<name>   # Trigger manually right now
```
