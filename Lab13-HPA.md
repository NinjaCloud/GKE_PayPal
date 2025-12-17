# Lab 13: Horizontal Pod Autoscaler (HPA)

## Objective
In this lab, you will learn how to configure and test **Horizontal Pod Autoscaler (HPA)** in Kubernetes.  
You will:
- Install the Metrics Server
- Deploy an application with CPU resource requests and limits
- Configure HPA to automatically scale pods based on CPU utilization
- Generate load to observe autoscaling behavior

---

## Prerequisites
- A running Kubernetes cluster (GKE / Minikube / any standard cluster)
- `kubectl` configured and connected to the cluster
- Basic understanding of Pods and Deployments

---

## Step 1: Install Metrics Server

HPA requires metrics such as CPU and memory usage. These metrics are provided by the **Metrics Server**.

Run the following command to install Metrics Server:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
````

Verify that Metrics Server pods are running:

```bash
kubectl get pods -n kube-system
```

Ensure the `metrics-server` pod is in `Running` state.

---

## Step 2: Create a Deployment

Create a deployment with CPU and memory **requests** and **limits**, which are mandatory for HPA.

Create a file named `hpa-deploy.yaml`:

```bash
vi hpa-deploy.yaml
```

Add the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          requests:
            cpu: 100m
            memory: 50Mi
          limits:
            cpu: 200m
            memory: 100Mi
```

### Explanation

* **replicas: 3** → Initial number of pods
* **cpu request (100m)** → Minimum CPU guaranteed
* **cpu limit (200m)** → Maximum CPU a pod can use
* HPA uses **CPU requests** as a reference to calculate utilization

---

## Step 3: Deploy the Application

Apply the deployment:

```bash
kubectl create -f hpa-deploy.yaml
```

Verify the deployment:

```bash
kubectl get deployments
```

Check resource usage of pods:

```bash
kubectl top pods
```

> If metrics are not visible, wait for a minute and retry.

---

## Step 4: Create Horizontal Pod Autoscaler

Create an HPA for the deployment using the following command:

```bash
kubectl autoscale deployment nginx-deployment \
  --min=3 \
  --max=6 \
  --cpu-percent=50
```

### Explanation

* **min=3** → Minimum number of pods
* **max=6** → Maximum number of pods
* **cpu-percent=50** → Target average CPU utilization per pod

---

## Step 5: Verify HPA

Check the HPA status:

```bash
kubectl get hpa
```

Describe the HPA to see detailed information:

```bash
kubectl describe hpa nginx-deployment
```

This shows:

* Current CPU utilization
* Target CPU utilization
* Current and desired number of replicas

---

## Step 6: Generate Load to Trigger Autoscaling

Login into one of the nginx pods:

```bash
kubectl exec -it <pod_name> -- bash
```

Run a CPU-intensive loop:

```bash
while true; do true; done
```

This command will increase CPU usage inside the container.

---

## Step 7: Observe Autoscaling

In a new terminal, monitor HPA and deployment scaling:

```bash
kubectl get hpa nginx-deployment
```

```bash
kubectl get deployments nginx-deployment
```

Describe HPA again to see scaling decisions:

```bash
kubectl describe hpa nginx-deployment
```

You should notice:

* CPU utilization crossing 50%
* Number of replicas increasing (up to 6)

---

## Step 8: Cleanup Resources

Stop the load by exiting the pod:

```bash
exit
```

Delete the HPA:

```bash
kubectl delete hpa nginx-deployment
```

Delete the deployment:

```bash
kubectl delete deployment nginx-deployment
```
