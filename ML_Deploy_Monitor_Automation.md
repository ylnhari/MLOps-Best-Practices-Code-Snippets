# ML Model Deployment and Monitoring on GKE with Seldon Core, Prometheus, and Grafana

## Overview
This guide provides step-by-step instructions to set up an environment for deploying machine learning models on Google Kubernetes Engine (GKE) using Seldon Core for serving the models, Prometheus for metrics collection, and Grafana for visualization.

## Prerequisites
- Google Cloud Platform (GCP) account
- Google Kubernetes Engine (GKE) cluster
- Helm installed
- kubectl installed

## Step 1: Set Up Google Kubernetes Engine (GKE)
1. **Create a GKE Cluster:**
   - Go to the GCP Console.
   - Navigate to Kubernetes Engine > Clusters.
   - Click "Create cluster" and configure the cluster settings.
   - Once created, configure your local environment to interact with the cluster:
     ```sh
     gcloud container clusters get-credentials <cluster-name> --zone <zone> --project <project-id>
     ```

2. **Install kubectl:**
   ```sh
   gcloud components install kubectl
   ```

## Step 2: Install Seldon Core
1. **Install Helm:**
  ```sh
  curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
  ```

2. Add Seldon Helm Repository and Install Seldon Core:
  ```sh
  Copy code
  helm repo add seldonio https://storage.googleapis.com/seldon-charts
  helm repo update
  helm install seldon-core seldonio/seldon-core-operator --namespace seldon-system --create-name
  ```

## Step 3: Deploy a Model with Seldon
1. **Create a Seldon Deployment YAML file (seldon-deployment.yaml):**

   ```yaml
   apiVersion: machinelearning.seldon.io/v1
   kind: SeldonDeployment
   metadata:
     name: seldon-model
     annotations:
       prometheus.io/scrape: 'true'
       prometheus.io/path: /metrics
       prometheus.io/port: '8000'
   spec:
     name: example
     predictors:
     - graph:
         children: []
         implementation: SKLEARN_SERVER
         modelUri: gs://your-model-path
         name: classifier
       name: default
       replicas: 1
   ```

2. Apply the Deployment:
   ```sh
   kubectl apply -f seldon-deployment.yaml
   ```

## Step 4: Set Up Prometheus
1. **Install Prometheus using Helm:**

   ```sh
   helm install prometheus prometheus-community/prometheus
   Configure Prometheus to scrape Seldon metrics:
   ```

2. **Create or update the Prometheus config to include Seldon endpoints**
   ```yaml
   scrape_configs:
     - job_name: 'seldon'
       kubernetes_sd_configs:
         - role: endpoints
       relabel_configs:
         - source_labels: [__meta_kubernetes_service_label_seldon_io]
           action: keep
           regex: true
   ```
   
3. **Deploy Prometheus:**
   ```sh
   helm install prometheus prometheus-community/prometheus -f prometheus-values.yaml
   ```
