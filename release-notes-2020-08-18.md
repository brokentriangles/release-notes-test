# Update for 2020-08-18
 
## What's changed?
 
### the vault migration

As you know vault is going away, here's the latest:

* azure key-vaults have now been created for all k8s deployed application and services for dev, stg and prd.
* most apps have now transitioned to using key-vault. And whoever hasn't should be doing that now. 
 
## What's new?

### cluster rebuilds

This sprint's round of rebuilds upgrades k8s a point-release version to 1.15.11.

> **NOTE** In case you didn't know: cluster rebuilds are how we maintain a level of consistency acorss environments. They allow us to transation the infrastructure from one state to another and address issues along the way.
> 
> We have two production clusters only one of which is live and reciving requests.


| aks-cluster-1-dev | aks-cluster-1-stg | aks-cluster-1-prd | aks-cluster-2-prd |
| ----------------- | ----------------- | ----------------- | ----------------- |
|    1.15.11        |    1.15.11        |     1.15.10       |     1.15.11       | 


In addition to a version upgrade, we are now also using [managed identities](https://docs.microsoft.com/en-us/azure/aks/use-managed-identity). This simplifies the aks service's integration with AzureRM apis with only a simple change to the terraform `cluster.tf` file: 

```hcl
  identity {
    type = "SystemAssigned"
  }
```

If you're interested you can [check out the PRs!](https://github.com/arnoldclark/ac-iac-platform/pull/329).

### cluster additions and tls

We also added stuff!

### 
  * [cert-manager](https://cert-manager.io/docs/) has been rolled out cluster wide acorss all environments.
  * a reworking of how tls is deployed as a cluster resource
  
Our approach to deploying tls has been simplified and we no longer require any tls config in the ingress resources that you deploy. This has lead us to reevaluate the cluster requrements for tls using default tls certificates [check it out](https://github.com/arnoldclark/ac-iac-platform/pull/349).

> **NOTE**  In case you didn't know the original motivation for cert-manager was automation and to fill the gap left by some hashivault features once we had migrated away from it. There's many ways to achieve the same thing, something we're all familiar with. You can think of this as an interation on the design of the tls component. 

Soon these infrastructure level changes will be transparently managed, **but for right now** we need you to do the doing. Here's a sample code snipit showing the unecessary config that we'd like you to remove:

> **NOTE**  These changes are safe to deploy as soon as possible.

```diff
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
-    cert-manager.io/cluster-issuer: ca-issuer
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"extensions/v1beta1","kind":"Ingress","metadata":{"annotations":{"cert-manager.io/cluster-issuer":"ca-issuer","kubernetes.io/ingress.class":"nginx","nginx.ingress.kubernetes.io/ssl-redirect":"true"},"name":"proof-of-identity-frontend","namespace":"proof-of-identity"},"spec":{"rules":[{"host":"dev.app.arnoldclark.com","http":{"paths":[{"backend":{"serviceName":"proof-of-identity-frontend","servicePort":80},"path":"/proof-of-identity-frontend"}]}},{"host":"aks-cluster-1-dev.arnoldclark.com","http":{"paths":[{"backend":{"serviceName":"proof-of-identity-frontend","servicePort":80},"path":"/proof-of-identity-frontend"}]}}],"tls":[{"hosts":["dev.app.arnoldclark.com","aks-cluster-1-dev.arnoldclark.com"],"secretName":"cert-manager"}]}}
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
  creationTimestamp: "2020-08-04T10:31:25Z"
  generation: 3
  name: proof-of-identity-frontend
  namespace: proof-of-identity
  resourceVersion: "1557120"
  selfLink: /apis/extensions/v1beta1/namespaces/proof-of-identity/ingresses/proof-of-identity-frontend
  uid: 7f6591ae-7799-40ec-ba0e-8db97492faf5
spec:
  rules:
  - host: dev.app.arnoldclark.com
    http:
      paths:
      - backend:
          serviceName: proof-of-identity-frontend
          servicePort: 80
        path: /proof-of-identity-frontend
  - host: aks-cluster-1-dev.arnoldclark.com
    http:
      paths:
      - backend:
          serviceName: proof-of-identity-frontend
          servicePort: 80
        path: /proof-of-identity-frontend
-  tls:
-  - hosts:
-    - dev.app.arnoldclark.com
-    - aks-cluster-1-dev.arnoldclark.com
-    secretName: cert-manager (this may say ingress-https)
status:
  loadBalancer:
    ingress:
    - ip: 10.122.7.253
```

  * log retention improvements
  
We're currently log >5GB a day, meaning our log retention is low. [we've added some additional log filtering](https://github.com/arnoldclark/ac-iac-platform/pull/335) so we don’t reach capacity quite so quickly, and therefore increase log retention! Thanks Mel!

  * 200 - OK healthchecks now removed
 
## What's coming (and stuff for you to do!)

> **NOTE** we will be upgrading to k8s 1.16.x soon and there's some big changes on the way with that release.
 
aks maintains a rolling policy on k8s versioning with an [N-2 approach](https://docs.microsoft.com/en-us/azure/aks/supported-kubernetes-versions). We are currently N-1 and so to ensure we stay up to date with the latest security patches etc, we *need to ensure* we keep up-to-date. 
 
* [k8s 1.16 removes specific apis](https://github.com/Azure/AKS/issues/1205) that have for the last few releases been deprecated.
* resources of type 'deployment' should update their use of the `extensions/v1beta` api to `apps/v1`
* re-test & re-deploy

> **! NOTE** this is where you come in. There will be comms sent out and it'll be mentioned again in future release notes but **be aware** that your deployment's api version's will be changing. 
 
## What's left?
 
If you haven't already please deploy your applications and services to UK West's newly rebuild `aks-cluster-2-prd` so that we can migrate traffic to it. And similarly if you haven't already please deploy out to UK South's `aks-cluster-1-stg` so that we may decommision our out of date resources.

