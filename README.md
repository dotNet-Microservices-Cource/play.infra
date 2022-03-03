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

## Installing Emissary-ingress
```powershell
helm repo add datawire https://app.getambassador.io
helm repo update

kubectl apply -f https://app.getambassador.io/yaml/emissary/2.2.2/emissary-crds.yaml
kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system

$namespace="emissary"
helm install emissary-ingress datawire/emissary-ingress --set service.annotations."service\.beta\.kubernetes\.io/azure-dns-label-name"=$registryname -n $namespace --create-namespace

kubectl rollout status deployment/emissary-ingress -n $namespace -w 
```

## Configuring Emissary-ingress routing
```powershell
kubectl apply -f .\emissary-ingress\listener.yaml -n $namespace
kubectl apply -f .\emissary-ingress\mappings.yaml -n $namespace
```

## Installing cert-manager
```powershell
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager --version v1.7.1 --set installCRDs=true --namespace $namespace
```

## Creating the cluster issuer
```powershell
kubectl apply -f .\cert-manager\cluster-issuer.yaml -n $namespace
kubectl apply -f .\cert-manager\acme-challenge.yaml -n $namespace
```

## Creating the tls certificate
```powershell
kubectl apply -f .\emissary-ingress\tls-certigicate.yaml -n $namespace
```

## Enabling TLS and HTTPS
```powershell
kubectl apply -f .\emissary-ingress\host.yaml -n $namespace
```