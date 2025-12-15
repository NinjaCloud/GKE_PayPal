# ResourceQuota lab — simple step-by-step (.md)

> Goal: teach how a `ResourceQuota` limits resource consumption **per namespace** in Kubernetes.
> Audience: beginners (not familiar with RBAC/ConfigMaps — simple examples only).
> Tested commands assume `kubectl` is configured to talk to a cluster.

---

## Prerequisites

* A Kubernetes cluster (minikube / kind / GKE / EKS etc.)
* `kubectl` configured and working
* Basic familiarity with `kubectl apply`, `kubectl get`, and YAML files

---

## Lab overview (what you'll do)

1. Create a namespace `rq-demo`.
2. Create a `ResourceQuota` that limits:

   * number of Pods
   * total CPU requests
   * total memory requests
3. Deploy simple nginx pods that stay **within** quota.
4. Try to create pods that **exceed** the quota and observe failures.
5. Inspect `ResourceQuota` usage and status.
6. Cleanup.

---

## 1 — Create the namespace

```bash
kubectl create namespace rq-demo
```

Verify:

```bash
kubectl get ns rq-demo
```

---

## 2 — Create a simple ResourceQuota

Create a file `resourcequota.yaml`:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: demo-quota
  namespace: rq-demo
spec:
  hard:
    pods: "2"                      # max 2 pods in this namespace
    requests.cpu: "1"              # total CPU requests (1 CPU)
    requests.memory: "1Gi"         # total memory requests (1 Gi)

```

Apply it:

```bash
kubectl apply -f resourcequota.yaml
```

Confirm:

```bash
kubectl get resourcequota -n rq-demo
kubectl describe resourcequota demo-quota -n rq-demo
```

---

## 3 — Create pods that **fit** within the quota

Create a file `nginx-small.yaml` (requests are small so two of these fit in the quota):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-small-1
  namespace: rq-demo
  labels:
    app: nginx-small
spec:
  containers:
  - name: nginx
    image: nginx:stable
    resources:
      requests:
        cpu: "250m"      # 0.25 CPU
        memory: "200Mi"  # 200 MiB
      limits:
        cpu: "500m"
        memory: "300Mi"
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-small-2
  namespace: rq-demo
  labels:
    app: nginx-small
spec:
  containers:
  - name: nginx
    image: nginx:stable
    resources:
      requests:
        cpu: "250m"
        memory: "200Mi"
      limits:
        cpu: "500m"
        memory: "300Mi"
```

Apply:

```bash
kubectl apply -f nginx-small.yaml
```

Check pods:

```bash
kubectl get pods -n rq-demo
```

Observe that both pods should become `Running`. The total requests used:

* CPU: 0.25 + 0.25 = 0.5 CPU (under 1)
* Memory: 200Mi + 200Mi = 400Mi (under 1Gi)
* Pod count: 2 (equals the `pods: "2"` quota)

---

## 4 — Try creating a pod that exceeds CPU/memory quota

Create `nginx-big.yaml` asking for large requests:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-big
  namespace: rq-demo
spec:
  containers:
  - name: nginx
    image: nginx:stable
    resources:
      requests:
        cpu: "800m"      # 0.8 CPU
        memory: "900Mi"  # 900 MiB
```

If you already used 0.5 CPU and 400Mi memory, creating this pod would request additional 0.8 CPU and 900Mi memory and exceed the `requests.cpu: "1"` and `requests.memory: "1Gi"` quota — the API server will reject it with a quota exceeded message.

Apply it:

```bash
kubectl apply -f nginx-big.yaml
```

Expected error message explaining which hard quota was exceeded.

---

## 5 — Inspect ResourceQuota usage and status

To see the current consumption vs quota:

```bash
kubectl get resourcequota demo-quota -n rq-demo -o yaml
```

Example fields to look for in the output:

* `status.hard` — the limits you set
* `status.used` — what the namespace is currently using (pods, cpu, memory, etc.)

Alternatively:

```bash
kubectl describe resourcequota demo-quota -n rq-demo
```

This shows `Used` and `Hard` values in human-readable form.

---

## 6 — Cleanup

```bash
kubectl delete namespace rq-demo
# or remove objects individually:
# kubectl delete -f nginx-small.yaml -n rq-demo
# kubectl delete -f resourcequota.yaml -n rq-demo
```


