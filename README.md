#  helm-k8s-chart_umbrella-dotnet_app

This repository contains a helm chart that is used to deploy a `dotnet core` application to kubernetes cluster. This chart can be extended to deploy any microservice written in any language as long as it accepts inputs and secrets (pulled from Azure Key vault) in form of environment variables.

It also supports pulling the secrets from Azure Key Vault (by using k8s csi plugin) and injecting them into the pod through volume mounts and environment variables.

## Authenticate to ACR

This form of authentication is using temporary token to authenticate to the ACR. This is useful when you are using helm registry plugin to push the chart to ACR.

```bash
# Authenticate
USER_NAME="00000000-0000-0000-0000-000000000000"
PASSWORD=$(az acr login --name dsopublic --expose-token --output tsv --query accessToken)

helm registry login dsopublic.azurecr.io \
  --username $USER_NAME \
  --password $PASSWORD
```

## Helm push to ACR

```bash
# Ensure the plugin is installed
helm plugin install https://github.com/chartmuseum/helm-push.git
# CD to the parent directory containing the chart folder
cd ..
# Package the chart
helm package helm-k8s-chart_umbrella-dotnet_app
# Push the chart to ACR
helm push dotnet-app-0.2.0.tgz oci://dsopublic.azurecr.io/helm

```

## Helm install from ACR

Installing the chart from ACR is not the same as installing from a helm registry. The following commands `DO NOT` work.

```bash
helm repo add myrepo oci://dsopublic.azurecr.io/helm
helm repo update
helm install myapp myrepo/dotnet-app
```

Instead, you need to follow the below steps to install the chart from ACR.

```bash
# Ensure you are logged in to the ACR
helm install sample oci://dsopublic.azurecr.io/helm/dotnet-app --version 0.2.0 --dry-run

helm upgrade  sample  oci://dsopublic.azurecr.io/helm/dotnet-app --version 0.2.0 -f override_values.yaml

```