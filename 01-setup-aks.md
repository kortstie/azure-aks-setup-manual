### Login to azure
    az login

### Set env
    read -p "Enter your projectname: " project
    export resource_group=${project}rg
    export cluster_name=${project}aks
    export acr_name=${project}acr
    export location=germanywestcentral

### Create Resource Group
    az group create --name=$resource_group --location=$location

### Create aks cluster

    az aks create \
    --resource-group $resource_group \
    --name $cluster_name \
    --enable-cluster-autoscaler \
    --node-count 3 \
    --min-count 3 \
    --max-count 6 \
    --zones 1 2 3
    --generate-ssh-keys \
    --node-vm-size Standard_B2s \
    --network-plugin azure \
    --enable-app-routing \
    --tier free

### Get aks Credentials
    az aks get-credentials --name $cluster_name --resource-group $resource_group

### Create Container Registry
    az acr create --resource-group $resource_group --name $acr_name --sku Basic

### Connect Container Registry to AKS Cluster
    az aks update -n $cluster_name -g $resource_group --attach-acr $acr_name

### Setup Service Principal

   groupId=$(az group show --name $resource_group --query id --output tsv)
   az ad sp create-for-rbac --scope $groupId --role Contributor --json-auth

Save this output.

### allow push to Container registry

   registryId=$(az acr show --name $acr_name --resource-group $resource_group --query id --output tsv)
   az role assignment create --assignee <ClientId> --scope $registryId --role AcrPush

### Import Container Image
    az acr import  -n $acr_name --source docker.io/library/nginx:latest --image nginx:v1
    

### shutdown AKS Cluster
    az aks stop --resource-group $resource_group --name $cluster_name

### show AKS Cluster Powerstate
    az aks show --resource-group aks-rg1 --name aksdemo1 |jq -r '.powerState'
