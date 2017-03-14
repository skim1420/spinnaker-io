# Configuring Notifications (Echo)

Out of the box, Spinnaker allows you to configure the following types of notifications:

* Email
* HipChat
* Slack
* SMS ( via Twilio )

This is discussed in the Configuring Notifications section below.

Additionally, Spinnaker allows you to set webhooks for git triggers. See the [http://www.spinnaker.io/docs/notifications-and-events-guide#section-setting-up-git-triggers-in-spinnaker] section.  

You can also set Spinnaker to stream all its events to a downstream listener. See the [Add a Webhook to Spinnaker](http://www.spinnaker.io/docs/notifications-and-events-guide#section-add-a-listening-webhook-to-spinnaker) section.

Additionally, Spinnaker is capable of handling cron-based triggers and detect changes in Jenkins builds and Docker images. This functionality will be documented at a later time. 

Notification configurations are set in echo.yml and settings.js. On a single instance virtual machine installation, notification values can also be set in spinnaker-local.yml, which will pass the details to echo.yml and settings.js.

You will need to set the `spinnaker.baseUrl` configuration value which is used by spinnaker templates. This should point back to the url for your spinnaker's UI ( deck ) instance. This url is used in notifications to link back to your spinnaker instance.

## Email

Email in spinnaker is provided by [Spring Boot Mail starter](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-email.html). The following is an example of using Gmail to send notifications.

in echo.yml
```
mail:
  enabled: true
  from: xxxx@gmail.com
spring:
  mail:
    host: smtp.gmail.com
    username: xxxx@gmail.com
    password: [ App Password - https://support.google.com/accounts/answer/185833?hl=en ]
    properties:
      mail:
        smtp:
          auth: true
          ssl:
            enable: true
          socketFactory:
            port: 465
            class: javax.net.ssl.SSLSocketFactory
            fallback: false
```
in settings.js (deck)
```
window.spinnakerSettings = {
// ...
  notifications: {
    email: {
      enabled: true
    },
// ...
```


## HipChat

For Hipchat, you will need to create a hipchat [authentication token](https://www.hipchat.com/docs/apiv2/auth) that is able to post messages. 

in echo.yml
```
hipchat:
  enabled: true
  baseUrl: https://xxx.hipchat.com
  token: <authToken>
```
in settings.js (deck)
```
window.spinnakerSettings = {
// ...
  notifications: {
    hipchat: {
      enabled: true,
      botName: '<username of bot>'
    },
// ...
```


Note: your users will need to invite the hipchat bot to private rooms that want to be notified.

## Slack

For slack, you will need to create a custom bot user (https://api.slack.com/bot-users#how_do_i_create_custom_bot_users_for_my_team), then get the access token associated with the new bot user. 

in echo.yml
```
slack:
  enabled: true
  token: <API token for bot>
```

in settings.js (deck)
```
window.spinnakerSettings = {
// ...
  notifications: {
    slack: {
      enabled: true,
      botName: '<username of bot>'
    },
// ...
```

Note: your users will need to invite the slack bot to private rooms that want to be notified.

## Twilio

For twilio, you need to add your account [credentials](https://www.twilio.com/help/faq/twilio-basics/what-is-the-auth-token-and-how-can-i-change-it). 

in echo.yml
```
twilio:
  enabled: true
  baseUrl: https://api.twilio.com/
  account: xxx
  token: xxx
  from: +18sp-inn-aker
```
in settings.js (deck)
```
window.spinnakerSettings = {
// ...
  notifications: {
    sms: {
      enabled: true
    },
// ...
```

## Using Notifications

Once notifications have been configured, you can use them to send changes in pipelines and in the manual judgment stage in Spinnaker.

To set up an application-wide notification, go to Application -> Config -> Notifications, 

Click on 'Add Notification'

