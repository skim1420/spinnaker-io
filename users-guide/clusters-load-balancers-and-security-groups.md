# Server Groups

A Server Group is a group of compute resource instances (VMs, container pods). The instances of a Server Group are identical and are managed as a group by the cloud platforms (e.g. ASGs, Kubernetes Replica Sets).

Server Groups mandatorily take on the naming convention *application-stack-detail-version*, where *stack* and *detail* are optional and *version* is system-assigned (incrementing version number). Spinnaker has no particular opinion on how the *stack* and *detail* fields are used, however it's common to use *stack* to represent a particular environment for a given microservice (e.g. dev, test, prod) and to use *detail* as an additional descriptor (e.g. feature branch, "experimental"). Finally, *version* indicates the specific Server Group, and individual instances of a Server Group are given a hash suffix in their names (e.g. "c9nzt") to uniquely identify each.

# Clusters

Clusters are logical groupings of Server Groups, by *application-stack-detail*. Clusters are used in situations where multiple Server Groups for a given microservice need to be coordinated in a single context, for example, during a red/black deployment.

# Load Balancers

Load Balancers route incoming traffic to associated and enabled Server Groups.

# Security Groups

Security Groups define traffic ingress rules for Server Groups.

# Resource Mappings from Spinnaker to Provider

Spinnaker | AWS | GCE | Kubernetes | Azure | OpenStack | Cloud Foundry
---|---|---|---|---|---|---
Server Group | ASG | MIG | Replica Set
Load Balancer | Load Balancer | L3/L4/HTTP/internal Load Balancer | Service
Security Group | Security Group | Firewall Rule | Ingress Rule
