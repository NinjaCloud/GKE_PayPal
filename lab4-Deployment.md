
# **Lab 3: Kubernetes Deployment**

This lab guides you through creating, updating, rolling back, and scaling a Kubernetes Deployment using an NGINX application.

---

## -------------------------------------------------------------

## **Task 1: Create and Apply a Deployment YAML**

## -------------------------------------------------------------

### **Step 1: Create a Deployment manifest**

Create a file named `dep-nginx.yaml` and add the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-dep
  labels:
    app: nginx-dep
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx-ctr
        image: nginx:1.12.2
        ports:
        - containerPort: 80
```

### **Step 2: Apply the Deployment**

```bash
kubectl apply -f dep-nginx.yaml
```

### **Step 3: View the Kubernetes objects created**

```bash
kubectl get deployments
kubectl get rs
```

### **Step 4: Access a Pod and check NGINX version**

```bash
kubectl get pods
kubectl exec -it <pod_name> -- /bin/bash
```

Inside the pod:

```bash
nginx -v
exit
```

---

## -------------------------------------------------------------

## **Task 2: Update the Deployment with a New Image**

## -------------------------------------------------------------

### **Step 1: Update the image version**

This updates the container image to a newer version (nginx:1.11):

```bash
kubectl set image deployment/nginx-dep nginx-ctr=nginx:1.11
```

### **Step 2: Describe Deployment**

See how old pods are replaced with new ones:

```bash
kubectl describe deployments
```

### **Step 3: Verify the new Pod version**

```bash
kubectl get pods
kubectl exec -it <pod_name> -- /bin/bash
```

Inside the pod:

```bash
nginx -v
exit
```

---

## -------------------------------------------------------------

## **Task 3: Rollback a Deployment**

## -------------------------------------------------------------

### **Step 1: View rollout history**

```bash
kubectl rollout history deployment/nginx-dep
```

### **Step 2: Rollback to a previous revision**

```bash
kubectl rollout undo deployment/nginx-dep --to-revision=1
```

### **Step 3: Check ReplicaSets**

```bash
kubectl get rs
```

### **Step 4: Verify rollback by checking NGINX version**

```bash
kubectl get pods
kubectl exec -it <pod_name> -- /bin/bash
```

Inside the pod:

```bash
nginx -v
exit
```

---

## -------------------------------------------------------------

## **Task 4: Scaling the Deployment**

## -------------------------------------------------------------

### **Step 1: Check current replicas**

```bash
kubectl get deployments
kubectl get pods
```

### **Step 2: Scale up to 8 replicas**

```bash
kubectl scale deployment nginx-dep --replicas=8
```

### **Step 3: Verify scaling**

```bash
kubectl get deployments
kubectl get pods
```

### **Step 4: Scale down to 2 replicas**

```bash
kubectl scale deployment nginx-dep --replicas=2
```

### **Step 5: Confirm scale-down**

```bash
kubectl get deployments
kubectl get pods
```

