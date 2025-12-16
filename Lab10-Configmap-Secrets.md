# Kubernetes Lab: ConfigMap & Secret (Beginner Friendly)

This lab will teach:

* How to create a **ConfigMap**
* How to create a **Secret**
* How to use both in a Pod (environment variables + mounted files)

No RBAC or advanced concepts used.

---

## Prerequisites

* A working Kubernetes cluster (Minikube / KIND / GKE / EKS)
* `kubectl` configured

---

# ================================================

# Lab 1 — ConfigMap in Kubernetes

# ================================================

## 🧪 Objective

Create a ConfigMap and use it in a Pod as:

* Environment variables
* Mounted file

---

## Step 1 — Create a Namespace

```bash
kubectl create namespace cm-secret-lab
```

---

## Step 2 — Create a ConfigMap

Create a file `app-configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-config
  namespace: cm-secret-lab
data:
  APP_MODE: "production"
  APP_COLOR: "blue"
  welcome.txt: |
    Welcome to Kubernetes ConfigMap Lab!
    This text comes from a ConfigMap file.
```

Apply it:

```bash
kubectl apply -f app-configmap.yaml
```

Verify:

```bash
kubectl get configmap -n cm-secret-lab
kubectl describe configmap demo-config -n cm-secret-lab
```

---

## Step 3 — Create Pod Using the ConfigMap

Create this file `cm-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm-demo-pod
  namespace: cm-secret-lab
spec:
  containers:
  - name: nginx
    image: nginx:stable
    env:
    - name: MODE
      valueFrom:
        configMapKeyRef:
          name: demo-config
          key: APP_MODE
    - name: COLOR
      valueFrom:
        configMapKeyRef:
          name: demo-config
          key: APP_COLOR
    volumeMounts:
    - name: cm-volume
      mountPath: /etc/config
  volumes:
  - name: cm-volume
    configMap:
      name: demo-config
```

Apply:

```bash
kubectl apply -f cm-pod.yaml
```

Check:

```bash
kubectl get pod -n cm-secret-lab
```

---

## Step 4 — Validate the ConfigMap inside the Pod

Open a shell inside the pod:

```bash
kubectl exec -it cm-demo-pod -n cm-secret-lab -- bash
```

Check env variables:

```bash
echo $MODE
echo $COLOR
```

Check mounted file:

```bash
cat /etc/config/welcome.txt
```

Exit the pod:

```bash
exit
```

---

# ================================================

# Lab 2 — Secret in Kubernetes

# ================================================

## 🧪 Objective

Create a Secret and use it in a Pod as environment variables.

---

## Step 1 — Create a Secret

### Method 1 (recommended for beginners — plain YAML)

Create `app-secret.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: demo-secret
  namespace: cm-secret-lab
type: Opaque
data:
  username: YWRtaW4=      # base64 for "admin"
  password: cGFzc3dvcmQ=  # base64 for "password"
```

Apply:

```bash
kubectl apply -f app-secret.yaml
```

Verify:

```bash
kubectl get secret -n cm-secret-lab
kubectl describe secret demo-secret -n cm-secret-lab
```

---

## Step 2 — Create a Pod Using the Secret

Create `secret-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo-pod
  namespace: cm-secret-lab
spec:
  containers:
  - name: alpine
    image: alpine
    command: ["/bin/sh", "-c", "sleep 3600"]
    env:
    - name: USER_NAME
      valueFrom:
        secretKeyRef:
          name: demo-secret
          key: username
    - name: USER_PASSWORD
      valueFrom:
        secretKeyRef:
          name: demo-secret
          key: password
```

Apply:

```bash
kubectl apply -f secret-pod.yaml
```

Check status:

```bash
kubectl get pod -n cm-secret-lab
```

---

## Step 3 — Validate Secret in the Pod

Enter the pod:

```bash
kubectl exec -it secret-demo-pod -n cm-secret-lab -- sh
```

Check env variables (they will be decoded automatically):

```bash
echo $USER_NAME
echo $USER_PASSWORD
```

Exit:

```bash
exit
```

---

# ================================================

# Cleanup (optional)

# ================================================

```bash
kubectl delete namespace cm-secret-lab
```


