# Prometheus & Grafana Setup for UAT-Bastion Server

This guide provides the steps to set up **Prometheus** and **Grafana** on the **UAT Bastion Server** by replicating configuration files from the **Production Bastion Server**.

---

## Step 1: Copy Required Files from Production to UAT Bastion

### 1. Copy monitoring configuration files
**Production Path:**
/root/gitops/apps/kong-prom

pgsql
Copy
Edit
- Copy the contents of the files in this directory from the **Prod server**.

**UAT Bastion Path:**
cd UEM-UAT/promstack

yaml
Copy
Edit
- Create two files in this path and paste the copied content from the **Prod server**.

---

### 2. Copy ingress files for Prometheus & Grafana
**Production Path:**
cd /root/gitops/apps/promstack

markdown
Copy
Edit
- Copy the content of:
  - `prometheus-ingress.yaml`
  - `grafana-ingress.yaml`

**UAT Bastion Path:**
cd UEM-UAT/promstack

yaml
Copy
Edit
- Create these same files and paste the content copied from the **Prod server**.

---

## Step 2: Install Prometheus Stack via Helm

From the `UEM-UAT/promstack` path, go two levels up:
```bash
cd ../..
Add Prometheus Helm repository:
----
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
Install Prometheus Stack:

Install Prometheus Stack:
-----
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace
Get Grafana service details:

Get Grafana service details:
----
kubectl get svc -n monitoring prometheus-grafana
Retrieve Grafana admin password:

Retrieve Grafana admin password:
----
kubectl get secret prometheus-grafana -n monitoring \
  -o jsonpath="{.data.admin-password}" | base64 --decode
Apply Grafana ingress:

Apply Grafana ingress:
-----
cd /root/gitops/apps/promstack
kubectl apply -f grafana-ingress.yaml -n monitoring
Step 3: Configure Persistent Volume Claims (PVC)
----
Create a custom values file:
------
cd UEM-UAT/promstack
nano pvc-values.yaml
Sample custom-values.yaml content:

prometheus:
  prometheusSpec:
    storageSpec:
      volumeClaimTemplate:
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 10Gi       # Adjust as needed
          storageClassName: gp2   # Optional: specify storage class (e.g., 'gp2' for AWS)

grafana:
  persistence:
    enabled: true
    accessModes: ["ReadWriteOnce"]
    size: 5Gi
    storageClassName: gp2
Upgrade Helm release with PVC settings:

Upgrade Helm release with PVC settings:
----
helm upgrade prometheus prometheus-community/kube-prometheus-stack \
  -f custom-values.yaml \
  -n monitoring
Verify PVCs and PVs:

bash
Copy
Edit
kubectl get pvc -n monitoring
Step 4: Apply Ingress Files
Apply the ingress configurations created in Step 1:

bash
Copy
Edit
kubectl apply -f grafana-ingress.yaml -n monitoring
kubectl apply -f prometheus-ingress.yaml -n monitoring
Notes
Ensure you have the required kubeconfig and permissions to apply these changes.

The storage size and storage class can be modified in custom-values.yaml to match your environment.

Update the ingress configurations with the correct UAT domain names before applying.

pgsql
Copy
Edit

---

If you copy and paste **this whole block** into a `README.md` file, it will be perfectly formatted on GitHub with code blocks, headings, and bullet points all in place.  

Do you want me to also **add a diagram** showing the flow from **Prod Bastion → UAT Bastion → Kubernetes Moni
