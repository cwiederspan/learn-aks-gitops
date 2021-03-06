# Learn AKS GitOps

A repository for creating a GitOps-enabled AKS cluster in Azure.

## Prerequisites

### Azure CLI Prerequisites

As part of this demo, make sure all of the preview extensions are up to date.

```bash

# Add (or update) the Azure CLI extensions
az extension add --name aks-preview
az extension add --name k8sconfiguration

az extension update --name aks-preview
az extension update --name k8sconfiguration

# Install the GitOps feature
az feature register --namespace "Microsoft.ContainerService" --name AKS-ExtensionManager

# Check the results
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/AKS-ExtensionManager')].{Name:name,State:properties.state}"

# Make sure it's enabled by reregistering
az provider register -n Microsoft.ContainerService

```

### Setup Shell Variables

These variables will be used throughout the demo.

```bash

# This will be used as the name of the RG and cluster
NAME=cdw-kubernetes-20211120

# The AKS gitops add-on is only available in a few locations while in preview
LOCATION=eastus

```

## Azure Resource Setup

### Create an Azure Resource Group

```bash

az group create -n $NAME -l $LOCATION

```

### Create an Azure Kubernetes Cluster

```bash

# Create cluster
az aks create \
--resource-group $NAME \
--name $NAME \
--location $LOCATION \
--kubernetes-version 1.21.2 \
--node-count 1 \
--generate-ssh-keys \
--enable-managed-identity

# --enable-addons monitoring

az aks get-credentials -n $NAME -g $NAME --overwrite

```

### Deploy a GitOps configuration

```bash

# Flux v2
az k8s-configuration flux create \
--resource-group $NAME \
--cluster-name $NAME \
--cluster-type managedClusters \
--name myconfig \
--scope cluster \
--namespace my-namespace \
--kind git \
--url https://github.com/Azure/arc-k8s-demo \
--branch main \
--kustomization name=my-kustomization

# Delete the configuration if necessary
az k8s-configuration flux delete \
--resource-group $NAME \
--cluster-name $NAME \
--cluster-type managedClusters \
--name myconfig

```

### Clean up the cluster or entire resource group

```bash

# Delete cluster
az aks delete \
--resource-group $NAME \
--name $NAME

# Delete the whole RG
az group delete -g $NAME

```
