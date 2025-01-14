---
title: "OpenID Security conformance"
linkTitle: "Security conformance"
description: How to used OpenID test suite to certify your solution deployment
weight: 2
date: 2021-09-02
---

As part of the certification process, the customer´s platforms should be compliance with OpenID Foundation security tests:

* <https://openid.net/certification/instructions/>

We have two different set of tests:

* DCR tests : dynamic client registration
* FAPI Advanced test : Financial-grade API

This section describes the setup tests that is necessary to configure the environment to run the security tests. 

## Conformance test suite

You can either run these tests:

* on Open ID platform <https://www.certification.openid.net>. You would need an account to login
* or, on your local test platform that you can deploy by following these instructions : <https://gitlab.com/openid/conformance-suite/-/wikis/Developers/Build-&-Run>

You should then be able to create and run new test plans

![Conformance Suite : Create Plan](/Images/conformance-suite/create-plan.png)

## Dynamic Client Registration Test

This test validates the TPP creation on Authorization Server via Central Directory request.

### Getting Central Directory information

It is important to run the test to get the following information from [Central Directory Sandbox](https://web.sandbox.directory.openbankingbrasil.org.br/) :

* Client ID : Central Directory client ID to register 
![client-id](/Images/central_directory_brazil_clientid.png)
* BRSEAL – message certificate (cert and key) – used for JWKS .
![BRSEAL](/Images/central_directory_brazil_brseal.png)
* BRCAC – transport certificate – used for MTLS comunication
![BRCAC](/Images/central_directory_brazil_brcac.png)

>You can find more details on [Central Directory Operation Guide](https://openbanking-brasil.github.io/areadesenvolvedor/documents/OpenBanking-Guia_Operacao_Diretorio_Central.pdf) (Portuguese)

### Get the application declared in APIM side

<!-- TODO : remove this chatper once limitation is overcome -->

>This release has a limitation that the ClientID is not automatically created on APIM.

On APIM there is an organization to support this test: _Testing_. And we need to create a new Application for the ClientID that will be used for this test.

* Create a new Application on Testing organization with access to the payment API. 
* Get the Application ID.  Use the API call to get all applications and find the _id_ corresponding to your new application.

    ```bash
    curl --user apiadmin https://api-manager.<domain-name>/api/portal/v1.3/applications  
    ```

* Create the OAuth Credentials for the application is:

```bash
curl --location --request POST 'https://api-manager.<domain-name>/api/portal/v1.3/applications/5e321c00-5e5e-4167-a295-dc05e40c4e50/oauth' \
--user apiadmin \
--header 'Content-Type: application/json' \
--data-raw '{
    "id": "c4fodmqo889qjstf7ibg", 
    "cert": null,
    "type": "public",
    "enabled": true,
    "redirectUrls": [https://www.certification.openid.net/test/a/OB-EKS-DEV/callback"],
    "corsOrigins": ["*"],
    "applicationId": "5e321c00-5e5e-4167-a295-dc05e40c4e50"
}'
```

>Note that the _applicationId_ appears both in the URL request (POST `https://hostname:port/api/portal/v1.3/applications/$applicationId/oauth`) and in the JSON data as _applicationId_. And the _client-id_ used for oauth appears in the JSON data as _id_

### Certificate configuration

Make sure the corresponding CA are configured in the Axway Open Banking solution, by following each section of [Certificate Management](/docs/configuration/certificate-management)

### Create the DCR test plan

Create a new plan and select Dynamic Client Registration Authorization server test:
![Conformance Suite : Create Plan](/Images/conformance-suite/dcr-plan-select.png)

Use the JSON tab to import this sample JSON test configuration : [conformance-test-dcr.json](https://axway-open-banking-docs.netlify.app/sample-files/conformance-test-dcr.json)

Then go back to the Form tab to customize the following :

* Test Information : include your company name and/or environment info. decide wether you want to publish the result or not.
* Server: provide the Authorization server URL (keep /.well-known/openid-configuration)
* Client : Change jwks with RSEAL information
* TLS certificates for client :  change certificate and key with BRCAC information
* Directory: change client ID with the new TPP client ID to register.

## FAPI Advanced Tests

### TPP configurations

For this test you need to have 2 registrered TPPs.

Make sure you have the following informations from [Central Directory Sandbox](https://web.sandbox.directory.openbankingbrasil.org.br/) for each TPP:

* BRSEAL – message certificate (cert and key) – used for JWKS .
* BRCAC – transport certificate – used for MTLS comunication

Make sure that the 2 test TPPs are configured in APIM:

* Connect to API Manager UI
* Under Client > application : you should see an application for each TPP. 
* In each application a OAuth clientId should be configured in the Authorization section.

Make sure that the 2 test TPPs are configured in ACP:

* Connect to ACP and select to workspace for Open Banking
* Under Application, you be able to find the each TPP with same application name as in APIM, and the same clientID
* Each application should be configured with a correct Redirect URI : `https://www.certification.openid.net/test/a/<test-alias>/callback` this test alias is the one used when creating th test plan later.
![app-details](/Images/acp-tpp-app-details.png)
* Each application should be configured with a correct client authorization method : 
    * choose _Private Key JWT_ for tests with JWT,
    * choose _TLS Client authentication_ for tests with MTLS.
![app-details](/Images/acp-tpp-auth-method.png)
* Each application should be configured with a correct client authorization details : 
![app-details](/Images/acp-tpp-auth-identifier.png)

### Solution configuration

Make sure the corresponding CAs are configured in the Axway Open Banking solution:

* Including the CA cert of each TPP on IngressMTLSCA entry of values.yaml of APIM package. See detailed instructions in [Certificate Management > MTLS](/docs/configuration/certificate-management/mtls)
* Including the CA cert of each TPP on ACP -> Settings -> Authorization tab. See detailed instructions in [Certificate Management > MTLS](/docs/configuration/certificate-management/mtls)
* Update the filters Jwt-Verify and Jwt-Sign, udpdating the JWKS certificate (bracac). See detailed instructions in [Certificate Management > JWKS](/docs/configuration/certificate-management/jwks)

### Create the FAPI Advanced test plan

Create a new plan and select FAPI Authorization server test:
![Conformance Suite : Create Plan](/Images/conformance-suite/fapi-plan-select.png)

Use the JSON tab to import this sample JSON test configuration : [conformance-test-fapi.json](https://axway-open-banking-docs.netlify.app/sample-files/conformance-test-fapi.json)

Then go back to the Form tab to customize the following :

* Test Information : include your company name and/or environment info. the alias should be the same as in apps Redirect URI. decide wether you want to publish the result or not.
* Server: provide the Authorization server URL (keep /.well-known/openid-configuration)
* Client : Set OAuth client Id of TPP1, jwks with RSEAL information of TPP1
* TLS certificates for client :  change certificate and key with BRCAC information of TPP1
* Second client : Set OAuth client Id of TPP2, jwks with RSEAL information of TPP2
* Second client TLS certificates :  change certificate and key with BRCAC information of TPP2.
* Resource : update URL with domain names, update organization id.