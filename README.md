# promstack-k8s

Documentation on Prometheus and Grafana setup  for UAT-Bastion Server 
 

 

Step1:                                                                                                                                                                     First copy the files from bastion server to UAT bastion server: 

PATH = /root/gitops/apps/kong-prom 

1.Copy content from files in above path in prod server 

2.Now in UAT Bastion Server go this path : 

cd UEM-UAT/promstack 

3.Create two files and paste content of Prod server files here which copied above. 

4.Now in Prod server go to this path 

cd /root/gitops/apps/promstack 

5.Copy content of Prometheus and Grafana ingress files. 

6.Now in UAT-Bastion Server go to path  “UEM-UAT/promstack” 

7.Create Prometheus and Grafana ingress files and paste content  from PROD server ingress files 

Step 2 : 

Now come to root directory from “UEM-UAT/promstack” 

 cd ../.. 

 Add Prometheus Community Helm Repository 

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts 

helm repo update 

 Install Prometheus Stack using Helm 

helm install prometheus prometheus-community/kube-prometheus-stack \ 

  --namespace monitoring --create-namespace 

 

 Get Prometheus Grafana Service Details 

kubectl get svc -n monitoring prometheus-grafana 

 

 Retrieve Grafana Admin Password 

kubectl get secret prometheus-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode 

 

Apply the Ingress Configuration 

cd /root/gitops/apps/promstack 

kubectl apply -f grafana-ingress.yaml -n monitoring 

 

Step 3:For PVC 

1. Create a custom  values.yaml file: 

cd UEM-UAT/promstack 

nano custom-values.yaml 

prometheus: 

  prometheusSpec: 

    storageSpec: 

      volumeClaimTemplate: 

        spec: 

          accessModes: ["ReadWriteOnce"] 

          resources: 

            requests: 

              storage: 10Gi       # Change size as needed 

          storageClassName: gp2   # Optional: use your storage class (e.g., 'gp2' for AWS) 

 

grafana: 

  persistence: 

    enabled: true 

    accessModes: ["ReadWriteOnce"] 

    size: 5Gi 

    storageClassName: gp2         

 

2. Upgrade Existing Helm Release 

Since you've already installed the chart, use helm upgrade with the values file: 

helm upgrade prometheus prometheus-community/kube-prometheus-stack \ 

  -f custom-values.yaml \ 

  -n monitoring 

3.Verify PVCs and PVs Created 

kubectl get pvc -n monitoring 

Step 4: 

Apply ingress files which created above in step1 

Kubectl apply -f grafana-ingress.yaml -n monitoring 

Kubectl apply -f prometheus-ingress.yaml -n monitoring 

 

 

 
