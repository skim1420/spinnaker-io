To run Spinnaker on an on-prem or localhost environment, type in the following command:

    bash <(curl --silent https://spinnaker.bintray.com/scripts/InstallSpinnaker.sh)

The above [script](https://github.com/spinnaker/spinnaker/blob/master/InstallSpinnaker.sh) installs and configures Spinnaker, and starts all Spinnaker components, including Redis and Cassandra, which Spinnaker components use to store data. If you see any errors, please just run the command again.

It will take several minutes to install and configure Spinnaker along with all of its dependencies. Once the install is complete, you will use your web browser to interact with Spinnaker.

