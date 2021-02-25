ACR_NAME=teleport
AKS_SP_NAME=teleport-aks
AKS_SP_PWD=$(az ad sp create-for-rbac --skip-assignment --name ${AKS_SP_NAME} --query password -o tsv)
AKS_SP_ID=$(az ad sp show --id http://${AKS_SP_NAME} --query appId -o tsv)
LOCATION=westus2
az aks create \
 -g $AKS_RG \
 -n MyAKS \
 --attach-acr $ACR_NAME \ 
 --service-principal $AKS_SP_ID \
 --client-secret $AKS_SP_PWD \
 --kubernetes-version 1.19.3 \
 -l eastus


az aks nodepool add \
 -g $AKS_RG \
 --cluster-name $AKS \
 --name teleportnp \
 --node-count 1 \
 --node-taints teleport=yes:NoSchedule

### Scratch/Notes

Both nodepools should have the same `vmSize`

```azurecli
    "vmSize": "Standard_DS2_v2",
```

Taints

```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name taintnp \
    --node-count 1 \
    --node-taints sku=gpu:NoSchedule \
    --no-wait
```
## Applying Labels

```azurecli-interactive
kubectl get nodes

kubectl label nodes \
  aks-teleporter-10583637-vmss000000 \
  teleport=enabled

kubectl label nodes \
  aks-shuttle-10583637-vmss000000 \
  teleport=disabled


kubectl describe node aks-nonteleport

```

## Troubleshooting commands

List of nodepools, with labels:

```azurecli
az aks nodepool list \
    --resource-group $AKS_RG \
    --cluster-name $AKS \
    -o jsonc
```

Note a few properties to assure like configurations

The following should only be available on the teleport enabled nodepools. The second non-teleport nodepool should not have this label.
```azurecli
 "nodeLabels": {
      "kubernetes.azure.com/enable-acr-teleport-plugin": "true"
    },
```

