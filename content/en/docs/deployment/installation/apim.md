---
title: "APIM Management Installation"
linkTitle: "APIM"
weight: 1
description: Installing APIM component of the Axway Open Banking solution
---


## Download Helm chart

Download Axway Open Banking APIM Helm charts to customize them locally

```bash
helm pull axway-open-banking/open-banking-apim --untar
helm pull axway-open-banking/open-banking-apim-config --untar
```

You should get `open-banking-apim` and `open-banking-apim-config` local folders.

## Customize APIM Helm chart

Customize the `open-banking-apim/values.yaml` file as follow

### Base parameters

The following parameters are required for any deployment.

| Value         | Description                           | Default value  |
|:------------- |:------------------------------------- |:-------------- |
| global.platform | select the platform to configure appropriate objects (like storage for RWM).<br>Possible values are AWS, AZURE, MINIKUBE | None |
| global.domainName | set the domainname for all ingress. | None |
| global.env | Set the default environment | dev |
| global.dockerRegistry.username | Login name to pull Docker images from Axway Repository. | None |
| global.dockerRegistry.token | Password token to pull Docker images from Axway Repository. | None |
| global.smtpServer.host | Smtp server host | None |
| global.smtpServer.port | Smtp server port | None |
| global.smtpServer.username | Smtp server username | None |
| global.smtpServer.password | Smtp server password | None |

<!--
TODO:
Add anm user and password change option. once https://jira.axway.com/browse/MED-472 and https://jira.axway.com/browse/MED-118 are solved

| anm.admin.username | API Gateway admin username | admin |
| anm.admin.password | API Gateway admin password | apiAdminPwd! |

Add apimgr  user and password change option. once https://jira.axway.com/browse/MED-835 is solved
| apimgr.admin.username | API Manager admin username | apiadmin |
| apimgr.admin.password | API Manager admin password | apiAdminPwd! |
-->

