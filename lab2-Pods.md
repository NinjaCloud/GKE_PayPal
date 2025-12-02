# Lab: Deploy Pods in Kubernetes (Imperative & Declarative) + Important kubectl Commands

**Goal:** Teach students how to create Pods using both *imperative* and *declarative* approaches, understand YAML basics, inspect resources, and practice core Kubernetes commands.

---

## Prerequisites

1. Access to a Kubernetes cluster (GKE Standard cluster recommended).
2. Access to Cloud Shell or local machine with `kubectl` configured.
3. Basic understanding of Kubernetes objects.

---

## Learning Objectives

* Create Pods using imperative commands
* Create Pods using YAML (declarative)
* Understand Pod lifecycle and metadata
* Use key `kubectl` commands to inspect, debug, and manage workloads

---

## Step 1 — Verify cluster access

Run:

```
kubectl get nodes
```

Expected output: list of nodes in **Ready** state.

---

## Step 2 — Create a Pod (Imperative Method)

### 2.1 Create a simple NGINX Pod

Run:

```
kubectl run nginx-pod --image=nginx --port=80
```

This creates a Pod without writing YAML.

### 2.2 Verify Pod creation

```
kubectl get pods
```

### 2.3 Get detailed Pod information

```
kubectl describe pod nginx-pod
```

### 2.4 View Pod logs (if applicable)

```
kubectl logs nginx-pod
```

---

## Step 3 — Create Pods using YAML (Declarative Method)

### 3.1 Create YAML manifest file

Create a YAML file named **nginx-pod.yaml**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-yaml
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

### 3.2 Apply YAML to create Pod

```
kubectl apply -f nginx-pod.yaml
```

### 3.3 Verify

```
kubectl get pods -o wide
```

### 3.4 Edit YAML and reapply (to show declarative nature)

Change image to `nginx:stable` and run:

```
kubectl apply -f nginx-pod.yaml
```

Kubernetes will update the Pod automatically.

---

## Step 4 — Delete Pods (Imperative & Declarative)

### Imperative delete

```
kubectl delete pod nginx-pod
```

### Declarative delete (using the YAML file)

```
kubectl delete -f nginx-pod.yaml
```


### 5.5 Namespace usage

```
kubectl get pods -n kube-system
kubectl create namespace demo
kubectl apply -f app.yaml -n demo
```

### 5.6 Generating YAML from imperative

```
kubectl run myapp --image=nginx --dry-run=client -o yaml
```

**End of Lab**
