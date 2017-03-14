In this codelab you will be creating a set of basic pipelines for deploying code from a Github repo to a Kubernetes cluster in the form of a Docker container.

Given that there are a number of fully-featured docker registries that both store and build images, Spinnaker doesn't build Docker images but instead depends on any registry that does.

The workflow generally looks like this:
  1. Push a new tag to your registry. (Existing tag changes are ignored for the sake of traceability - see below).
  2. Spinnaker sees the tag and deploys the new tag in a fresh Replication Controller, and optionally deletes or disables any old Replication Controllers running this image.
  3. The deployment is verified externally.
  4. Spinnaker now redeploys this image into a new environment (production), and disables the old version the Replication Controller was managing

# 0. Setup

We need a few things to get this working. 

  1. [A Github repo containing the code we want to deploy](http://www.spinnaker.io/v1.0/docs/kubernetes-source-to-prod#section-configuring-github).
  2. [A Dockerhub repo configured to build on changes to the above repo](http://www.spinnaker.io/v1.0/docs/kubernetes-source-to-prod#section-configuring-dockerhub).
  3. [A running Kubernetes cluster](http://www.spinnaker.io/v1.0/docs/kubernetes-source-to-prod#section-configuring-kubernetes).
  4. [A running Spinnaker deployment configured with the contents of steps 2 and 3](http://www.spinnaker.io/v1.0/docs/kubernetes-source-to-prod#section-configuring-kubernetes).

## Configuring Github

The code I'll be deploying is stored [here](https://github.com/lwander/spin-kub-demo). Feel free to fork this into your own account, and make changes/deploy from there. What's needed is a working <code>Dockerfile</code> at the root of the repository that can be used to build some artifact that you want to deploy. If you're completely unfamiliar with Docker, I recommend starting [here](https://docs.docker.com/engine/getstarted/).

## Configuring Dockerhub

Create a new [repository on Dockerhub](https://hub.docker.com/). [This guide](https://docs.docker.com/docker-hub/builds/) covers how to get your Github repository hooked up to your new Dockerhub repository by creating an automated build that will take code changes and build Docker images for you. In the end your repository should look something like [this](https://hub.docker.com/r/lwander/spin-kub-demo/).

## Configuring Kubernetes

Follow one of the guides [here](http://kubernetes.io/docs/getting-started-guides/). Once you are finished, make sure that you have an up-to-date <code>~/.kube/config</code> file that points to whatever cluster you want to deploy to. Details on kubeconfig files [here](http://kubernetes.io/docs/user-guide/kubeconfig-file/).

## Configuring Spinnaker

We will be deploying Spinnaker to the same Kubernetes cluster it will be managing. To do so, follow the steps in [this guide](https://github.com/spinnaker/spinnaker/tree/master/experimental/kubernetes/simple), being sure to use [this section](https://github.com/spinnaker/spinnaker/tree/master/experimental/kubernetes/simple/#anything-else-except-for-ecr) to configure your registry.

# 1. Create a Spinnaker Application

Spinnaker applications are groups of resources managed by the underlying cloud provider, and are delineated by the naming convention `<app name>-`. Since Spinnaker and a few other Kubernetes-essential pods are already running in your cluster, your _Applications_ tab will look something like this:

![image](https://files.readme.io/DGVwp6usRgSjEniNqNvd_applications.png)

Under the _Actions_ dropdown select _Create Application_ and fill out the following dialog:

![image](https://files.readme.io/jgatmNQLSFSbl53dYSGx_appfill.png)

You'll notice that you were dropped in this _Clusters_ tab for your newly created application. In Spinnaker's terminology a _Cluster_ is a collection of _Server Groups_ all running different versions of the same artifact (Docker Image). Furthermore, _Server Groups_ are Kubernetes [Replication Controllers](http://kubernetes.io/docs/user-guide/replication-controller/) (this is subject to change to either [Replica Sets](http://kubernetes.io/docs/user-guide/replicasets/), or [Deployments](http://kubernetes.io/docs/user-guide/deployments/) in the future).



![image](https://files.readme.io/ae7v5KS8RGsH6cI6USKL_clusterscreen.png)


# 2. Create a Load Balancer

We will be creating a pair of Spinnaker _Load Balancers_ (Kubernetes [Services](http://kubernetes.io/docs/user-guide/services/)) to serve traffic to our _dev_ and _prod_ versions of our app. Navigate to the _Load Balancers_ tab, and select _Create Load Balancer_ in the top right corner of the screen. 

First we will create the _dev_ _Load Balancer_:


![image](https://files.readme.io/VMwxEcKzTU6pd1UHZuhv_devlb.png)


Once the _dev_ _Load Balancer_ has been created, we will create an external-facing load balancer. Select _Create Load Balancer_ again:


![image](https://files.readme.io/x4tGpwX0T3SVlrsysejU_prodlb.png)

At this point your _Load Balancers_ tab should look like this:

![image](https://files.readme.io/qzY1pO6kTZeNQjdQH4J6_loadbalancers.png)

# 3. Create a Demo Server Group

Next we will create a _Server Group_ as a sanity check to make sure we have set up everything correctly so far. Before doing this, ensure you have at least 1 tag pushed to your Docker registry with the code you want to deploy. Now on the _Clusters_ screen, select _Create Server Group/Job_, choose _Server Group_ from the drop down and hit _Next_ to see the following dialog:



![image](https://files.readme.io/JRMxxbaSQ1mmD5VH8EtD_firstSG1.png)


Scroll down to the newly created _Container_ subsection, and edit the following fields:


![image](https://files.readme.io/qaBd9hZQZakHXVxNp57c_firstSG2.png)


Once the create task completes, open a terminal and type <code>$ kubectl proxy --port 7777</code>, and now navigate in your browser to http://localhost:7777/api/v1/proxy/namespaces/default/services/serve-dev:80/ to see if your application is serving traffic correctly.



Once you're satisfied, don't close the proxy or browser tab just yet as we'll use that again soon.

# 4. Git to _dev_ Pipeline

Now let's automate the process of creating server groups associated with the _dev_ loadbalancer. Navigate to the _Pipelines_ tab, select _Configure_ > _Create New..._ and then fill out the resulting dialog as follows:

![image](https://files.readme.io/g7b322SFRlO4fg48LS6k_createdevdeploy.png)

In the resulting page, select _Add Trigger_, and fill the form out as follows:



![image](https://files.readme.io/ZV0WoYPyTQSwLJ1CvysC_dockertrigger.png)


Now select _Add Stage_ just below _Configuration_, and fill out the form as follows:


![image](https://files.readme.io/Aq6satGqTCKuRG64LXnu_deploydev.png)


Next, in the _Server Groups_ box select _Add Server Group_, where you will use the already deployed server group as a template like so:


![image](https://files.readme.io/kUBKsh2RwOT2f2PMieVE_templateselection.png)


In the resulting dialog, we only need to make one change down in the _Container_ subsection. Select the image that will come from the Docker trigger as shown below:


![image](https://files.readme.io/myFzaIjTxuAqfemPebZQ_configuredynamic.png)

Lastly, we want to add a stage to destroy the previous server group in this _dev_ cluster. Select _Add Stage_, and fill out the form as follows:

![image](https://files.readme.io/nx0SldzSmuijKB4N8LDA_destroysg.png)

#5. Verification Pipeline

Back on the _Pipelines_ dialog, create a new pipeline as before, but call it "Manual Judgement". On the first screen, add a Pipeline trigger as shown below:

![image](https://files.readme.io/RY6xIdMlR1SXv2C4ED86_judgetrigger.png)

We will only add a single stage, which will serve to gate access to the _prod_ environment down the line. The configuration is shown here:

![image](https://files.readme.io/1helP7sSUKjmk1UtRJud_manjudge.png)

Keep in mind, more advanced types of verification can be done here, such as running a Kubernetes batch job to verify that your app is healthy, or calling out to an external Jenkins server. For the sake of simplicity we will keep this as "manual judgement".

# 6. Promote to _prod_

Create a new pipeline titled "Deploy to Prod", and configure a pipeline trigger as shown here:

![image](https://files.readme.io/X3INVNHjRzqdFETWuqgv_mantrigger.png)

Now we need to find the deployed image in _dev_ that we previously verified. Add a new stage and configure it as follows:

![image](https://files.readme.io/NftKIHtUQkCXfA3oQr9j_fimage.png)

Now, to deploy that resolved image, add a new stage and configure it as follows:

![image](https://files.readme.io/FsfZpwIfQ62bm7G5ctZY_proddeploy1.png)

Select _Add Server Group_, and again use the _dev_ deployment as a template:

![image](https://files.readme.io/C0VZMELhTAqqScv0XbaO_templateselection.png)

This time we need to make three changes to the template. First, change the "stack" to represent our _prod_ cluster:

![image](https://files.readme.io/B4zMXkqsSDy4cKRgW9Lc_changestack.png)

Next in the load balancers section:

![image](https://files.readme.io/vPByp3geSgGgogZkbptg_prodlbse.png)

Lastly in the container section:

![image](https://files.readme.io/ASBOqj8Tw2nQOwfyTcyQ_fimingres.png)

Now to prevent all prior versions of this app in production from serving traffic once the deploy finishes, we will add a "Disable Cluster" stage like so:

![image](https://files.readme.io/HMjKsHhTTqvL3oDdC5K6_disablecluster.png)

Save the pipeline, and we are ready to go!

# 7. Run the Pipeline

Push a new branch to your repo, and wait for the pipeline to run.

<code>NEW_VERSION=v1.0.0
 git checkout -b $NEW_VERSION
 git push origin $NEW_VERSION</code>

Once the Manual Judgement stage is hit, open http://localhost:7777/api/v1/proxy/namespaces/default/services/serve-dev:80/ to "verify" your deployment, and hit _continue_ once you are ready to promote to _prod_.

![image](https://files.readme.io/cv8j0EeQe2y4jqA6metn_manjudge2.png)

![image](https://files.readme.io/AA5gceLKTnCggsVY6cRx_trigfind.png)

![image](https://files.readme.io/5POkkeL5QvygBdPA1HGQ_linktocluster.png)