With these base parameters set, you can already install the helm chart : [Install APIM Helm chart](#install-apim-helm-chart)

This deployment will use cert-manager and let's encrypt issuer to provide certificates. This requires to have an ingress controller (nginx) that listen on a public IP.

You can also customize further the chart values with the following sub-sections.

### Product licence

A temporary license file is embedded in the default docker image.

This license key has a lifetime to 2 months maximum.

This license is perfect for a demo or a POC but another License key must be added for real environments.

| Value         | Description                           | Default value  |
|:------------- |:------------------------------------- |:-------------- |
| global.apimLicense | Insert your license key. An example is in the default value file. | None |

### External Cassandra

According to the reference architecture, the Cassandra database is external to the cluster. Change the following values according to the cassandra configuration.

The helm chart is delivered with an internal cassandra database, that would work for non-procution environments. You can change this parameter to use an external one. It is required at least for production environments.

```yaml
cassandra:
   external: true
   adminName: "cassandra"
   adminPasswd: "cassandra"
   host1: "cassandra"
   host2: "cassandra"
   host3: "cassandra"
```

Please refer to the Axway documentation to configure and manage the Apache Cassandra database for API Gateway and API Manager: [Administer Apache Cassandra](https://docs.axway.com/bundle/axway-open-docs/page/docs/cass_admin/index.html)

### Root CA for MTLS clients

Optionnaly, you can add new root CA for MTLS ingress during the first deployment.

The mutual authentication is provided by Nginx. It requires a Kubernetes secret that contains all rootCA used for client certificates (used by TPPs).

The differents root CA certificats must be concatenate and encoded in base64.

| Value         | Description                           | Default value  |
|:------------- |:------------------------------------- |:-------------- |
| apitraffic.ingressMtlsRootCa | all concatenate root CA encoded in base64 | yes |

### Customize storage class

Only if needed, you can change the storage class.

The APIM deployment needs a storage class in Read/Write Many. A custom storage class can be setted if the cluster doesn't use the standard deployment for Azure, AWS or if the deployment is on a vanilla Kubernetes.

| Value         | Description                           | Default value  |
|:------------- |:------------------------------------- |:-------------- |
| Global.customStorageClass.scrwm | Allow to specify a storageclass to mount a “Read Write Many” volume on pod.<br>It’s used to share metrics between monitoring and analytics. | None |

### Specify a Wildcard certificate

If you don't use cert-manager for your cluster, you can specify a unique certificate for all ingress of this chart.

It's possible to use a custom wildcard certifcate. change values listed below. Note: the cert field must contains the full chain.

```yaml
global:
   ingress:
      certManager: false
      wildcard: true
      cert: |
         -----BEGIN CERTIFICATE-----
         <base64-encoded certificate>
         -----END CERTIFICATE-----
         -----BEGIN CERTIFICATE-----
         <base64-encoded certificate>
         -----END CERTIFICATE-----
         ...

      key: |
         -----BEGIN RSA PRIVATE KEY-----
         <base64-encoded key>
         -----END RSA PRIVATE KEY-----
```

Refer to [Certificate Management](/docs/configuration/certificate-management) for certficate configuration of the entire solution

### Specify different certificates

If you don't use cert-manager for your cluster, you can specify a certificate for each ingress of this chart.

It's possible to define a different certificate for each ingress. Change values listed below. keep an empty line after the key or the certificate.

```yaml
global:
   ingress:
      certManager: false
      wildcard: false
anm:
   ingressCert: ...
   ingressKey: ...
apimgr: 
   ingressCert: ...
   ingressKey: ... 
apitraffic:
   ingressCert: ...
   ingressKey: ...
   ingressCertMtls: ...
   ingressKeyMtls: ...
   ingressCertHttps: ...
   ingressKeyHttps: ...
```

Each cert and key should have be inserted with the following format (same indent and empty lines):

```yaml
   ingressCert: |
      -----BEGIN CERTIFICATE-----
      < insert here base64-encoded certificate >
      -----END CERTIFICATE-----
      -----BEGIN CERTIFICATE-----
      < insert here base64-encoded certificate >
      -----END CERTIFICATE-----

   ingressKey: |
      -----BEGIN RSA PRIVATE KEY-----
      < insert here base64-encoded key >
      -----END RSA PRIVATE KEY----- 

```

> Note : Oauth component is activated but ingress isn't enabled. It's not required to create a certificate for this ingress.

Refer to [Certificate Management](/docs/configuration/certificate-management) for certficate configuration of the entire solution

### Configure Amplify Agents

The following values must be set to reports API and their usage on the **Amplify platform**.

```yaml
amplifyAgents:
   enabled: true
   centralOrgID:  "ORGANIZATION_ID"
   centralAuthClientID: "DOSA_xx_yy_zz"
   centralEnvName:  "ENVIRONMENT_NAME"
   centralTeam: "Default Team"   
   centralUrl: "https://apicentral.axway.com"  
   centralPublicKey:  <Public key for the service account on Central>
   centralPrivateKey: <Private key for the service account on Central>
```

>Note : Private Key and Public Key must be encoded in base64.

Refer to [Amplify Agents](/docs/configuration/amplify-agents) to connect Amplify agent to Amplify platform.

## Install APIM Helm chart

Create the target namespace on the cluster:

```bash
kubectl create namespace open-banking-apim
```

Install the APIM  helm charts:

```bash
helm install apim open-banking-apim -n open-banking-apim
```

Check that the status of the helm command is deployed:

```console
   NAME: apim 
   LAST DEPLOYED: <current date and time>
   NAMESPACE: open-banking-apim 
   STATUS: deployed 
   REVISION: 1 
   TEST SUITE: None
```

### Verifications

Wait a few minutes and use the following commands to check the status of the deployment.

```bash
kubectl get pods -n open-banking-apim 
```

Verify that :

* **pods** with name anm-xxx-xxx, apimgr-xxx-xxx, traffic-xxx-xxx, cassandra-0 are **Running** and Restart is **0**.
* **jobs** with name db-create-mysql-apigw-xxx is **Completed**.

```console
   NAME                                 READY   STATUS         RESTARTS 
   anm-6d86b7dfbd-4wbnx                 1/1     Running        0 
   apimgr-544b55fffb-qsn87              1/1     Running        0 
   cassandra-0                          1/1     Running        0 
   db-create-mysql-apigw-379e224c-...   0/1     Completed      0 
   filebeat-analytics-86d588954b-lsx2p  1/1     Running        0 
   mysql-aga-757495f88f-vpw79           1/1     Running        0 
   traffic-5d986c7d55-cv6dv             1/1     Running        0
```

Check all ingress with this command :

```bash
kubectl get ingress -n open-banking-apim 
```

Verify that these ingress have been provisioned. They must have a public ip or a dns value is in the ADDRESS column.

```console
   NAME            HOSTS                               ADDRESS                        PORTS 
   apimanager      api-manager.<domain-name>           xxxxxxxxxxxxx.amazonaws.com    80, 443 
   gatewaymanager  api-gateway-manager.<domain-name>   xxxxxxxxxxxxx.amazonaws.com    80, 443 
   oauth           oauth.<domain-name>                 xxxxxxxxxxxxx.amazonaws.com    80, 443
   traffic         api.<domain-name>                   xxxxxxxxxxxxx.amazonaws.com    80, 443 
   traffichttps    services-api.<domain-name>          xxxxxxxxxxxxx.amazonaws.com    80, 443 
   trafficmtls     mtls-api.<domain-name>              xxxxxxxxxxxxx.amazonaws.com    80, 443
```

Check that you can access the following User Interfaces:

* **API Gateway Manager** `https://api-gateway-manager.<domain-name>`.

    * Login with username *admin* and password *apiAdminPwd!*
    * Check in the topology section that pods apimgr and traffic are available.

* **API Manager** `https://api-manager.<domain-name>`.

    * Login with username *apiadmin* and password *apiAdminPwd!*
    * Check that API and Client configurations are empty for now

## Customize APIM configuration helm chart

Customize the `open-banking-apim-config/values.yaml` file as follow

| Value         | Description                           | Default value  |
|:------------- |:------------------------------------- |:-------------- |
| global.domainName | set the domainname for all ingress. | None |
| global.env | Set the default environment |dev |
| global.dockerRegistry.username | Login name to pull Docker images from Axway Repository. | None |
| global.dockerRegistry.token | Password token to pull Docker images from Axway Repository. | None |
| apimcli.settings.email | sender email address used in api-manager settings | None |
| apimcli.users.publicApiUser | username of user to access Public APIs from API Portal | _publicuser_ |
| apimcli.users.publicApiPassword | password of user to access Public APIs from API Portal | _publicUserPwd!_ |
| backend.serviceincident.host | ServiceNow URL | None|
| backend.serviceincident.username | ServiceNow username |None|
| backend.serviceincident.password | ServiceNow password |None|

## Install APIM config helm chart

Install the APIM config helm chart:

```bash
helm install apim-config open-banking-apim-config -n open-banking-apim
```

Check that the status of the helm command is deployed:

   ```
   NAME: apim-config 
   LAST DEPLOYED: <current date and time>
   NAMESPACE: open-banking-config 
   STATUS: deployed
   REVISION: 1 
   TEST SUITE: None
   ```

### Verifications

Wait a few minutes and use the following commands to check the status of the deployment.

```
kubectl get pods -n open-banking-apim 
```

Verify that :

* **jobs** with name import-api-27983c3f-xxx  are **Completed** :

   ```
   NAME                                 READY   STATUS      RESTARTS 
   anm-6d86b7dfbd-4wbnx                 1/1     Running     0 
   apimgr-544b55fffb-qsn87              1/1     Running     0 
   cassandra-0                          1/1     Running     0 
   db-create-mysql-apigw-379e224c-...   0/1     Completed   0 
   filebeat-analytics-86d588954b-lsx2p  1/1     Running     0 
   import-api-27983c3f-...              0/1     Completed   0 
   mysql-aga-757495f88f-vpw79           1/1     Running     0 
   traffic-5d986c7d55-cv6dv             1/1     Running     0
   ```

Check the following User Interfaces:

* **API Manager** `https://api-manager.<domain-name>` :

    * Refresh or login again
    * Make sure that Open Banking APIs are in the API Catalog
    * Make sure that Default apps are in Client applications

## Post Deployment

Once APIM helm charts and Cloud Entity Helm chart are deployed, update the KPS configuration as follow to integrate the component together.

### Update KPS configuration

You need to import some configurations in th Key Properties Store. They are used in policies for consent flows.

To change the KPS

* The organization ID is different for each bank, please modify the helm chart file `open-banking-apim-config/files/kps/kpsConfig1.json` to change the organizationId with your own bank/PSPSP ID.

* Execute the following command:

```shell
APIMGR_POD="$(kubectl get pod -n open-banking-apim -l app=apimgr -o jsonpath='{.items[0].metadata.name}')"
ANM_INGRESS_NAME="$(kubectl get ingress -n open-banking-apim gatewaymanager -o jsonpath='{.spec.rules[0].host}')"
# check variables APIMGR_POD and ANM_INGRESS_NAME are not empty
echo $APIMGR_POD : $ANM_INGRESS_NAME
ANM_USERNAME=admin
ANM_PASSWORD='apiAdminPwd!'
curl -k -X PUT -u "$ANM_USERNAME:$ANM_PASSWORD" \
     -H "Content-Type: application/json" \
     -d @open-banking-apim-config/files/kps/kpsConfig1.json \
     "https://${ANM_INGRESS_NAME}:443/api/router/service/${APIMGR_POD}/api/kps/cfg/1" 
awk  -F '\t' '{ \
         if ($5==Y) {$5="true"} else {$5="false"}; \
         print "Create id"$1; \
    system("curl -k -X PUT -u \"'"${ANM_USERNAME}"':'"${ANM_PASSWORD}"'\" -H \"Content-Type: application/json\" 'https://${ANM_INGRESS_NAME}:443'/api/router/service/'${APIMGR_POD}'/api/kps/'mediciobie_endpoint'/"$1" -d '\''{\"id\":\""$1"\",\"category\":\""$2"\",\"name\":\""$3"\",\"segment\":\""$4"\",\"used\":"$5"}'\''")}' \
     "open-banking-apim-config/files/kps/obie_endpoint.txt"
```

Verify the insertion in the KPS table:

* Login the API Gateway Manager UI and go on Settings > Key Property Stores
* Click on AMPLIFY/Configuration
* Check the column **k_values** that isn't empty. Click on it to check details