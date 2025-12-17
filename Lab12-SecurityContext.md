# Lab: Security Context in Kubernetes on GKE 

## What is Security Context?

A **Security Context** defines privilege and access control settings for:

* **Pod level** (applies to all containers)
* **Container level** (overrides pod settings)

Commonly used fields:

* `runAsUser`
* `runAsGroup`
* `fsGroup`
* `runAsNonRoot`
* `allowPrivilegeEscalation`
* `readOnlyRootFilesystem`

---

## Lab Prerequisites

* GKE cluster up and running
* kubectl configured

Verify access:

```bash
kubectl get nodes
```

---

## Create Banking Namespace

```bash
kubectl create namespace banking
```

---

# Lab 1: Pod Running as Root (Insecure Banking App)

## Scenario

A legacy banking container runs as **root**, which is a security risk.

---

### Step 1: Create Pod Without Security Context

Create file:

```bash
vi insecure-banking-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: insecure-banking-app
  namespace: banking
spec:
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - |
      echo "Running as user:";
      id;
      sleep 3600;
```

---

### Step 2: Apply Pod

```bash
kubectl apply -f insecure-banking-pod.yaml
```

---

### Step 3: Check Container User

```bash
kubectl logs insecure-banking-app -n banking
```

Expected output:

```
uid=0(root) gid=0(root)
```

⚠️ This violates banking security best practices.

---

# Lab 2: Pod-Level Security Context (Non-Root Banking App)

## Scenario

Enforce **non-root execution** for a banking transaction service.

---

### Step 1: Create Pod with Pod-Level Security Context

Create file:

```bash
vi secure-banking-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-banking-app
  namespace: banking
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
    runAsNonRoot: true

  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - |
      echo "Running as user:";
      id;
      touch /data/testfile;
      sleep 3600;
    volumeMounts:
    - name: data-volume
      mountPath: /data

  volumes:
  - name: data-volume
    emptyDir: {}
```

---

### Step 2: Apply Pod

```bash
kubectl apply -f secure-banking-pod.yaml
```

---

### Step 3: Verify User and Group

```bash
kubectl logs secure-banking-app -n banking
```

Expected output:

```
uid=1000 gid=3000 groups=2000
```

---

# Lab 3: Container-Level Security Context (Read-Only Root FS)

## Scenario

Banking containers should not modify system files.

---

### Step 1: Create Pod with Container-Level Security Context

Create file:

```bash
vi readonly-banking-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readonly-banking-app
  namespace: banking
spec:
  containers:
  - name: app
    image: busybox
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
    command:
    - sh
    - -c
    - |
      echo "Trying to write to root filesystem";
      touch /testfile || echo "Write blocked";
      sleep 3600;
```

---

### Step 2: Apply Pod

```bash
kubectl apply -f readonly-banking-pod.yaml
```

---

### Step 3: Check Logs

```bash
kubectl logs readonly-banking-app -n banking
```

Expected output:

```
Write blocked
```

---



✅ This lab aligns Kubernetes security with **real banking compliance needs on GKE**.
