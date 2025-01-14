---
title: "Developer Portal Installation"
linkTitle: "Developer Portal"
weight: 2
description: Installing Developer Portal of the Axway Open Banking solution
---


## Download Helm chart

Download Axway Open Banking Developer Portal Helm chart to customize it locally

```bash
helm pull axway-open-banking/open-banking-developer-portal --untar
```

You should get a `open-banking-developer-portal` local folder.

## Customize Developer Portal Helm chart

Customize the `open-banking-developer-portal/values.yaml` file as follow

| Value         | Description                           | Default value  |
|:------------- |:------------------------------------- |:-------------- |
| global.platform | select the platform : AWS, AZURE, MINIKUBE | AWS |
| global.domainName | set the domainname for all ingress. | None |
| global.dockerRegistry.username | Login name to pull Docker images from Axway Repository. | None |
| global.dockerRegistry.token | Password token to pull Docker images from Axway Repository. | None |
| apiportal.adminPasswd | password to access Developer Portal Joomla admin console | _portalAdminPwd!_ |
| apiportal.company | name of you company, sued for brandind | _Griffin Bank_ |
| apiportal.chatraid |  your Chatra account |  |
| apiportal.recaptchkey | recaptcha key associated to your external domain name |  |
| apiportal.recaptchsecret |  corresponding recaptcha key associated to your external domain name |  |
| apiportal.demoAppSource |   the demo app source URL to be used on the portal home page | `https://demo-apps.<domain-name>/app.js?version=1.1` |
| apiportal.authorizationHost |   the OAuth server public name |  acp.\<domain-name> |
| apiportal.apiWhitelist |  coma-separated list of hosts exposing APIs | api.\<domain-name>,mtls-api-proxy.\<domain-name> |
| apiportal.oauthWhitelist |  coma-separated list of hosts used for external Oauth | acp.\<domain-name> |
| apiportal.serviceDeskEndPoint | URL of service desk service  | `https://api.<domain-name>/services/v1/incident`   |
| apiportal.apiReviewEndPoint |   URL of API review service  | `https://api.<domain-name>/api/portal/v1.2/reviewapi` |
| mysqlPortal.rootPasswd | root password for the database to be created | _portalDBRootPwd!_ |
| mysqlPortal.adminPasswd  | admin password for the database to be created | _portalDBAdminPwd!_ |
| apimgr.publicApiUser | username of API Manager user to access Public APIs | _publicuser_ |
| apimgr.publicApiPassword | password of API Manager user to access Public APIs | _publicUserPwd!_ |

## Install Developer Portal Helm chart

Create the target namespace on the cluster:

```bash
kubectl create namespace open-banking-developer-portal
```

Install the APIM  helm charts:

```bash
helm install developer-portal open-banking-developer-portal -n open-banking-developer-portal
```

Check that the status of the helm command is deployed:

```
    NAME: developer-portal 
    LAST DEPLOYED: <current date and time>
    NAMESPACE: open-banking-developer-portal 
    STATUS: deployed 
    REVISION: 1 
    TEST SUITE: None
```

### Verifications

Wait a few minutes and use the following commands to check the status of the deployment.

```
kubectl get pods -n open-banking-developer-portal 
```

Verify that :

* **pods** with name api-portal-xxx-xxx, mysql-portal-xxx-xxx, redis-xxx-xxx are **Running** and Restart is **0**.

```
    NAME                            READY   STATUS    RESTARTS   AGE  
    api-portal-7d8fb64c98-bt6jg     1/1     Running   0          2m
    mysql-portal-5dc9487c64-jccpm   1/1     Running   0          2m
    redis-7c9bf54b6-dn55s           1/1     Running   0          2m
```

Check ingress with this command :

```bash
kubectl get ingress -n open-banking-developer-portal 
```

Verify that one ingress has been provisioned. It must have a public ip or a dns value is in the ADDRESS column.

```
    NAME         HOSTS                           ADDRESS                       PORTS     AGE
    api-portal   developer-portal.<domain-name>  xxxxxxxxxxxxx.amazonaws.com   80, 443   2m
```

Check the differents User Interfaces:

* Developer Portal home page  : `https://developer-portal.<domain-name>`. If APIM helm charts were successfully deployed, you should already be able to see APIs on the API Catalog (click on API tab)
* Joomla admin interface: `https://developer-portal.<domain-name>/administrator`

    * Login with username *apiadmin* and password *apiAdminPwd!*.
