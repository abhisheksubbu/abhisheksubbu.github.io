---
title: Two Way SSL in Salesforce (Outbound)
layout: blog
category: [Salesforce, Security]
excerpt: Securely connecting to an external system from Salesforce is very important in enterprise setup. In this blog, I will explain how to develop mutual authentication from Salesforce using certificate bundles.
comments: true
---

In traditional authentication, any client can connect to a server. The clients know the server which is why they can request the server for some action to be performed. The server doesn't know which client is connecting to it. This is fine when server is hosting a website and client requests the website.

But if we have an API hosted on the server and we allow any client to access it, there is a hidden vulnerability that can possibly exploit the server since the server doesn't know how to validate which client is allowed to talk to the server.

Two Way SSL was introduced to solve this security hole. This technique is also popularly known as "Mutual Authentication". Here, the client has to present a certificate in the request to the server. Server will only allow the request to be processed if the server identifies & validates the certificate presented by the client. Therefore, server knows which clients will approach it and can deny all others who provide incorrect certificate or no certificate.

## Scenario
Salesforce wants to call a REST API exposed by an external system. External System imposes 2 way SSL on every request. The External System provides few certificates which has to be used by the client while initiating calls to it.

## Pre-Requisites
1. You should have **OPENSSL** installed in your local machine
2. You should have **JDK version 8** (not the latest version) installed in your local machine. This is for accessing keytool.
3. You should obtain the following from the external system provider.
    1. Private key file (**privateKey.pem**)
    2. Application Certificate (**cert.pem**)
    3. Any additional certificates like platform certs (**SYSCA.pem**) & CA certs (**ROOTCA.crt**).

## Implementation Steps
#### STEP 1 - Prepare the JKS (Java Keystore) bundle using the certificates.
2. Run the command in terminal to obtain a p12 bundle
``` console
openssl pkcs12 -export -in cert.pem -inkey "privateKey.pem" -certfile cert.pem -out myBundle.p12
```
3. Run the command in terminal to convert the p12 bundle to a jks bundle
``` console
keytool -importkeystore -srckeystore myBundle.p12 -srcstoretype PKCS12 -destkeystore myBundle.jks
```
4. Run the command in terminal to list the contents of the jks bundle. It should just have the cert.pem added into it.
``` console
keytool -list -v -keystore myBundle.jks
```
5. Run the command in terminal to rename any numeric alias to alphabets. This is required as Salesforce will not accept any jks bundle that has number in its alias.
``` console
keytool -keystore myBundle.jks -changealias -alias 1 -destalias MYBUNDLE
```
6. Run the command in terminal to add the SYSCA cert (in other words, platform cert) to the jks bundle
``` console
keytool -import -alias ejbca -keystore myBundle.jks -file SYSCA.pem -storepass mybundle123
```
7. Run the command in terminal to add the ROOTCA cert to the jks bundle
``` console
keytool -import -alias digicert -keystore myBundle.jks -file RootCA.crt -storepass mybundle123
```

Now, you have a jks bundle which houses the application cert, sysCA cert & rootCA cert. This is now called a JKS with certificate chain.

#### STEP 2 - Implementation in Salesforce
1. Navigate to Setup --> Security --> Certificate and Key Management.
2. Click the button named "Import from Keystore"
3. Upload the JKS bundle file and enter the keystore password that we used in Step 6 and Step 7
4. Click Save
5. Navigate to Setup -> Security -> Named Credentials and create a New Named Credential as shown below.

    > Assumption here is that External System uses Username-Password Authentication. Based on your use-case, you can change the Authentication Protocol.

    ![Named Credential](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/mybundle-named-credential.png)

6. Open the Developer Console and go to Debug -> Open Execute Anonymous Window
7. Type the following code and execute
```java
    HttpRequest req = new HttpRequest();
    req.setEndpoint('callout:External_System/users');
    req.setMethod('GET');
    Http http = new Http();
    HTTPResponse res = http.send(req);
    System.debug(res.getBody());
```
8. You should get a proper response in the debug logs if the external system is up & running properly.

> Without the MYBUNDLE certificate in the Named Credential, the same code when executed will throw an error.