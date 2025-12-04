Below is a **clean, simple `.md` lab** to teach **emptyDir** and **hostPath** for data persistence on Kubernetes.
No RBAC, no ConfigMaps — only very basic YAML so your banking client can easily understand.

---

# 🧪 Kubernetes Storage Lab

## Understanding `emptyDir` and `hostPath` Volumes (Beginner Friendly)

---

## 📌 **Lab Overview**

This lab explains two basic Kubernetes volume types:

### **1. `emptyDir` Volume**

* Storage created when a Pod starts
* Lives **as long as the pod lives**
* Data is deleted when the pod is removed
* Used for temporary data, caching, buffering, scratch space

### **2. `hostPath` Volume**

* Mounts a **directory from the node’s filesystem**
* Data **persists even if the pod is deleted**
* Used for logging, node-level storage, or daemon tools
* Not recommended for production clusters, but great for learning

This lab includes **hands-on examples** using Nginx.

---

## 🧰 **Prerequisites**

* Working Kubernetes cluster (GKE or Minikube or K3s)
* `kubectl` installed
* Basic knowledge of Pods and YAML

---

# -------------------------------------------------------

# 🧪 LAB PART 1: Understanding `emptyDir` Volume

# -------------------------------------------------------

## 🎯 **Goal**

Create a pod where:

* NGINX writes logs into a shared `emptyDir` volume
* Verify that data is **lost** when the pod is deleted

---

## Step 1 — Create Pod Using `emptyDir`

Create a YAML file:

```bash
vi nginx-emptydir.yaml
```

Paste:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-emptydir
spec:
  volumes:
  - name: cache-storage
    emptyDir: {}       # temporary storage
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: cache-storage
      mountPath: /usr/share/nginx/html
```

Apply:

```bash
kubectl apply -f nginx-emptydir.yaml
```

---

## Step 2 — Verify Pod and Volume

```bash
kubectl get pod nginx-emptydir
```

Enter the container:

```bash
kubectl exec -it nginx-emptydir -- bash
```

Inside:

```bash
echo "Hello from emptyDir" > /usr/share/nginx/html/test.txt
cat /usr/share/nginx/html/test.txt
```

Exit:

```bash
exit
```

---

## Step 3 — Delete Pod and Observe Data Loss

```bash
kubectl delete pod nginx-emptydir
```

Recreate the pod:

```bash
kubectl apply -f nginx-emptydir.yaml
```

Check inside again:

```bash
kubectl exec -it nginx-emptydir -- ls /usr/share/nginx/html
```

**Result:**
`test.txt` will NOT be present — **data lost**.
Great for demonstrating that **emptyDir is temporary**.

---

# -------------------------------------------------------

# 🧪 LAB PART 2: Understanding `hostPath` Volume

# -------------------------------------------------------

## 🎯 **Goal**

Create a pod where:

* Data is stored on the **node’s filesystem**
* Data persists **even after the pod is deleted**

---

## Step 1 — Identify the Node Name

```bash
kubectl get nodes
```

Note your node name (e.g., `gke-cluster-default-pool-12345`).

---

## Step 2 — SSH into the Node (GKE)

```bash
gcloud compute ssh <NODE_NAME> --zone <ZONE>
```

Create a directory on the node:

```bash
mkdir data
cd data
echo "Hey Good Morning Welcome to Kubernetes training" > index.html
```

Exit:

```bash
exit
```

---

## Step 3 — Create Pod Using `hostPath`

```bash
vi nginx-hostpath.yaml
```

Paste:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-hostpath
spec:
  volumes:
  - name: host-storage
    hostPath:
      path: <PathOfYourDir>  # node local storage
      type: DirectoryOrCreate
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: host-storage
      mountPath: /usr/share/nginx/html
  nodeName: <PasteNodeNamehere> 
```

Apply:

```bash
kubectl apply -f nginx-hostpath.yaml
```

---

## Step 4 — Write Data into hostPath Volume

```bash
kubectl exec -it nginx-hostpath -- bash
```

Inside:

```bash
echo "This is persistent hostPath data" > /usr/share/nginx/html/info.txt
cat /usr/share/nginx/html/info.txt
```

Exit:

```bash
exit
```

---

## Step 5 — Delete the Pod

```bash
kubectl delete pod nginx-hostpath
```

---

## Step 6 — Recreate Pod

```bash
kubectl apply -f nginx-hostpath.yaml
```

Verify:

```bash
kubectl exec -it nginx-hostpath -- cat /usr/share/nginx/html/info.txt
```

**Result:**
The file **still exists** → **data persisted** because it lived on the node, not in the pod.

---

# 🧹 Cleanup

```bash
kubectl delete -f nginx-emptydir.yaml
kubectl delete -f nginx-hostpath.yaml
```

