# 🧪 Kubernetes DaemonSet Lab on GKE

### Deploying a Node Monitoring Agent (Node Exporter) on Every Node

---

## 📌 **Lab Overview**

This lab teaches the concept of **DaemonSets** in Kubernetes using a simple, real-world example:

> Deploying a lightweight **node monitoring agent** (node-exporter) on **every node** of a GKE cluster.

By the end of this lab, participants will understand:

* What a DaemonSet is
* Why it is used
* How it ensures **one pod per node**
* How DaemonSets behave during node additions/removal
* How to verify the agent is running on every node

---

## 🧰 **Prerequisites**

* Google Cloud Project with billing enabled
* Access to create GKE clusters
* `gcloud` and `kubectl` installed locally
* Basic Kubernetes knowledge: pods, deployments

---

## 🚀 **Step 1 — Create a GKE Cluster (Using Google Cloud Console)**

1. Open **Google Cloud Console**
2. Go to:
   **Navigation Menu ➜ Kubernetes Engine ➜ Clusters**
3. Click **Create**
4. Choose **Standard Cluster**
5. Configure:

   * **Name:** `gke-daemonset-lab`
   * **Location:** any preferred region/zone
   * **Node Pool Size:** 3 nodes
   * **Machine type:** `e2-medium` (or similar)
6. Click **Create**

Wait until the status becomes **RUNNING**.

---

## 🔗 **Step 2 — Connect to the GKE Cluster**

Open your terminal:

```bash
gcloud config set project <YOUR_PROJECT_ID>

gcloud container clusters get-credentials gke-daemonset-lab --zone <YOUR_ZONE>

kubectl get nodes
```

You should see 3 nodes.

---

## 📂 **Step 3 — Create a Namespace for the Lab**

```bash
kubectl create namespace daemonset-lab
kubectl config set-context --current --namespace=daemonset-lab
```

---

## 🧪 **Step 4 — Create a Simple DaemonSet (Node Exporter)**

This DaemonSet will run **one pod on each node** and expose basic system metrics.

Create a file:

```bash
vi node-exporter-daemonset.yaml
```

Paste the YAML:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  labels:
    app: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
          name: metrics
        resources:
          requests:
            cpu: "50m"
            memory: "64Mi"
          limits:
            cpu: "100m"
            memory: "128Mi"
```

Apply the DaemonSet:

```bash
kubectl apply -f node-exporter-daemonset.yaml
```

---

## 🔍 **Step 5 — Verify the DaemonSet Behavior**

### ✔️ Check DaemonSet

```bash
kubectl get daemonset
```

You should see something like:

```
node-exporter   DESIRED:3   CURRENT:3   READY:3
```

### ✔️ Check Pods (must be one per node)

```bash
kubectl get pods -o wide
```

Confirm:

* 3 nodes → 3 pods
* Each pod running on a different node (look at `NODE` column)

### ✔️ Describe DaemonSet

```bash
kubectl describe daemonset node-exporter
```

Explain key points:

* `desiredNumberScheduled`
* `currentNumberScheduled`
* `numberReady`
* update strategy

---

## 🌐 **Step 6 — Access Node Exporter Metrics (Optional Demo)**

Expose one of the node-exporter pods (temporarily):

```bash
kubectl expose pod <NameOfPod> --type LoadBalancer --port 9100 --target-port 9100
```

Open in browser:

```
http://<ExternanIPAddress>:9100
```

You will see raw node-level metrics like:

```
node_cpu_seconds_total
node_memory_MemFree_bytes
node_filesystem_size_bytes
```

This shows that the agent is **collecting real node metrics**.

---

## 💡 **Teaching Notes (Explain to the Client)**

* A **Deployment** runs pods based on replica count
* A **DaemonSet** runs pods **per node**
* Useful for:

  * Logging agents
  * Monitoring agents
  * Security agents
  * Network proxies
* When a node is added → DaemonSet automatically adds a pod
* When a node is removed → corresponding pod disappears

This behavior is **automatic and guaranteed** by the DaemonSet controller.

---

## 🧹 **Step 7 — Cleanup**

```bash
kubectl delete daemonset node-exporter
kubectl delete namespace daemonset-lab
```



