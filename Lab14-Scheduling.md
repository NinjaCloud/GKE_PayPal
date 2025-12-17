# Lab: Node Selector, Node Affinity, Taints and Tolerations in Kubernetes on GKE


## Lab Assumptions

* GKE cluster with **3 worker nodes**
* kubectl configured

Verify nodes:

```bash
kubectl get nodes
```

---

## Step 1: Label Nodes Based on Banking Use Case

We will logically dedicate nodes for different banking workloads.

### Step 1.1: View Node Names

```bash
kubectl get nodes --show-labels
```

Assume node names:

* node-1
* node-2
* node-3

---

### Step 1.2: Apply Labels

```bash
kubectl label node node-1 workload=core-banking
kubectl label node node-2 workload=analytics
kubectl label node node-3 workload=general
```

Verify:

```bash
kubectl get nodes --show-labels | grep workload
```

---

# Lab 1: Node Selector (Simple Scheduling)

## Scenario

Run a **core banking transaction service** only on nodes labeled `core-banking`.

---

### Step 2: Create Pod Using Node Selector

Create file:

```bash
vi node-selector-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: core-banking-app
spec:
  nodeSelector:
    workload: core-banking
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - |
      echo "Running Core Banking Service";
      sleep 3600;
```

---

### Step 3: Apply Pod and Verify Node

```bash
kubectl apply -f node-selector-pod.yaml
kubectl get pod core-banking-app -o wide
```

Observe the **NODE** column – pod runs only on `core-banking` node.

---

### Key Learning (Node Selector)

* Very simple scheduling
* Exact match only
* Suitable for **basic workload separation**

---

# Lab 2: Node Affinity (Advanced Scheduling)

## Scenario

Run **analytics or reporting jobs** on nodes meant for analytics, but allow flexibility.

---

### Step 4: Create Pod Using Node Affinity

Create file:

```bash
vi node-affinity-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: analytics-app
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: workload
            operator: In
            values:
            - analytics
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: workload
            operator: In
            values:
            - general
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - |
      echo "Running Banking Analytics Job";
      sleep 3600;
```

---

### Step 5: Apply and Verify

```bash
kubectl apply -f node-affinity-pod.yaml
kubectl get pod analytics-app -o wide
```

---

### Key Learning (Node Affinity)

* Supports **hard (required)** and **soft (preferred)** rules
* More flexible than nodeSelector
* Ideal for **banking analytics & reporting**

---

# Lab 3: Taints and Tolerations (Strict Isolation)

## Scenario

A **PCI-compliant node** should accept **only authorized banking workloads**.

---

### Step 6: Taint a Node

Taint the core banking node:

```bash
kubectl taint node node-1 pci=true:NoSchedule
```

Verify:

```bash
kubectl describe node node-1 | grep Taint
```

---

### Step 7: Try Scheduling Pod Without Toleration

```bash
kubectl apply -f node-selector-pod.yaml
kubectl get pods
```

Pod will remain in **Pending** state.

---

### Step 8: Add Toleration to Pod

Create file:

```bash
vi toleration-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pci-banking-app
spec:
  nodeSelector:
    workload: core-banking
  tolerations:
  - key: "pci"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - |
      echo "Running PCI-Compliant Banking Service";
      sleep 3600;
```

---

### Step 9: Apply and Verify

```bash
kubectl apply -f toleration-pod.yaml
kubectl get pod pci-banking-app -o wide
```

Pod now runs successfully on tainted node.

---


## Cleanup (Optional)

Remove taint:

```bash
kubectl taint node node-1 pci=true:NoSchedule-
```

Delete pods:

```bash
kubectl delete pod core-banking-app analytics-app pci-banking-app
```


