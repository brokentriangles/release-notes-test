# Update for 2020-07-24
 
## What's changed?
 
### vault migration

As you know vault is going away, here's the latest:

* azure key-vaults have now been created for all k8s deployed application and services for dev, stg and prd.
* most apps have now transitioned to using key-vault. And whoever hasn't should be doing that now. 

> **NOTE** Some squad level PRs are still outstanding and it's *essential* that these get merged in order to complete this migration phase.

## What's new?

### cluster rebuilds

This sprint's round of rebuilds upgrades k8s a point-release version to 1.15.11.

> **NOTE** In case you didn't know: cluster rebuilds are how we maintain a level of consistency acorss environments. They allow us to transation the infrastructure from one state to another and address issues along the way.
> 
> We have two production clusters only one of which is live and reciving requests.


| aks-cluster-1-dev | aks-cluster-1-stg | aks-cluster-1-prd | aks-cluster-2-prd |
| ----------------- | ----------------- | ----------------- | ----------------- |
|    1.15.11    |    1.15.11    |     1.15.10   |     1.15.11   | 
> **TODO** The `aks-cluster-1-prd` rebuild to be complete 2020-08-03 will bring an end to this cycle of cluster rebuilds.

In addition to a version upgrade, we are now also using [managed identities](https://docs.microsoft.com/en-us/azure/aks/use-managed-identity). This simplifies the aks service's integration with AzureRM apis with only a simple change to the terraform `cluster.tf` file: 

```hcl
  identity {
    type = "SystemAssigned"
  }
```

If you're interested you can [check out the PRs!](https://github.com/arnoldclark/ac-iac-platform/pull/329).

### cluster additions

We also added stuff!

### 
  * cert-manager has been rolled out cluster wide.

This allows us to automate the provisioning of certificates for your app ingress simply but adding an annotation. Pretty cool. 

```yaml 
metadata:
  annotations:
    cert-manager.io/cluster-issuer: ca-issuer
```

  * log retention improvements
  
> **NOTE** we're currently logging >5GB a day, meaning our log retention is low. we've added some additional log filtering so we don’t reach capacity quite so quickly, and therefore increase log retention!

  * 200 - OK healthchecks now removed
 
## What's coming (and stuff for you to do!)
 
> **NOTE** we will be upgrading to k8s 1.16.x soon and there's some big changes on the way with that release.
 
aks maintains a rolling policy on k8s versioning with an [N-2 approach](https://docs.microsoft.com/en-us/azure/aks/supported-kubernetes-versions). We are currently N-1 and so to ensure we stay up to date with the latest security patches etc, we *need to ensure* we keep up-to-date. 
 
* [k8s 1.16 removes specific apis](https://github.com/Azure/AKS/issues/1205) that have for the last few releases been deprecated.
* resources of type 'deployment' should update their use of the `extensions/v1beta` api to `apps/v1`
* re-test & re-deploy

> **! NOTE** this is where you come in. There will be comms sent out and it'll be mentioned again in future release notes but **be aware** that your deployments api verions will be changing. 
 
## What's left?
 
If you haven't already please deploy your applications and services to the newly rebuild `aks-cluster-2-prd` so that we can direct the traffic toward into UK South, and the `aks-cluster-1-stg` so that we may decommision that cluster. 

