The current security model allows all authenticated users to see all resources (read access) within all accounts and applications. If enabled, specific accounts can be locked down to only allow users who are members of a certain group to perform mutating operations (write access). As of this writing, only groups from a SAML identity provider or Google Groups within your Google Apps for Work account are supported.

## Google Apps for Work (GAFW)
A prerequisite for authorization using Google Groups is a Google Apps for Work account. You can [sign up here](https://apps.google.com) if your organization does not have one.

### Assign groups

1. In the [Admin Console](https://admin.google.com), open the left navigation and click "Groups." Here, you can add groups that specify various roles in Spinnaker.
2. Create a new group with the plus icon and give it a name.

3. Add users to your newly created group.

### Configure Domain-wide Delegation

Gate uses a Google service account to access the Admin APIs on your behalf. We will create and configure this service account below. If you are running on Google Compute Engine (GCE) and already have a service account, you can reuse the same service account instead of creating a new one.

1. Follow the instructions to [create the service account and its JSON credentials](https://developers.google.com/admin-sdk/directory/v1/guides/delegation#create_the_service_account_and_its_credentials).
2. Follow the instructions to [delegate Domain-wide access to the service account](https://developers.google.com/admin-sdk/directory/v1/guides/delegation#delegate_domain-wide_authority_to_your_service_account). Your service account will need _ONLY_  the **https://www.googleapis.com/auth/admin.directory.group.readonly** scope.
3. Make sure you [enable the Admin SDK](https://console.cloud.google.com/apis/api/admin/overview).

### Configure Gate

Now that we have all the Google setup finished, we need to configure Spinnaker to
enforce the roles we have set up. In your Spinnaker instance, add the following
section to your `gate-local.yml` file:

```
auth:
  groupMembership:
    service: google
    google:
      credentialPath: /path/to/JSON.key
      adminUsername: admin@mydomain.com
      domain: mydomain.com
```

Once you have this configuration complete, restart Spinnaker and enjoy your enforced authN/Z roles!
Each time you log into Spinnaker, you will be redirected to a Google sign-in page if you do not currently have a session. If you do have a session, Gate will enforce your required group memberships to access Spinnaker resources.

### Configure Clouddriver
The last step is to ensure Clouddriver knows to restricted access for particular accounts to those users with access. If you specify multiple groups, a user only needs to be a member of *one* of the specified groups to be granted access. 

Create a `clouddriver-local.yml` file to override and enhance the `clouddriver.yml` file from the [spinnaker/spinnaker config](https://github.com/spinnaker/spinnaker/blob/master/config/clouddriver.yml). An example is shown below:

```
google:
  accounts:
  - name: my-protected-google-account
    project: my-spinnaker-project
    jsonPath: /opt/spinnaker/config/gafw-service-account.json
    requiredGroupMembership:
    - spinnaker-users
```

## Test Driving your changes
Restart Gate and Clouddriver to have these changes take effect. If everything is configured correctly, mutating calls, such as "Terminate Instances" should return an error in the UI like below.

![image](https://files.readme.io/dGZa7AIaTtKGFXcRXCrY_authdenied (1).png)



