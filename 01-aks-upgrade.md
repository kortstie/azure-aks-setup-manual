# Azure aks upgrades

### get available k8s versions

    az aks get-versions --location germanywestcentral --output table

### create aks in an older version - deactivae auto-upgrade

    az aks create \
    --resource-group $resource_group \
    --name $cluster_name \
    --kubernetes-version 1.28.3 \
    --auto-upgrade-channel none \
    --node-count 3 \
    --zones 1 2 3
    --generate-ssh-keys \
    --node-vm-size Standard_B2s \
    --network-plugin azure \
    --enable-app-routing \
    --tier free

### show upgrade options

    az aks get-upgrades --resource-group $resource_group --name $cluster_name  --output table

### start aks upgrade

    az aks list --resource-group $resource_group --output table

### monitor the upgrade

    az aks get-credentials --name $cluster_name --resource-group $resource_group

    watch kubectl get no -o wide

    kubectl get events

    
