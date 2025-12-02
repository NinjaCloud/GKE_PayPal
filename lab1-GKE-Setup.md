# Lab: Create a **Standard GKE (Google Kubernetes Engine)** Cluster using the Google Cloud Console (Web Portal)

**Goal:** Walk students through creating a zonal or regional Standard GKE cluster entirely through the Google Cloud Console (no local CLI required), deploy a sample NGINX workload, expose it with a LoadBalancer, verify monitoring/logging, then clean up.

---

## Lab prerequisites

1. Google Cloud account with billing enabled.
2. Console access with a user granted one of these roles in the target project: **Kubernetes Engine Admin** (`roles/container.admin`) or **Owner/Editor**. For network creation you may also need Compute Network Admin (`roles/compute.networkAdmin`).
3. Browser access to the Google Cloud Console ([https://console.cloud.google.com](https://console.cloud.google.com)) and permission to create projects (if creating a new project).
4. (Optional) Familiarity with basic Kubernetes concepts (Pod, Deployment, Service).

---

## Quick checklist before starting

* [ ] Project created and selected in Console.
* [ ] Billing enabled for the project.
* [ ] Kubernetes Engine API and Compute Engine API enabled (Console will prompt to enable if not).
* [ ] Enough quota for CPUs and external IP addresses in the chosen region/zone.

---

## Lab variables (choose values in the Console when prompted)

* **Project:** `your-project-id` (create or select from dropdown)
* **Location:** pick a zone (e.g., `us-central1-a`) for a zonal cluster or a region (e.g., `us-central1`) for a regional cluster
* **Cluster name:** `gke-standard-lab`
* **Network (optional):** `gke-lab-network` (you can use default or create a custom VPC)
* **Node machine type:** `e2-standard-4` (changeable)
* **Initial node count:** `3`

> Note: This lab shows the recommended VPC-native (alias IP) configuration.

---

## Step 1 — Open the Google Cloud Console & select or create a project

1. Open: [https://console.cloud.google.com](https://console.cloud.google.com)
2. From the project selector (top bar), choose an existing project or click **NEW PROJECT** to create one. Provide a project name and note the Project ID (you will use it in the console UI).
3. Ensure billing is enabled for the selected project (Console will show a banner if not).

---

## Step 2 — Enable required APIs (if prompted)

1. In Console, navigate to **APIs & Services > Library**.
2. Search for and enable:

   * **Kubernetes Engine API**
   * **Compute Engine API**
3. If you created a new project, the Console may show prompts and an "Enable APIs" button when you open Kubernetes Engine. Click it.

---

## Step 3 — (Recommended) Create a custom VPC and subnet via Console

Using a custom VPC gives you full control over IP ranges and is recommended for production-like labs.

1. From the left-hand menu, go to **VPC network > VPC networks**.
2. Click **CREATE VPC NETWORK**.

   * Name: `gke-lab-network`
   * Subnet creation mode: **Custom**
3. Click **+ ADD SUBNET** and provide:

   * Subnet name: `gke-lab-subnet`
   * Region: `us-central1` (or your preferred region)
   * IP address range: `10.10.0.0/16`
4. Leave other settings default and **Create** the network.

> If you prefer, you can use the `default` network; GKE will create necessary routes. For VPC-native alias IPs (recommended), you will set alias IP ranges when creating the cluster.

---

## Step 4 — Open Kubernetes Engine > Clusters and start "Create Cluster"

1. From the Console left menu, go to **Kubernetes Engine > Clusters**.
2. Click **CREATE** (or **Create cluster**).
3. Choose **Standard** (not Autopilot) when asked for cluster mode.

---

## Step 5 — Configure the cluster basics (Control plane)

On the **Create a cluster** page, configure:

1. **Name:** `gke-standard-lab` (or your chosen name)
2. **Location type:** choose **Zonal** or **Regional**

   * For zonal: pick a **Zone** (e.g., `us-central1-a`)
   * For regional (HA control plane): pick a **Region** (e.g., `us-central1`)
3. **Release channel:** you can choose **Regular** (recommended for labs) or leave default
4. **Kubernetes version:** Leave as default or pick the version shown (Console will suggest supported versions)

---

## Step 6 — Configure Networking & VPC-native (Alias IP)

1. Expand **Networking**.
2. **Network:** select `gke-lab-network` (or `default` if you did not create custom VPC).
3. **Subnetwork:** select `gke-lab-subnet` (if using custom VPC).
4. **VPC native (Enable IP alias):** ENABLE this option (recommended). This enables alias IPs for Pods and Services.
5. Optionally adjust secondary ranges (Console may auto-create ranges). Leave Console defaults for a lab unless you want precise CIDRs.

---

## Step 7 — Configure Node pool (default pool)

Scroll to **Node pools** or the **default-pool** section:

1. **Node pool name:** `default-pool`
2. **Location:** same as cluster (zone/region) or choose single zone for a zonal nodepool in regional cluster.
3. **Node image & OS:** Container-Optimized OS (COS) or Ubuntu — leave default unless you have special needs.
4. **Machine type:** choose `e2-standard-4` (or `e2-medium` for smaller labs).
5. **Nodes:** set **Number of nodes** to `3`.
6. (Optional) Enable **Autoscaling**: toggle `Enable autoscaling` and set min/max (e.g., min 1, max 5).
7. (Optional) Enable **Node auto-upgrade** and **Node auto-repair** for best practices.
8. Expand **Security** within the node pool to enable **Shielded GKE Nodes** if desired.

---

## Step 8 — Configure additional features (optional but recommended)

1. **Workload Identity:** On the main cluster creation page, locate **Security > Workload Identity** and enable it to allow pods to access GCP APIs securely.
2. **Cloud Monitoring & Logging:** Under **Operations**, ensure **Enable Cloud Operations for GKE** is ON (it usually is). This enables Cloud Monitoring & Logging for your cluster.
3. **HTTP Load Balancing / Ingress:** If you plan to create Ingress (multi-service HTTP routing), leave the default and enable L7 features later when creating an Ingress in workloads.

---

## Step 9 — Labels, tags, and review

1. Add any **Labels** or **Tags** you want for billing and organization (e.g., `env:lab`, `owner:instructor`).
2. Click **Create** at the bottom of the page.

> Creation can take several minutes. The Console will show status and eventually the new cluster row with **Running** state.

---

## Step 10 — Verify cluster & view details

1. On **Kubernetes Engine > Clusters**, click your cluster name `gke-standard-lab`.
2. Review the **Cluster details**: Control plane version, endpoint, networking, node pools, operations, and operations logs.
3. Click **Nodes** in the cluster view to inspect node pool nodes and their status.

---

## Step 11 — Deploy a sample NGINX workload using the Console (no CLI)

GKE Console provides a "Deploy" flow to create workloads without `kubectl`.

1. In your cluster page, click **Workloads** (left menu under the cluster view) or **Deploy** button.
2. Click **Deploy** (or **Deploy workload**).
3. Fill in the form:

   * **Deployment name:** `nginx-deployment`
   * **Image:** `nginx` (use `nginx:stable` or `nginx:latest`)
   * **Container port:** `80`
   * **Replicas:** `3`
4. Click **Show optional settings** if you need to set environment variables, resource limits, or labels.
5. Click **Deploy**.

You will see the workload created and the Pods starting (Console shows status: Running).

---

## Step 12 — Expose the nginx deployment as a LoadBalancer service (Console)

1. Open the **Workloads** page and click the `nginx-deployment` workload.
2. Click **Expose** or look for a button like **Create service** / **Service > Expose**.
3. In the Expose form:

   * **Service type:** `LoadBalancer`
   * **Port:** `80`
   * **Target port:** `80`
   * **Service name:** `nginx-service`
4. Click **Create** (or **Expose**).

The Console will create a Service and GCP will provision an external IP via a Network Load Balancer or GCP load balancer depending on configuration. In the **Services & Ingress** section you can watch the External IP column. It may take a minute to become available.

---

## Step 13 — Verify the workload externally

1. In **Services & Ingress**, locate `nginx-service`. Note the **External IP** assigned.
2. Open a new browser tab and `http://<EXTERNAL_IP>/` — you should see the NGINX welcome page.

> If you cannot reach it, check firewall rules and ensure the service created the forwarding rule. In GCP Console, go to **VPC network > Firewall > Firewall rules** to verify traffic is allowed to NodePort/LoadBalancer ports (default GKE setups handle this automatically).

---

## Step 14 — Inspect logs and metrics (Cloud Monitoring & Logging)

1. In Console open **Operations > Logging** and filter by **Kubernetes Container** to see pod logs. Select the `nginx-deployment` namespace and pods to view logs.
2. Open **Operations > Monitoring** to view default dashboards for Kubernetes clusters: resource usage, node CPU/Memory, pod metrics.
3. You can create an alert policy (if desired) to notify on CPU high usage.

---

## Step 15 — Optional: Configure an Ingress with HTTPS

1. If you want an L7 Ingress, create a Service of type `NodePort` or `ClusterIP` (for backend), then go to **Networking > Load balancing** or use **Services & Ingress > Create Ingress** to configure a HTTP(S) load balancer that forwards to your service.
2. To enable HTTPS, configure a managed certificate in **Network services > SSL Certificates** or use **Ingress > HTTPS** options in the GKE console; add domains and request a managed certificate.

---

## Step 16 — Clean up (delete resources via Console)

1. Delete the workload(s): **Kubernetes Engine > Workloads** → select `nginx-deployment` → **Delete**.
2. Delete service/Ingress: **Kubernetes Engine > Services & Ingress** → delete `nginx-service` and any Ingress.
3. Delete the cluster: **Kubernetes Engine > Clusters** → select `gke-standard-lab` → **Delete**.
4. (If created) Delete the custom VPC: **VPC network > VPC networks** → select `gke-lab-network` → **Delete**. Make sure there are no dependent resources (forwarding rules, subnets) before deletion.
5. (If you created a project just for the lab) **IAM & Admin > Settings** → **Shut down** project (or use Project settings menu → Shut down).

> Deleting resources promptly avoids unexpected charges.


**End of lab**
