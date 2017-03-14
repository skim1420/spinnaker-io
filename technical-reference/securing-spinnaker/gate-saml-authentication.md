As of release 2.54.0, Gate has changed the way the SAML authentication mechanism is configured. This guide will help previous installations move to the new configuration with minimal downtime.

## YAML Configuration Changes 
An example of a previous configuration:

```
saml:
  enabled: true
  requireAuthentication: false
  url: https://my.saml.provider.com/idp/SSO.saml2
  keyStore: /path/to/my/keystore.jks
  keyStorePassword: hunter2
  keyStoreAliasName: server
  issuerId: spinnaker.prod
  userAttributeMapping:
    roles: googleGroups
  certificate: "\
BLAHBLAH
BLAHBLAH"
```

Changes required:
* `requireAuthentication` is no longer supported. Authentication is always required when SAML is configured.
* `url` and `certificate` have merged into `metadataUrl`, which is the URL endpoint of the Identity Provider's metadata.xml file. If your provider does not expose this metadata file publicly, you can download the file and use the prefix `file:` to point to this local copy.  
* `keyStore` must now be prefixed with `file:` if the file is not on the classpath. 
* `redirectHostname` is used to construct the redirect endpoint sent to the IdP. It should be the hostname of your Gate instance.

An example of a newly migrated configuration:

```
saml:
  enabled: true
  metadataUrl: https://my.saml.provider.com/sso/saml/metadata
  keyStore: file:/path/to/my/keystore.jks
  keyStorePassword: hunter2
  keyStoreAliasName: server
  issuerId: spinnaker.prod
  redirectHostname: spinnaker.mydomain.com:8084 # can use localhost:8084 for local development
  userAttributeMapping:
    roles: googleGroups
```

## Identity Provider (IdP) Changes
Previously, SAML Identity Providers needed to be have their destination URL set to `/auth/signIn`. Now, they must use `/saml/SSO`

## Deck Changes
Deck has a few small changes as well to get the UI login flow going too. In your `settings.js` file:

```
window.spinnakerSettings = {
  gateUrl: ... // change to HTTPS as necessary
  authEnabled: true,
  authEndpoint: gateUrl + '/auth/user',
  // ... the rest
}
```

