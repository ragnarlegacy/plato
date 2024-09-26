# GKE provisioning using Terraform

## Prerequisites
- [Google Cloud SDK](https://cloud.google.com/sdk/docs/install)
- [Terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli)
- [Helm](https://helm.sh/docs/intro/install/)
- [Docker](https://docs.docker.com/get-docker/)
- A GCP account with billing enabled
- Kubectl
- gke-gcloud-auth-plugin
  

## Setup Instructions

### 1. Configure GCP Project
1. Create a new GCP project.
2. Enable the Kubernetes Engine API and Compute Engine API.
3. Authenticate using the Google Cloud SDK:
```sh
gcloud auth application-default login

OR

gcloud auth login
gcloud config set project <your-gcp-project-id>
```

### 2. Create Google Cloud Storage and Service account to configure remote backend
Either, you can login to GCP console and create manually or follow the below given instructions.

1. Clone the repository:
```sh
git clone https://github.com/ragnarlegacy/GKE.git
cd plato/terraform
```

global.auto.tfvars:
```sh
# terraform.tfvars
cluster_name = "cluster-name"
project_id = "project-id"
region     = "region"
```
2. Initialize Terraform:
```sh
terraform init
```
3. Plan the Terraform Configuration
```sh
terraform plan -auto-approve
```

4. Apply the Terraform configuration:
```sh
terraform apply -auto-approve
```

### 3. Interact with the GKE Cluster

```   
After the cluster is created, you can connect to it using kubectl:
```sh
gcloud container clusters get-credentials YOUR_GKE_CLUSTER --region us-central1 --project YOUR_PROJECT_ID

kubectl cluster-info
kubectl get nodes
```
### 4. Deploy Kubernetes Workloads

```sh
kubectl apply -f kustomize
```
### 5. Test Backend Pod security 

```sh
kubectl exec -it <pod-name> -n project-plato /bin/sh
touch /tmp/data  ## /tmp is writable
touch /data ## write is not allowed at root location /
```
### Confirm the functionality of the ReadinessProbe.

```sh
kubectl describe pods pod-name -n project-plato
Look for readiness-probe in there   ## if faild connection is there it means pod is not ready to accept traffic 
```
### Confirm the functionality of the newly created NetworkPolicy.

```sh
kubectl exec -it backend-pod-name -- nc -zv db1-pod-name 6379
kubectl exec -it backend-pod-name -- nc -zv db1-pod-name 5432

It tries to connect and start streaming if network-policy is properly configured and 
otherwise command will terminated with connection timeout error
```

### Confirm the availability of the Secret contents in the Deployment.

```sh
kubectl describe pods db2-* -n project-plato ## you will secret having strings as key/values
```
### Deploy ‘Postgres’

```sh
* Add the Bitnami Helm Repository

helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

* Install PostgreSQL with Postgres Exporter Enabled
helm install my-postgresql bitnami/postgresql \
  --set metrics.enabled=true \
  --set metrics.serviceMonitor.enabled=true \
  --set metrics.serviceMonitor.namespace=project-plato
```

### Deploy kube-prometheus-stack

```sh
*Add Prometheus Community Helm Repository

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

* Install kube-prometheus-stack

helm install my-kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace project-plato --create-namespace
```

### Check Prometheus Connection

```sh
kubectl port-forward svc/my-kube-prometheus-stack-prometheus 9090:9090 -n project-plato
Open your browser and visit http://localhost:9090

* Query PostgreSQL Metrics 
pg_stat_activity_count
pg_up
```

### Grafana Dashboard Connection

```sh
kubectl port-forward svc/my-kube-prometheus-stack-grafana 3000:80 -n project-plato
open your browser and go to http://localhost:3000
default admin credentials:

Username: admin
Password: prom-operator (you can retrieve this using kubectl get secret if changed)
```


### Proposed Infra Diagram with Specific Tools and Technologies
```sh
Work with Proposed-Infra.drawio file
```

