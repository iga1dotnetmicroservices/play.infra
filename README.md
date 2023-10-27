# Play.Infra

Play Economy Infrastructure components

# Add the Github package source

MacOS

```shell
owner='iga1dotnetmicroservices'
gh_pat='[PAT HERE]'

dotnet nuget add source --username USERNAME --password $gh_pat --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$owner/index.json"
```

Windows Powershell

```powershell
$owner='iga1dotnetmicroservices'
$gh_pat='[PAT HERE]'

dotnet nuget add source --username USERNAME --password $gh_pat --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$owner/index.json"
```

## Creating the Azure resource group 

MacOS

```shell
appname='iga1playeconomy'
az group create --name $appname --location eastus
```

Windows

```powershell
$appname='iga1playeconomy'
az group create --name $appname --location eastus
```

## Creating the Cosmos DB account

MacOS

```shell
az cosmosdb create --name $appname --resource-group $appname --kind MongoDB --enable-free-tier
```

Windows

```powershell
az cosmosdb create --name $appname --resource-group $appname --kind MongoDB --enable-free-tier
```

## Creating the Service Bus namespace

MacOS

```shell
az servicebus namespace create --name $appname --resource-group $appname --sku Standard
```

Windows

```powershell
az servicebus namespace create --name $appname --resource-group $appname --sku Standard
```

## Creating the Container Registry

MacOS

```shell
az acr create --name $appname --resource-group $appname --sku Basic
```

Windows

```powershell
az acr create --name $appname --resource-group $appname --sku Basic
```

## Creating the AKS cluster

MacOS

```shell
az feature register --name EnablePodIdentityPreview --namespace Microsoft.ContainerService
az extension add --name aks-preview

az aks create --name $appname --resource-group $appname --node-vm-size Standard_B2s --node-count 2 --attach-acr $appname --enable-pod-identity --network-plugin azure

az aks get-credentials --name $appname --resource-group $appname
```

Windows

```powershell
az feature register --name EnablePodIdentityPreview --namespace Microsoft.ContainerService
az extension add --name aks-preview

az aks create --name $appname --resource-group $appname --node-vm-size Standard_B2s --node-count 2 --attach-acr $appname --enable-pod-identity --network-plugin azure

az aks get-credentials --name $appname --resource-group $appname
```

## Creating the Azure Key Vault

```powershell
az keyvault create -n $appname -g $appname
```

## Installing Emissary-ingress

```powershell
helm repo add datawire https://app.getambassador.io
helm repo update

kubectl apply -f https://app.getambassador.io/yaml/emissary/3.8.2/emissary-crds.yaml
kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system

$namespace='emissary'

helm install emissary-ingress datawire/emissary-ingress --set service.annotations."service\.beta\.kubernetes\.io/azure-dns-label-name"=$appname -n $namespace --create-namespace

kubectl rollout status deployment/emissary-ingress -n $namespace -w 
```

## Configuring Emissary-ingress routing

```powershell
kubectl apply -f .\emissary-ingress\listener.yaml -n $namespace
kubectl apply -f .\emissary-ingress\mappings.yaml -n $namespace
```