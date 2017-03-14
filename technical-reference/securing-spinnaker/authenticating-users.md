Authentication is proving that a user is who they say they are. Spinnaker has many other responsibilities, so we prefer to delegate this need to an external identity provider. This has the added advantage of not requiring users to create and remember yet another username/password combination.

Spinnaker supports 3 forms of user authentication (some with authorization capabilities):
* [OAuth 2.0](#section-oauth-2-0)
* [SAML 2.0](#section-saml-2-0)
* [X.509 Certificates](#section-x-509)

# OAuth 2.0
Spinnaker includes OAuth 2.0 profile templates for Google (`googleOAuth`), Azure (`azureOAuth`), and GitHub (`githubOAuth`). This is achieved using Spring profiles, which we will configure and activate below. Any of the settings in the pre-configured profiles can be overridden in your `gate-<providerProfile>.yml` file. We will use the Google provider in this tutorial.

##  OAuth Provider
1. Navigate to [https://console.developers.google.com/apis/credentials](https://console.developers.google.com/apis/credentials).
2. Create a new OAuth client ID.
3. Select "Web Application", and enter a name.
4. Under "Authorized redirect URIs", add `https://localhost:8084/login`, replacing "localhost" with your Gate hostname, if applicable. Click Create.
5. Note the generated client ID and client secret. Copy these to a safe place (like the config file you'll create in the next section).

![image](https://files.readme.io/tGGYpBY9Q0akkjGDlrwO_y22f2ycaxLR.png)

## Gate Configuration
1. Create a `gate-googleOAuth.yml` file in `/opt/spinnaker/config/`. These values can be obtained by creating a new OAuth Client ID using the [Cloud Developers Console](https://console.developers.google.com/apis/credentials).

```
spring:
  oauth2:
    client:
      clientId: my-client-id
      clientSecret: ssshhh-its-a-sekret
```

2. Activate the `googleOAuth` profile by adding an entry in `/etc/default/spinnaker`

```
# Spinnaker system defaults
# ... file omitted

GATE_OPTS="-Dspring.profiles.active=local,googleOAuth"
```


## Deck Configuration
1. Enable authentication in Deck via `spinnaker-local.yml`. It is also required to specify the user-accessible address of Deck

```
services:
  deck:
    baseUrl: https://localhost:9000
    auth:
      enabled: true
```

2. Generate the new `settings.js` file for Deck:
```
sudo /opt/spinnaker/bin/reconfigure_spinnaker.sh
```
3. Restart Gate.
4. Navigate to Deck (generally `https://localhost:9000`). Notice the "Authenticating" screen that then redirects to the Google.

![image](https://files.readme.io/z8ARmUqhR7GfoC0FSSuT_auth.png)

![image](https://files.readme.io/epetJEQmQgSpTh6O0r8U_glogin.png)

## Bring-Your-Own OAuth Provider

If you'd like to configure your own OAuth provider, you'll need to provide the following configuration values in your `gate-local.yml` file:

```
spring:
  oauth2:
    client:
      clientId:
      clientSecret:
      userAuthorizationUri: # Used to get an authorization code
      accessTokenUri: # Used to get an access token
      scope:
    resource:
      userInfoUri: # Used to the current user's profile
    userInfoMapping: # Used to map the userInfo response to our User
      email: 
      firstName:
      lastName:
      username:
```

#  SAML 2.0
Spinnaker also provides SAML support. If you previously had SAML configured, please consult the [Gate SAML Authentication Migration Guide](doc:gate-saml-config). 

## Identity Provider Configuration
These are the general steps to take on your Identity Provider's Administrative console.

1. Download the `metadata.xml` file from the identity provider. It should look something like

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<md:EntityDescriptor 
    xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata"    
    entityID="https://accounts.google.com/o/saml2?idpid=SomeValueHere" 
    validUntil="2021-05-16T15:17:27.000Z">
  <md:IDPSSODescriptor 
      WantAuthnRequestsSigned="false" 
      protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
    <md:KeyDescriptor use="signing">
      <ds:KeyInfo xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
        <ds:X509Data>
          <ds:X509Certificate>MIIDdDCCAlygAwIBAgIGAVS/Sw5yMA0GCSqGSIb3DQEBCwUAMHsxFDASBgNVBAoTC0dvb2dsZSBJ
bmMuMRYwFAYDVQQHEw1Nb3VudGFpbiBWaWV3MQ8wDQYDVQQDEwZHb29nbGUxGDAWBgNVBAsTD0dv
b2dsZSBGb3IgV29yazELMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWEwHhcNMTYwNTE3
MTUxNzI3WhcNMjEwNTE2MTUxNzI3WjB7MRQwEgYDVQQKEwtHb29nbGUgSW5jLjEWMBQGA1UEBxMN
TW91bnRhaW4gVmlldzEPMA0GA1UEAxMGR29vZ2xlMRgwFgYDVQQLEw9Hb29nbGUgRm9yIFdvcmsx
CzAJBgNVBAYTblVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMIIBIjANBgkqhkiG9w0BAQEF46OCAQ8A
MIIBCgKCAQEA4JsnpS0ZBzb7DtlU7Zop7l+Kgr7NzusKWcEC6MOsFa4Dlt7jxv4ScKZ/61M5WKxd
5YX0ol1rPokpNztj+Zk7OXrG8lDic0DpeDutc9pcq0+9/NYFF7WR7TDjh4B7Txnq7SerSB78fT8d
4rK7Bd+cu/cBIyAAyZ5tLeLbmTnHAk093Y9vF3mdWQnfAhx4ldOfstF6G/d2ev7I5xjSKzQuH6Ew
3bb3HLcM4uEVevOfNAlh1KoV4vQr+qzbc9UEFcPRwzuTwGa6QjfspWW7NgXKbHHC+X6a+gqJrke/
6l2VvHaQBJ7oIyt4PCdel2cnUkvuxvzHPYedh1AgrIiSP1brSQIDAQABMA0GCSqGSI34DQEBCwUA
A4IBAQCPqMAIau+pRDs2NZG1nGfyEMDfs0qop6FBa/wTNis75tLvay9MUlxXkTxm9aVxgggjEyc6
XtDjpV0onrH0jBnSc+vRI1GFQ48EO3owy3uBIeR1aMy13ZwAA+KVizeoOrXBJbvIUZHo0yfKRzIu
gtM58j58BdAFeYo+X9ds/ysvZ8FIGTLqMl/A3oO/yBNDjXR9Izoqgm7RX0JJXGL9Y1AgmEjxyqo9N
MhxZAGxOHm9HZWWfVMcoe8p62mRJ2zf4lkNPBnDHrQ8MDPSsXewAuiSnRBDLxhdBgyThT/KW7Q06
rGa6Dp0rntKWzZE3hGQS0AdsnuFY/OXbmkNG9WUrUg5x</ds:X509Certificate>
        </ds:X509Data>
      </ds:KeyInfo>
    </md:KeyDescriptor>
    <md:NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress</md:NameIDFormat>
    <md:SingleSignOnService 
        Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" 
        Location="https://accounts.google.com/o/saml2/idp?idpid=SomeValueHere"/>
    <md:SingleSignOnService 
        Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" 
        Location="https://accounts.google.com/o/saml2/idp?idpid=SomeValueHere"/>
  </md:IDPSSODescriptor>
</md:EntityDescriptor>
```

2. Create a Spinnaker SAML application.
3. Specify the login URL as `https://localhost:8084/saml/SSO`.
4. Specify a unique entity ID (we'll use `io.spinnaker:test` in our example).
5. Enable the users you'd like to have access to your Spinnaker instance.

## Gate Configuration
1. Generate a keystore and key in a new Java Keystore with some password:
```
keytool -genkey -v -keystore saml.jks -alias saml -keyalg RSA -keysize 2048 -validity 10000
```
2. Create or modify your `gate-local.yml` file to include the following settings:

```
saml:
  enabled: true
  metadataUrl: file:/opt/spinnaker/config/metadata.xml
  keyStore: file:/opt/spinnaker/config/saml.jks
  keyStorePassword: hunter2
  keyStoreAliasName: saml
  issuerId: io.spinnaker:test
  redirectHostname: localhost:8084

```


## Deck Configuration
1. Enable authentication in Deck via `spinnaker-local.yml`. It is also required to specify the user-accessible address of Deck.

```
services:
  deck:
    baseUrl: https://localhost:9000
    auth:
      enabled: true
```

2. Generate the new `settings.js` file for Deck:
```
sudo /opt/spinnaker/bin/reconfigure_spinnaker.sh
```
3. Restart Gate.
4. Navigate to Deck (generally `https://localhost:9000`). Notice the "Authenticating" screen that then redirects to your SAML Provider.

![image](https://files.readme.io/kZLq0mstSmCO1H8Nuc1y_auth.png)

![image](https://files.readme.io/MIoEORFSHy0ZcBLXG9L1_oktaAuth.png)

# X.509
Client certificates are the last way to authenticate users. It is the preferred way to access Spinnaker's API endpoints via scripts. Remember that Certificate Authority (CA) we created in the [Part 1](#section-certificate-authority)? We'll use it to create a new client certificate. 

## Client Certificate
1. Create the client key. Keep this file safe!
```
openssl genrsa -des3 -out client.key 4096
```
2. Generate a certificate signing request for the server.
```
openssl req -new -key client.key -out client.csr
```
3. Use the CA to sign the server's request. If using an external CA, they will do this for you.
```
openssl x509 -req -days 365 -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt
```
4. Format the client certificate into browser importable form.
```
openssl pkcs12 -export -clcerts -in client.crt -inkey client.key -out client.p12
```

## Browser Configuration
1. In Google Chrome, navigate to [chrome://settings/certificates](chrome://settings/certificates), click the `Authorities` tab, and import the CA.

![image](https://files.readme.io/nV3yLHOmQUSybp2JJ41e_hys5KjMaXKe.png)

2.  Click the `Your Certificates` tab, and import your newly signed client certificate (`client.crt

## Gate Configuration
Enable X.509 in `gate-local.yml` with the following `x509` tree as well as the new trustStore settings. Restart Gate after making these changes.

```
x509:
  enabled: true
  subjectPrincipalRegex: EMAILADDRESS=(.*?)(?:,|$) # optional
  
server:
  ssl:
    enabled: true
    keyStore: /opt/spinnaker/config/keystore.jks
    keyStorePassword: hunter2
    keyAlias: server
    trustStore: /opt/spinnaker/config/keystore.jks
    trustStorePassword: hunter2
```

## Multiple Authentication Mechanisms

X.509 can be enabled independent of OAuth  and SAML. In this scenario, users hitting Deck will be prompted through the OAuth/SAML Identity Provider login screen, and API users (scripts) can simultaneously make calls with X.509 certificates.

