Spinnaker's top level groupings are Projects, Applications and Infrastructure.

# Applications

Applications are a logical grouping of infrastructure components - Server Groups and Clusters, Load Balancers, Security Groups and Pipelines. All components under a given Application share infrastructure credentials and top-level permissions. There are a few areas where cross-Application activities occur (e.g. a Stage that starts a Pipeline in another application), however in most areas, references are relegated to within the Application.

An Application typically corresponds to a particular microservice.

# Projects

Projects are arbitrary collections of infrastructure resources and pipelines. This is a convenience feature which allows you to create a dashboard containing Applications, Clusters and Pipelines by wildcard.

# Infrastructure

This is a catch net area from which potentially lists all infrastructure components across all of Spinnaker that you have access to. You can search for Applications, Clusters, Server Groups, Instances, Load Balancers and Security Groups by wildcard.
