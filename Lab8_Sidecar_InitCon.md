# Lab: Init Containers and Sidecar Containers in Kubernetes (GKE)

---

## Banking Context

In banking applications, Pods often need:

* **Pre-checks before application starts** (config, data, dependency validation)
* **Supporting containers** for logging, monitoring, auditing, or security

Kubernetes provides two powerful patterns:

* **Init Containers** → Run *before* the main application
* **Sidecar Containers** → Run *along with* the main application

---

## Concept Overview

### Init Container

* Runs **before** main application containers
* Runs **sequentially**
* Must complete successfully before app starts
* Used for **setup, validation, pre-processing**

### Sidecar Container

* Runs **parallel** to main application
* Shares network & volumes with app container
* Used for **logging, monitoring, proxy, auditing**

---

## Lab Prerequisites

* GKE cluster up and running
* kubectl configured

Verify:

```bash
kubectl get nodes
```

---

## Lab Namespace

```bash
kubectl create namespace banking
```

---

# Lab 1: Init Container (Banking Startup Validation)

## Scenario

Before starting a **banking transaction service**, ensure:

* Configuration file is present
* Startup validation is completed

If validation fails → application should not start.

---

### Step 1: Create Pod with Init Container

Create file:

```bash
vi banking-init-container.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: transaction-service
  namespace: banking
spec:
  volumes:
  - name: shared-data
    emptyDir: {}

  initContainers:
  - name: init-config-check
    image: busybox
    command:
    - sh
    - -c
    - |
      echo "Validating banking configuration...";
      sleep 5;
      echo "config=validated" > /data/config.txt;
    volumeMounts:
    - name: shared-data
      mountPath: /data

  containers:
  - name: app-container
    image: busybox
    command:
    - sh
    - -c
    - |
      echo "Starting Transaction Processing Service";
      cat /data/config.txt;
      sleep 3600;
    volumeMounts:
    - name: shared-data
      mountPath: /data
```

---

### Step 2: Apply Pod

```bash
kubectl apply -f banking-init-container.yaml
```

---

### Step 3: Observe Init Container Execution

```bash
kubectl get pods -n banking
```

You will notice:

```
Init:0/1 → Running → Completed
```

---

### Step 4: View Init Container Logs

```bash
kubectl logs transaction-service -c init-config-check -n banking
```

Expected output:

```
Validating banking configuration...
```

---

### Step 5: View Application Container Logs

```bash
kubectl logs transaction-service -c app-container -n banking
```

Expected output:

```
Starting Transaction Processing Service
config=validated
```


---

# Lab 2: Sidecar Container (Banking Audit Logging)

## Scenario

A **transaction service** writes logs.
A **sidecar audit container** collects and prints those logs continuously.

---

### Step 1: Create Pod with Sidecar Container

Create file:

```bash
vi banking-sidecar.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: transaction-with-audit
  namespace: banking
spec:
  volumes:
  - name: log-volume
    emptyDir: {}

  containers:
  - name: transaction-app
    image: busybox
    command:
    - sh
    - -c
    - |
      while true; do
        echo "Transaction processed at $(date)" >> /logs/transactions.log;
        sleep 5;
      done
    volumeMounts:
    - name: log-volume
      mountPath: /logs

  - name: audit-sidecar
    image: busybox
    command:
    - sh
    - -c
    - |
      tail -f /logs/transactions.log
    volumeMounts:
    - name: log-volume
      mountPath: /logs
```

---

### Step 2: Apply Pod

```bash
kubectl apply -f banking-sidecar.yaml
```

---

### Step 3: Verify Pod Status

```bash
kubectl get pods -n banking
```

---

### Step 4: View Logs from Sidecar (Audit Logs)

```bash
kubectl logs transaction-with-audit -c audit-sidecar -n banking
```

Sample output:

```
Transaction processed at Tue Dec 15 11:30:01 IST
Transaction processed at Tue Dec 15 11:30:06 IST
```

---

### Step 5: View Logs from Main App

```bash
kubectl logs transaction-with-audit -c transaction-app -n banking
```
---

## Cleanup (Optional)

```bash
kubectl delete namespace banking
```

