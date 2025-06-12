# Setup kind cluster
```
kind create cluster --name crossplane
```

# Install Crossplane
```
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
helm search repo crossplane-stable --versions

export VERSION=1.20.0
helm install crossplane \
--namespace crossplane-system \
--create-namespace crossplane-stable/crossplane \
--version $VERSION
```

# Verify installation
```
kubectl get deployments -n crossplane-system
```

# Available api resources in Crossplane
```
kubectl api-resources | grep crossplane.io

compositeresourcedefinitions        xrd,xrds     apiextensions.crossplane.io/v1        false        CompositeResourceDefinition
compositionrevisions                comprev      apiextensions.crossplane.io/v1        false        CompositionRevision
compositions                        comp         apiextensions.crossplane.io/v1        false        Composition
environmentconfigs                  envcfg       apiextensions.crossplane.io/v1beta1   false        EnvironmentConfig
usages                                           apiextensions.crossplane.io/v1beta1   false        Usage
configurationrevisions                           pkg.crossplane.io/v1                  false        ConfigurationRevision
configurations                                   pkg.crossplane.io/v1                  false        Configuration
controllerconfigs                                pkg.crossplane.io/v1alpha1            false        ControllerConfig
deploymentruntimeconfigs                         pkg.crossplane.io/v1beta1             false        DeploymentRuntimeConfig
functionrevisions                                pkg.crossplane.io/v1                  false        FunctionRevision
functions                                        pkg.crossplane.io/v1                  false        Function
imageconfigs                                     pkg.crossplane.io/v1beta1             false        ImageConfig
locks                                            pkg.crossplane.io/v1beta1             false        Lock
providerrevisions                                pkg.crossplane.io/v1                  false        ProviderRevision
providers                                        pkg.crossplane.io/v1                  false        Provider
storeconfigs                                     secrets.crossplane.io/v1alpha1        false        StoreConfig

```

# The crossplane providers are the apis that are used to manage the resources in the cloud provider.
### For example, the azure provider is used to manage azure resources.
```
kubectl api-resources | grep crossplane.io

providers                                        pkg.crossplane.io/v1                  false        Provider
```
### The providers creates kubneretes objects that are used to manage the resources in the cloud provider. Providers are responsible for all aspects of connecting to non kubernetes resources, that includes authentication, making external api calls and providing kubernetes controller logic for any external resources.

# Install Crossplane Azure Provider
### Here we are installing the azure provider which is used to manage azure resources.
#### NOTE: Check the demo project for more examples of how to use the azure provider [here](../demo-project/)

```
kubectl apply -f demo-project/crossplane/azure-provider.yaml
```
# Verify Azure Provider installation
```
kubectl get providers

NAME                            INSTALLED   HEALTHY   PACKAGE                                                     AGE
provider-azure-management       True        Unknown   xpkg.upbound.io/upbound/provider-azure-management:v1.13.0   7s
upbound-provider-family-azure   True        False     xpkg.upbound.io/upbound/provider-family-azure:v1.13.0       3s

```

### It might take few minutes for the provider to be healthy, describe the provider if in case to see the status.

```
kubectl describe provider provider-azure-management

kubectl get providers
NAME                            INSTALLED   HEALTHY   PACKAGE                                                     AGE
provider-azure-management       True        True      xpkg.upbound.io/upbound/provider-azure-management:v1.13.0   3m12s
upbound-provider-family-azure   True        True      xpkg.upbound.io/upbound/provider-family-azure:v1.13.0       3m8s
```

## Our controll plane is now ready to use the providers, But we need to configure the providers with the credentials to access the cloud provider resources. Hence we need to create a provider credential configuration.

### It is [recommended](https://marketplace.upbound.io/providers/upbound/provider-family-azure/v1.13.0/resources/azure.upbound.io/ProviderConfig/v1beta1) to use the managed identity for the azure provider, but for the sake of simplicity we will use the service principal credentials.

```
az ad sp create-for-rbac \
--sdk-auth \
--role Owner \
--scopes /subscriptions/<subscription-id> 

# Replace <subscription-id> with your azure subscription id
```

### The above command will output a json object with the credentials, we need to create a kubernetes secret with this credentials.

```
kubectl create secret generic azure-credentials \
--namespace crossplane-system \
--from-file=creds='<json-credentials>'
```

## A provider configuration is created with the credentials [here](../demo-project/providerconfig-azure.yaml), let us apply it to the cluster.

```
kubectl apply -f demo-project/providerconfig-azure.yaml
```

# Verify Provider Configuration
```
kubectl get providerconfigs
NAME                            INSTALLED   HEALTHY   PACKAGE                                                     AGE
providerconfig-azure            True        True      xpkg.upbound.io/upbound/provider-family-azure:v1.13.0       3m12s
```

### The provider configuration is now ready to use the azure resources, we can create a resource claim to create a resource in the cloud provider.
### For example, we can create a resource claim for an azure resource group [here](../demo-project/resourcegroup.yaml).

```
kubectl apply -f demo-project/resourcegroup.yaml
```

# Verify Resource Group Creation
```
kubectl get resourcegroups.azure.upbound.io
NAME                            READY   SYNCED   EXTERNAL-NAME   AGE
crossplane-rg                   True    True     crossplane-rg   3m12s
```