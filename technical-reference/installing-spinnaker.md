# Overview

The platform that Spinnaker is installed on is independent of which platforms that instance of Spinnaker can deploy to. For example, you can run your Spinnaker instance on Google Cloud Platform and from there deploy to Amazon Web Services or a Kubernetes cluster.

Spinnaker is composed of several microservices that provide each piece of the functionality of the system.

**Component Name** | **Functionality** | **Default Port**
---|---|---
Deck | User interface | 9000
Gate | Api gateway. All external requests to Spinnaker are directed through Gate. | 8084
Orca | Orchestration of pipelines and ad hoc operations. | 8083
Clouddriver | Interacts with and mutates infrastructure on underlying cloud providers. | 7002
Rosco | Machine image bakery. A machine image is a static view of the state and disk of a machine that can be 'deployed' into a running instance. Representation varies by cloud provider. | 8087
Front50 | Interface to persistent storage, such as Amazon S3 or Google Cloud Storage. | 8080
Igor | Interface to Jenkins. Can both listen to and fire Jenkins jobs and collect contextual job and build information. | 8088
Echo | Event bus for notifications and triggers. Triggers are things like git commits, Jenkins jobs finishing and other Spinnaker pipelines finishing. Notifications can send emails, slack notifications, SMS messages, etc. | 8089

