# promstack-k8s
# Prometheus & Grafana Setup for UAT-Bastion Server

This guide documents the steps to set up **Prometheus** and **Grafana** on the **UAT Bastion Server** by replicating configurations from the **Production Bastion Server**.

---

## Step 1: Copy Required Files from Production Bastion to UAT Bastion

**1. Copy monitoring files from the production server:**

**Production Path:**
/root/gitops/apps/kong-prom

pgsql
Copy
Edit
- Copy the content of files in the above path from the **Prod server**.

**UAT Bastion Path:**
cd -UAT/promstack

yaml
Copy
Edit
- Create two files in this path and paste the content copied from the **Prod server**.

---

**2. Copy Ingress files for Prometheus & Grafana:**

**Production Path:**
cd /root/gitops/apps/promstack

markdown
Copy
Edit
- Copy the content of `prometheus-ingress.yaml` and `grafana-ingress.yaml` from the **Prod server**.

**UAT Bastion Path:**
cd -UAT/promstack

yaml
Copy
Edit
- Create the same ingress files (`prometheus-ingress.yaml` and `grafana-ingress.yaml`) and paste the copied content.

---

## Step 2: Install Prometheus Stack via Helm

From the `-UAT/promstack` path, go two levels up:
```bash
cd ../..
Add Prometheus Helm repository:

bash
Copy
Edit
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
Install Prometheus Stack:

bash
Copy
Edit
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace
Get Grafana service details:

bash
Copy
Edit
kubectl get svc -n monitoring prometheus-grafana
Retrieve Grafana admin password:

bash
Copy
Edit
kubectl get secret prometheus-grafana -n monitoring \
  -o jsonpath="{.data.admin-password}" | base64 --decode
Apply Grafana Ingress:

bash
Copy
Edit
cd /root/gitops/apps/promstack
kubectl apply -f grafana-ingress.yaml -n monitoring
Step 3: Configure Persistent Volume Claims (PVC)
Create a custom values file:

bash
Copy
Edit
cd -UAT/promstack
nano pvc-values.yaml
Sample custom-values.yaml content:

yaml
Copy
Edit
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

bash
Copy
Edit
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

The ingress configurations must be updated to reflect the correct domain names for UAT.

yaml
Copy
Edit

---

If you want, I can also **add diagrams** showing the setup flow between **Prod Bastion → UAT Bastion → Kuber

 
