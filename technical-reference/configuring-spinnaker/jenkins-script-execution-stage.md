### Purpose

The purpose of this document is to provide instructions on how
to set up and run the Jenkins script execution stage in Spinnaker (referred to as "script stage").

The script stage lets a Spinnaker user run an arbitrary shell, python, or 
groovy script on a Jenkins instance as a first class stage in Spinnaker.
This is good for launching an integration/functional test battery
 after a bake and deploy stage from a pipeline instead of doing it manually.

### Assumptions

There are a few assumptions we make in the following directions:

* You have a running Spinnaker instance, with access to configuration files.

* You have a running Jenkins instance at `<jenkins_host>`, with a user profile set up with admin access.


### Configuring Jenkins

* `ssh` into your Jenkins machine.

* `wget` or `curl` the [raw job xml config file](https://storage.googleapis.com/jenkins-script-stage-config/scriptJobConfig.xml).

* To create the Jenkins job, run:
```shell
curl -X POST -H "Content-Type: application/xml" -d @scriptJobConfig.xml \
"http://<username>:<user_api_token>@<jenkins_host>/jenkins/createItem?name=<JOB_Name>"
```
where `<JOB_NAME>` is the name of the Jenkins job you create, e.g. "runSpinnakerScript"
and `<user_api_token>` is the API token for your user, located at "/user/`<username>`/configure".

* In the job config in the Jenkins UI, set the GitHub repository containing your scripts as
well as the git credentials.

* In the UI, go to "Manage Jenkins" >> "Configure System" and set your git `user.name` and `user.email`.

At this point, you should be able to manually run the script job in Jenkins
(with parameters) and see it succeed.

### Configuring Spinnaker

* Enable Igor.

* In `spinnaker-local.yml`, set:
  - `jenkins.enabled = true`
  - `jenkins.defaultMaster.name = <jenkins_name>`
  - `jenkins.defaultMaster.baseUrl = http://<jenkins_host>/jenkins` Note that "/jenkins" might not be the base path, it depends on how Jenkins is configured.
  - `jenkins.defaultMaster.username = <username>`
  - `jenkins.defaultMaster.password = <password>`

* In `orca.yml`, add:
```yml
script:
    master: <jenkins_name> # name of Jenkins master in Spinnaker
    job: <JOB_NAME> # from Jenkins job configuration
```

* Restart Orca and Igor if you didn't have a Jenkins master
configured in Spinnaker.

### Summary

You should now be able to add a stage called "Script" to your pipelines,
where you can specify:

* Repository Url: git repository housing your scripts.
* Script path: path from the root of your git repository to your script's
directory.
* Command: name of the script with arguments to run.

Among other environment parameters (e.g. image, account, etc).

The current version of the script stage is a bit rudimentary, but we'll
soon have support for a separate "job" stage that will be much more robust and encapsulate
the same behavior as the script stage. The script stage is a temporary
solution for the time being.

