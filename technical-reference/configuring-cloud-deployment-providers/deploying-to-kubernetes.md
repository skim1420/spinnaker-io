First, follow the [Kubernetes getting started](http://kubernetes.io/docs/getting-started-guides/) for setting up a cluster. For ease of setup, it's recommended to use one of the hosted solutions. 

Once your cluster is running, you need to get its authentication details in your local [kubeconfig file](http://kubernetes.io/docs/user-guide/kubeconfig-file/). Most hosted providers will generate this file for you as a part of the setup process, and place it in <code>~/.kube/config</code>. You can verify that these credentials are working by running <code>kubectl get namespaces</code>.

Since Kubernetes deploys containers built outside of Spinnaker, it needs to know where to find the images you want to deploy. Therefore, we need to enable the Docker Registry provider, which is explained below, and in much more detail [here](doc:target-deployment-configuration:#section-docker-registry).

**Basic Configuration**

If you're running Spinnaker inside a Kubernetes cluster configured [here](doc:kubernetes), it is already configured to manage Kubernetes. If you are running Spinnaker on a VM, keep reading.

First make sure Spinnaker is up-to-date, as many of these features are only available in more recent builds (late Q1 2016):

    $ sudo apt-get update
    $ sudu apt-get upgrade spinnaker*
    
Now, make sure that your kubeconfig file, described in the [Kubernetes setup](doc:target-deployment-setup#section-kubernetes-cluster-setup), is placed at <code>/home/spinnaker/.kube/config</code>. Next, pick your favorite editor and open <code>/opt/spinnaker/config/spinnaker-local.yml</code>. Under the <code>providers</code> section, you'll find the <code>kubernetes</code> and <code>dockerRegistry</code> subsections, which we've annotated with comments:

    kubernetes:
      enabled:                  # set to true to deploy to kubernetes
      primaryCredentials:
        name:                   # used to identify this set of credentials later
        dockerRegistryAccount:  # must match the name field of the docker registry credentials below

    dockerRegistry:
      enabled:                  # set to true to deploy to kubernetes
      primaryCredentials:
        name:                   # used to identify this set of credentials later
        address:                # baseUrl of the registry api
        repository:             # library name of the repository you are deploying
    
You'll need to enable both providers, and set sensible values for the other fields (which are likely already filled). Finally, restart Clouddriver with <code>sudo restart clouddriver</code>.

**Advanced Configuration**

There is much to configure with the Kubernetes provider, if you so choose. This is done inside your <code>/opt/spinnaker/config/clouddriver-local.yml</code> file, where you can set fields under the <code>providers</code> section as follows:


    kubernetes:
      enabled:               # boolean indicating whether or not to use kubernetes as a provider
      accounts:              # list of kubernetes accounts
        - name:              # required unique name for this account
          kubeconfigFile:    # optional location of the kube config file
          namespaces:        # optional list of namespaces to manage
          context:           # optional context in kubeconfig to use as a user/cluster pair
          dockerRegistries:  # required (at least 1) docker registry accounts used as a source of images
            - accountName:   # required name of the docker registry account
              namespaces:    # optional list of namespaces this docker registry can deploy to

By default, the kubeconfig file at <code>~/.kube/config</code> is used, unless the field <code>kubeconfigFile</code> is specified. The context field is derived from the <code>current-context</code> field in the kubeconfig file, unless it is explicitly provided. If no namespace is found then all existing namespaces will be used, and periodically refreshed. Any referenced namespaces that do not exist will be created. Each Spinnaker Kubernetes account is linked to a specific Kubernetes context. If you want to deploy to multiple Kubernetes clusters you'll need to explicitly list which context each account is linked to:

    kubernetes:
      enabled: true
      accounts:
        - name: dev-cluster
          context: dev-context # found in ~/.kube/config
          dockerRegistries:
            - accountName: my-docker-account # configured under dockerRegistry section
        - name: test-cluster
          context: test-context # found in ~/.kube/config
          dockerRegistries:
            - accountName: my-docker-account # configured under dockerRegistry section

The Docker Registry accounts referred to by the above configuration are also configured inside Clouddriver. The details of that implementation can be found [here](https://github.com/spinnaker/spinnaker/wiki/Docker-Registry-Implementation). The Docker authentication details (username, password, email, endpoint address), are read from each listed Docker Registry account, and configured as an [image pull secret](http://kubernetes.io/v1.1/docs/user-guide/images.html#specifying-imagepullsecrets-on-a-pod). The <code>namespaces</code> field of the <code>dockerRegistry</code> subblock defaults to the full list of namespaces, and is used by the Kubernetes provider to determine which namespaces to register the image pull secrets with. Every created pod is given the full list of image pull secrets available to its containing namespace.

The Kubernetes provider will periodically (every 30 seconds) attempt to fetch every provided namespace to see if the cluster is still reachable.

