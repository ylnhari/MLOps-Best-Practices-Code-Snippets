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

## Step 2: Install Seldon Core
1. **Install Helm:**
  ```sh
  curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

2. Add Seldon Helm Repository and Install Seldon Core:
  ```sh
  Copy code
  helm repo add seldonio https://storage.googleapis.com/seldon-charts
  helm repo update
  helm install seldon-core seldonio/seldon-core-operator --namespace seldon-system --create-name
