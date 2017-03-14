First, you'll need to have a GCP project set up. If you've already got one, please skip to the next step. Otherwise, please follow the
instructions below.

Sign into the [Google Developers Console](https://console.developers.google.com) and create a
project. Use your project name in place of <code>my-spinnaker-project</code> below.

1. Enable APIs in the <code>my-spinnaker-project</code> project.
  * Go to the API Management page.
  * Enable the [Compute Engine](https://console.developers.google.com/apis/api/compute_component/overview?project=_)
    and [Compute Engine Autoscaler](https://console.developers.google.com/apis/api/autoscaler/overview?project=_) APIs.
    

2. Obtain service account credentials.
  * This step is only required to manage your GCP project from Spinnaker running outside that project (e.g. Spinnaker is running on AWS or in a different GCP project).
  * Go to the Credentials tab on the API Management page.
  * Select the **Service account key** item from the **New credentials** menu.
  * Select a service account, the **JSON** key type, and click **Create**.
  * Safeguard the JSON file that your browser will download. We will later
    copy this into your Spinnaker deployment so that it can manage your
    GCP project.

**Basic Configuration**

The [GCP click to deploy](doc:creating-a-spinnaker-instance#section-google-cloud-platform) solution will configure Spinnaker to be able to deploy to the project the instance running Spinnaker exists in. If you want to tweak another installation to deploy to GCP, pick your favorite editor and open <code>/opt/spinnaker/config/spinnaker-local.yml</code>. Under the <code>providers</code> section, you'll find the <code>google</code> subsection, which we've annotated with comments:
  
    google:
      enabled:             # set to true to deploy to GCP
      defaultRegion:       # default region to target for deployments
      defaultZone:         # default zone to target for deployments
      primaryCredentials:  
        name:              # used to identify this set of credentials later
        project:           # project ID of the project you want to manage
        jsonPath:          # path to the .json service account key for the above project;
                           # must be owned by Spinnaker

Once you made the necessary changes (probably updating, <code>enabled</code>, <code>project</code>, and <code>jsonPath</code>), restart Clouddriver with <code>sudo restart clouddriver</code>.

**Advanced Configuration**

To configure Spinnaker to deploy to multiple GCP projects, append to the <code>google</code> block in <code>/opt/spinnaker/config/spinnaker-local.yml</code>:

```yml
google:
  # ...
  # previous config
  # ...
  accounts:
    - name:                # name of Spinnaker account for project 1
      project:             # name of GCP project 1
      jsonPath:            # path to json key for service account in project 1
    - name:                # name of Spinnaker account for project 2
      project:             # name of GCP project 2
      jsonPath:            # path to json key for service account in project 2
```

