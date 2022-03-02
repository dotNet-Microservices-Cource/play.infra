# Play.Infra
Play Economy Infrastructure components

## Add the GitHub package source
```powershell
$owner="dotNet-Microservices-Cource"
$gh_pat="[PAT HERE]"

dotnet nuget add source --username USERNAME --password $gh_pat --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$owner/index.json"
```

## Creating the Azure resource group
```powershell
$appname="playeconomy"
az group create --name $appname --location eastus
```

## Creating the Cosmos DB account
```powershell
az cosmosdb create --name $appname --resource-group $appname --kind MongoDB --enable-free-tier
```

## Creating the Service Bus namespace
```powershell
az servicebus namespace create --name $appname --resource-group $appname --sku Standard
```

## Creating the Container Registry
```powershell
$registryname="playeconomyapp"
az acr create --name $registryname --resource-group $appname --sku Basic
```

## Creating the AKS clusters
```powershell
az feature register --name EnablePodIdentityPreview --namespace Microsoft.ContainerService
az extension add --name aks-preview

az aks create -n $registryname -g $appname --node-vm-size Standard_B2s --node-count 2 --attach-acr $registryname --enable-pod-identity --network-plugin azure

az aks get-credentials --name $registryname --resource-group $appname
```

## Creating the Azure Key Vault
```powershell
az keyvault create -n $registryname -g $appname
```