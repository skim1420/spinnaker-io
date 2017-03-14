The Spinnaker Openstack driver is developed and tested against the Openstack Mitaka release.
Due to the limitless ways to configure Openstack and its services, here is a list of API versions that are required to be enabled:

 * Keystone (Identity) v3
 * Compute v2
 * LBaaS v2
 * Networking v2
 * Orchestration (Heat)
 * Ceilometer
 * Glance v2

#### Clouddriver Configuration

**Basic Configuration**

To configure Spinnaker to manage Openstack, pick your favorite editor and open <code>/opt/spinnaker/config/spinnaker-local.yml</code>. Under the <code>provider</code> section, you'll find the <code>openstack</code> subsection, which we've annotated with comments:

````yml
  openstack:
    enabled:            # set to true to deploy to Openstack
    primaryCredentials:
      name:             # used to identify this set of credentials later
      authUrl:          # URL of the Keystone v3 API e.g. https://openstack.example.com:5000/v3
      username:         # username to connect to Openstack
      password:         # password for the user
      projectName:      # name of the project to manage
      domainName:       # name of the domain to manage
      regions:          # comma separated list of regions manage, may be a subset of all available regions
      insecure:         # skip SSL verification when connecting to Openstack, defaults to false
````

The example `spinnaker-local.yml` file `default-spinnaker-local.yml` will read all these options from environment variables if that is how you prefer to configure your environment.

The `insecure` option is needed if the Openstack server is self-signed certificate, such as a Devstack setup. It is not recommended to set `insecure` to `true` in production.

Once you made the necessary changes, restart Clouddriver with <code>sudo restart clouddriver</code>.

**Advanced Configuration**

If managing load balancers in your Openstack environment through Spinnaker is failing due to timeouts, you may need to increase the timeout and polling interval. This is more likely to occur in a resource constrained Openstack environment such as Devstack or another smaller test environment. Openstack requires that the load balancer be in an ACTIVE state for it to create associated  relationships (i.e. listeners, pools, monitors). Each modification will cause the load balancer to go into a PENDING state and back to ACTIVE once the change has been made. Spinnaker needs to poll Openstack, blocking further load balancer operations until the status returns to ACTIVE.

To do this, update the <code>openstack.accounts</code> block in <code>/opt/spinnaker/config/clouddriver-local.yml</code>

````yml
  openstack:
    #...
    accounts:
      - name:
        authUrl:
        #...
        lbaas:
          pollTimeout: 60   # timeout in seconds for waiting for a load balancer status to return to ACTIVE
          pollInterval: 5   # interval in seconds in which to poll Openstack for the status of a load balancer
````
To configure Spinnaker to deploy to multiple Openstack accounts or projects, append to the <code>openstack.accounts</code> block in <code>/opt/spinnaker/config/clouddriver-local.yml</code>

````yml
  openstack:
    #...
    accounts:
      - name:         # name of Spinnaker account for Openstack account or project 1
        authUrl:      # URL of the Keystone v3 API for Openstack account or project 1
        username:     # username to connect to Openstack account or project 1
        password:     # password for the user for Openstack account or project 1
        projectName:  # name of the project to manage for Openstack account or project 1
        domainName:   # name of the domain to manage for Openstack account or project 1
        regions:      # comma separated list of regions manage for Openstack account or project 1
        lbaas:        # load balancer configuration for Openstack account or project 1
          pollTimeout:
          pollInterval:
      - name:         # name of Spinnaker account for Openstack account or project 2
        authUrl:      # URL of the Keystone v3 API for Openstack account or project 2
        username:     # username to connect to Openstack account or project 2
        password:     # password for the user for Openstack account or project 2
        projectName:  # name of the project to manage for Openstack account or project 2
        domainName:   # name of the domain to manage for Openstack account or project 2
        regions:      # comma separated list of regions manage for Openstack account or project 2
        lbaas:        # load balancer configuration for Openstack account or project 2
          pollTimeout:
          pollInterval:
````

#### Rosco Configuration

To configure Spinnaker to bake Openstack images, pick your favorite text editor and create or open <code>/opt/spinnaker/rosco-local.yml</code>. Add or update the <code>openstack</code> section.

````yml
  openstack:
    enabled:            # enables ability to bake Openstack images
    bakeryDefaults:
      username:         # username to authenticate to Openstack with
      password:         # uassword for the user
      authUrl:          # URL to the Openstack identity service
      domainName:       # domain name you are authenticating with
      networkId:        # network UUID to attach to instance while baking the image
      floatingIpPool:   # Name of the floating IP pool to use to allocate a floating IP
      securityGroups:   # List of security groups by name to add to the instance while baking the image
      projectName:      # project name you are authenticating with
      insecure:         # skip SSL verification when connecting to Openstack, defaults to false
      templateFile:     # packer template file, defaults to openstack.json
      baseImages:       # list of base images that can be used to bake an image
      - baseImage:
          id: vivid             
          shortDescription: 15.04
          detailedDescription: Ubuntu Vivid Vervet v15.04
          packageType: deb      # type of packages to be installed, either deb or rpm
        virtualizationSettings: # list of different regions with virtualization settings for the base image
        - region:               # A region to bake the image in
          instanceType:         # The instance type to use for baking the image e.g. smem-2vcpu, mmem-2vcpu
          sourceImageId:        # a UUID of the source image to use when baking the image
          sshUserName: ubuntu   # username for sshing into the instance for baking the image
        - region:               # Another region or/and virtualization settings to choose
          instanceType:         
          sourceImageId:
          sshUserName:
````

