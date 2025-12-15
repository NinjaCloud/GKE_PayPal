# Lab: Understanding Jobs and CronJobs in Kubernetes (GKE)

## Banking Use Case Context

Banks often run **one-time** or **scheduled background tasks** such as:

* End-of-Day (EOD) transaction reconciliation
* Daily interest calculation on savings accounts
* Nightly report generation
* Periodic cleanup of temporary transaction files

In Kubernetes:

* **Job** → Run a task **once** until completion
* **CronJob** → Run a task **on a schedule** (like a Linux cron)

---

## Lab Prerequisites

* GKE cluster already created (Standard or Autopilot)
* kubectl configured and connected to the cluster

Verify cluster access:

```bash
kubectl get nodes
```

---

## Lab 1: Kubernetes Job (One-Time Banking Task)

### Scenario

Run a **one-time End-of-Day transaction reconciliation job**.

---

### Step 1: Create a Namespace for Banking Apps

```bash
kubectl create namespace banking
```

```bash
kubectl get namespaces
```

---

### Step 2: Create Job YAML (EOD Reconciliation)

Create a file:

```bash
vi eod-reconciliation-job.yaml
```

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: eod-reconciliation-job
  namespace: banking
spec:
  backoffLimit: 3
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: reconciliation
        image: busybox
        command:
        - sh
        - -c
        - |
          echo "Starting End-of-Day Reconciliation";
          sleep 10;
          echo "Reconciling transactions...";
          sleep 10;
          echo "EOD Reconciliation Completed Successfully";
```

---

### Step 3: Apply the Job

```bash
kubectl apply -f eod-reconciliation-job.yaml
```

---

### Step 4: Verify Job Status

```bash
kubectl get jobs -n banking
```

Expected output:

```
NAME                        COMPLETIONS   DURATION   AGE
eod-reconciliation-job      1/1           20s        1m
```

---

### Step 5: Check Pod Created by Job

```bash
kubectl get pods -n banking
```

---

### Step 6: View Job Logs (Banking Output)

```bash
kubectl logs <pod-name> -n banking
```

Sample output:

```
Starting End-of-Day Reconciliation
Reconciling transactions...
EOD Reconciliation Completed Successfully
```

---

## Lab 2: Kubernetes CronJob (Scheduled Banking Task)

### Scenario

Run **daily interest calculation** for savings accounts at **midnight**.

---

### Step 1: Create CronJob YAML

Create a file:

```bash
vi daily-interest-cronjob.yaml
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-interest-calculation
  namespace: banking
spec:
  schedule: "0 0 * * *"
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 2
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: interest-calculator
            image: busybox
            command:
            - sh
            - -c
            - |
              echo "Starting Daily Interest Calculation";
              sleep 5;
              echo "Interest calculated for all savings accounts";
```

---

### Step 2: Apply the CronJob

```bash
kubectl apply -f daily-interest-cronjob.yaml
```

---

### Step 3: Verify CronJob

```bash
kubectl get cronjobs -n banking
```

Expected output:

```
NAME                         SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
daily-interest-calculation   0 0 * * *     False     0        <none>          1m
```

---

### Step 4: Verify Job and Pod

```bash
kubectl get jobs -n banking
kubectl get pods -n banking
```

---

### Step 5: View Logs

```bash
kubectl logs <pod-name> -n banking
```

Sample output:

```
Starting Daily Interest Calculation
Interest calculated for all savings accounts
```
---

## Cleanup (Optional)

```bash
kubectl delete namespace banking
```

---

