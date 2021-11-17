# Learn AKS GitOps

A repository for creating a GitOps-enabled AKS cluster in Azure.

## Prerequisites

### Azure CLI Prerequisites

As part of this demo, make sure all of the preview extensions are up to date.

```bash

# Add (or register) the Azure CLI extensions
az extension add --name aks-preview
az extension add --name k8sconfiguration

az extension update --name aks-preview
az extension update --name k8sconfiguration

# Install the GitOps feature
az feature register --namespace "Microsoft.ContainerService" --name aks-gitops

# Check the results
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/aks-gitops')].{Name:name,State:properties.state}"

# Make sure it's enabled by reregistering
az provider register -n Microsoft.ContainerService

```

### Setup Shell Variables

These variables will be used throughout the demo.

```bash

NAME=cdw-kubernetes-20211117

# Azure Arc is limited to East US (among a few others), so use it
LOCATION=eastus2

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
--generate-ssh-keys \
--enable-managed-identity \
--enable-addons gitops

# --enable-addons monitoring

az aks get-credentials -n $NAME -g $NAME --overwrite

# Delete cluster
az aks delete \
--resource-group $NAME \
--name $NAME

```
