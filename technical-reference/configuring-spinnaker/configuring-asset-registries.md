# Docker Registry


**Basic Configuration**

Pick your favorite editor and open <code>/opt/spinnaker/config/spinnaker-local.yml</code>. Under the <code>providers</code> section, you'll find the <code>dockerRegistry</code> subsection, which we've annotated with comments:

    dockerRegistry:
      enabled:                # set to true to deploy to kubernetes
      primaryCredentials:
        name:                 # used to identify this set of credentials later
        address:              # baseUrl of the registry api
        repository:           # library name of the repository you are deploying
    
The address field depends on your choice of provider:

DockerHub: https://index.docker.io
Google Container Registry: https://gcr.io, https://us.gcr.io, https://eu.gcr.io, https://asia.gcr.io
Quay: https://quay.io

**Advanced Configuration**

There is much to configure with the Docker registry provider, if you so choose. This is done inside your <code>/opt/spinnaker/config/clouddriver-local.yml</code> file, where you can set fields at the top level as follows:

    dockerRegistry:
      enabled:          # boolean indicating whether or not to use docker registries as a provider
      accounts:         # list of docker registry accounts
        - name:         # required unique name for this account
          address:      # required address of the registry. e.g. https://index.docker.io
          username:     # optional username for authenticating with the registry
          password:     # optional password for authenticating with the registry
          passwordFile: # optional fully-qualified path to a file containing the registry password
          email:        # optional email for authenticating with the registry
          repositories: # optional list of registries. if none configured, registry must support `/_catalog` endpoint
          cacheThreads: # optional (default is 1) number of threads to cache registry contents across
          clientTimeoutMillis: # optional (default is 1 minute) time before the registry connection times out
          paginateSize: # optional (default is 100) number of entries to request from /_catalog at a time

Example:

    dockerRegistry:
      enabled: true
      accounts:
        - name: my-docker-registry-account
          address: https://index.docker.io
          username: myusernamehere
          password: mypasswordhere
          email: myuseramehere@domain.com
          repositories:
             - library/nginx
     
Clouddriver authenticates itself following the [official v2 Docker registry specification](https://docs.docker.com/registry/spec/auth/token/). The motivation for storing the credentials here, rather than loading them from <code>~/.docker/config</code>, was to allow the user to segregate accounts with access to the same Registry, but with different sets of permissions.

Additionally, in order for Docker Registry changes to be utilized as a trigger in a Spinnaker pipeline, ensure the following are available inside your <code>/opt/spinnaker/config/igor-local.yml</code> file:

    dockerRegistry:
      enabled: true
    

# Aptly

# jFrog Bintray

