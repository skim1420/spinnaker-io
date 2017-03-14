This section will discuss a number of strategies for securing your Spinnaker installation, including transport encryption and authentication.

### Contents
* [Encrypting Communication with Secure Socket Layer (SSL)](#section-encrypting-communication-with-secure-socket-layer-ssl-)
* [Configuring Session Timeout](#section-configuring-session-timeout)
* [Troubleshooting](#section-troubleshooting)


# Encrypting Communication with Secure Socket Layer (SSL)

This section will cover communication external to your Spinnaker instance. That is, any requests between your browser and the Deck (the UI) host, between Deck and Gate (the API gateway), and between any other client and Gate.

We will use `openssl` to generate a Certificate Authority (CA) and a server certificate.


### Certificate Authority
Use the steps below to create a certificate authority. If you're using an external CA, skip to the next section. 

1. Create the CA key.
```
openssl genrsa -des3 -out ca.key 4096
```
2. Self-sign the CA certificate.
```
openssl req -new -x509 -days 365 -key ca.key -out ca.crt
```

### Server Certificate
1. Create the server key. Keep this file safe!
```
openssl genrsa -des3 -out server.key 4096
```
2. Generate a certificate signing request for the server. Specify `localhost` or Gate's eventual fully-qualified domain name (FQDN) as the Common Name (CN). 
```
openssl req -new -key server.key -out server.csr
```
3. Use the CA to sign the server's request. If using an external CA, they will do this for you.
```
openssl x509 -req -days 365 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt
```
4. Format server certificate into Java Keystore (JKS) importable form.
```
openssl pkcs12 -export -clcerts -in server.crt -inkey server.key -out server.p12
```
5. Create Java Keystore by importing CA certificate
```
keytool -keystore keystore.jks -import -trustcacerts -alias ca -file ca.crt
```
6. Import server certificate
```
keytool -importkeystore -srckeystore server.p12 -srcstoretype pkcs12 -srcalias 1 -destkeystore keystore.jks -deststoretype jks -destalias server
```

Voilà! You now have a Java Keystore with your certificate authority and server certificate ready to be used by Spinnaker!

## Gate Configuration
You need to configure Gate to read and use this file. We'll assume `keystore.jks` is in the same directory as our Gate configuration files. 

In the Spinnaker configuration file directory (`/opt/spinnaker/config`), create a `gate-local.yml` file and use a similar configuration as the one below:

```
server:
  ssl:
    enabled: true
    keyStore: /opt/spinnaker/config/keystore.jks
    keyStorePassword: hunter2
    keyAlias: server
```

Restart Gate and confirm `https://localhost:8084/` is available

## Deck Configuration
Since Deck is served as static files from Apache, we must configure Apache to serve those files over HTTPS. 

1. Enable `mod-ssl` on Apache
```
sudo a2enmod ssl
```
2. Edit your `/etc/apache2/sites-available/spinnaker.conf` file to add your server certificate and key

```
<VirtualHost 127.0.0.1:9000>
  SSLEngine on
  SSLCertificateFile "/opt/spinnaker/config/server.crt"
  SSLCertificateKeyFile "/opt/spinnaker/config/server.key"

  // ... rest of file omitted
</VirtualHost>
```

3.  Point Deck to the newly secured Gate endpoint. We'll use the preferred way of generating the `settings.js`, rather than editing it directly. In your `spinnaker-local.yml` file:

```
services:
  deck:
    gateUrl: https://localhost:8084
```

4. Regenerate the `settings.js` file.
```
sudo /opt/spinnaker/bin/reconfigure_spinnaker.sh
```
5. Restart Deck
```
$ sudo service apache2 restart
Apache needs to decrypt your SSL Keys for localhost:9000 (RSA)
Please enter passphrase:
```

# Configuring Session Timeout

By default, a user's session will expire after 30 minutes of inactivity. You can change this duration by adding an entry in the `gate.yml` file:

```
server:
  session:
    timeoutInSeconds: 3600
```

# Troubleshooting

### Something's not right. How do I turn on debug logs?
Add the following to your configuration to turn on logging. 

```
logging:
  level:
    com.netflix.spinnaker.gate.security: DEBUG
    org.springframework.security: DEBUG
    org.springframework.web: DEBUG
```

### I see Deck requesting `/auth/info` and hanging. What gives?

This is the old endpoint. You'll need to update your `settings.js` file. There are two ways to do this.
1. Edit your `/opt/spinnaker/config/settings.js` file and change the `authEndpoint` to: 

```
window.spinnakerSettings = {
  // ...
  authEndpoint: gateUrl + '/auth/user',
  //...
}
```

and run this to regenerate the `/opt/deck/html/settings.js` file:
```
sudo /opt/spinnaker/bin/reconfigure_spinnaker.sh
```

2. Update your `AUTH_ENDPOINT` environment var to end with `/auth/user`. This is likely applicable if you're using Docker Compose or a development instance.

---

### I'm getting an `Error: redirect_uri_mismatch` from my OAuth provider.

The full error may look something like:
```
Error: redirect_uri_mismatch. The redirect URI in the request, https://<some_url>/login, 
does not match the ones authorized for the OAuth client.
```

This likely means you've not set up your OAuth credentials correctly. Ensure that the Authorized Request URIs list contains "http**s**://my-gate-address/login" (no trailing /).

In the case that this is set correctly, see the next answer.

---

### I terminate SSL connections outside of Gate, and my `redirect_uri` configuration is correct (see above). How do I make this work?

By default, Gate will use its own protocol, host, and port when constructing the `redirect_uri`. In the case where you want to terminate SSL connections outside of Gate, you'll need to override this by providing the exact address in your gate-<provider>.yml:

```
spring:
  oauth2:
    client:
      preEstablishedRedirectUri: https://my-real-gate-address.com/login
      useCurrentUri: false
```

 ---

### I'm able to login to my identity provider, but afterwards I get a `Requested redirect address not recognized` error

In your `spinnaker-local.yml` file, you need to set `services.deck.baseUrl` to the address of Deck. We only allow redirects to URLs that match the protocol, host, and port of the `services.deck.baseUrl`. For example:  

```
services:
  deck:
    baseUrl: https://localhost:9000
```


