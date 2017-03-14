Pipelines are repeatable proceses comprised of stages that do things like make changes to your infrastructure or run outside jobs. They can be configured to be triggered by outside actions or scheduled on a cron.

To create a pipeline, go to the PIPELINES section, where you can manage pipelines and view details of current and previous pipeline runs.

# Pipeline Stages

### Bake
Bakes an image in the specified region

### Check Preconditions
Checks for precuditions before continuing

### Clone Server Group
Clones a server group

### Deploy
Deploys the previously baked or found image

### Destroy Server Group
Destroys a server group

### Disable Cluster
Disables a cluster

### Disable Server Group
Disables a server group

### Enable Server Group
Enables a server group

### Find Image from Cluster
Finds an image to deploy from an existing cluster

### Find Image from Tags
Finds an image to deploy from existing tags

### Jenkins
Runs a Jenkins job

### Manual Judgment
Waits for user approval before continuing

### Modify Scaling Process
Suspend/Resume Scaling Processes

### Pipeline
Runs a pipeline

### Resize Server Group
Resizes a server group

### Run Job
Runs a container

### Scale Down Cluster
Scales down a cluster

### Script
Runs a script

### Shrink Cluster
Shrinks a cluster

### Tag Image
Tags an image

### Wait
Waits a specified period of time


# Triggering Pipelines

Pipelines can be triggered by the following:

### CRON
Specify a cron expression to schedule the pipeline.

### Docker Registry
Specify the registry/organization/image and optionally a tag to trigger the pipeline. Leaving the tag empty triggers the pipeline on any new tag that is pushed to the registry.

### Git
Works with GitHub and Stash. Specify organization/username (e.g. spinnaker), project (e.g. echo) and branch.

### Jenkins
Specify the Jenkins instance and job that should complete successfully for the pipeline to trigger.

### Pipeline
Specify the application and pipeline that should trigger this pipeline. Can trigger on when the upstream pipeline completes successfully or failed, or is canceled.


# Pipeline Notifications

You can set notifications to be sent via any of the notifications services that are configured, and you can set multiple notifications. Notifications can be sent when a pipeline is starting, has completed, and/or has failed.

