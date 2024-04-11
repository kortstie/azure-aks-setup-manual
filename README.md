# Manual setup of an azure aks multi-zone cluster

![title](images/title.jpg)

## Purpose of this repo

- show how to set up a multi-zone azure aks cluster using the azure CLI
- create an acr image registry and connect it to the aks cluster
- setup a service principal to allow github actions to deploy images into the container registry

## Prerequisites

- azure account - you can test it for free, see https://azure.microsoft.com
- working environment with all nessecary tools, [here is my setup guide](https://github.com/kortstie/k8s-working-environment)

## Setup aks (azure k8s service)

### Login to azure
    az login

### Set some variables to name our objects

Change **project** to whatever you want - but not "demo", because demoacr is already in use

    export project=kortstie
    export resource_group=${project}rg
    export cluster_name=${project}aks
    export acr_name=${project}acr
    export location=germanywestcentral

### Create resource group
    
    az group create --name=$resource_group --location=$location

### Create aks cluster

    az aks create \
    --resource-group $resource_group \
    --name $cluster_name \
    --node-count 3 \
    --zones 1 2 3
    --generate-ssh-keys \
    --node-vm-size Standard_B2s \
    --network-plugin azure \
    --enable-app-routing \
    --tier free

### Get aks credentials
    
    az aks get-credentials --name $cluster_name --resource-group $resource_group

Now we can interact with our brand-new k8s cluster - let's verify that the nodes are distributed across three availability zones:

    kubectl get no -L failure-domain.beta.kubernetes.io/zone

## Setup an azure acr container registry

### Create container registry
    
    az acr create --resource-group $resource_group --name $acr_name --sku Basic

### Connect container registry to aks cluster
    
    az aks update -n $cluster_name -g $resource_group --attach-acr $acr_name

## Setup a service principal to automate things

... like pushing images from github actions

    groupId=$(az group show --name $resource_group --query id --output tsv)
    az ad sp create-for-rbac --scope $groupId --role Contributor --json-auth

**Save** this output, we will store it later in github secrets.

    registryId=$(az acr show --name $acr_name --resource-group $resource_group --query id --output tsv)
    az role assignment create --assignee <ClientId> --scope $registryId --role AcrPush
    
## Some usefull az Commands to work with our environment

### Scale aks cluster
    az aks scale --name kortstieaks --resource-group kortstierg --node-count 2

### shutdown AKS Cluster
    
    az aks stop --resource-group $resource_group --name $cluster_name

### show AKS Cluster Powerstate
    
    az aks show --resource-group aks-rg1 --name aksdemo1 |jq -r '.powerState'

### Import Container Image

    az acr import  -n $acr_name --source docker.io/library/nginx:latest --image nginx:v1

### Show images

    az acr repository list -n kortstieacr --tag

### Delete Resource Group (Delete EVERYTHING!)

   az group delete --resource-group $resource_group
