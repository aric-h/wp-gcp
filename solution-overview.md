# Overview

## Edge Layer

#### [External Load Balancing](https://cloud.google.com/load-balancing/docs/https)
Per the [documentation](https://cloud.google.com/load-balancing/docs/https#cross-region_load_balancing) - 
and assuming that this project is utilizing the GCP premium network tier - an external HTTPS load balancer will 
intelligently route traffic to the geographically closest backend resources.
This will enhance response times for the geographically-dispersed users of the WordPress site.

#### [Cloud CDN](https://cloud.google.com/cdn/docs/overview)
Configured on the external load balancer, Cloud CDN will further enhance the user experience by caching content
at the edge of Google's network. When a user makes a request for cached content, it is returned immediately,
without the backend having to process and serve the request.

#### [Cloud Armor](https://cloud.google.com/armor)
Also configured at the edge, Cloud Armor provides DDoS protection, as well as a number of other predefined rules
to protect against common cyber attacks.

## Compute Layer

#### [Compute Instances](https://cloud.google.com/compute/docs/instances)
The WordPress site will be deployed to autoscaling, self-healing [instance groups](https://cloud.google.com/compute/docs/instance-groups).
There will be a group deployed to us-west2 and a group deployed to asia-northeast1, to position resources close to the primary userbases.
The instance groups will [autoscale](https://cloud.google.com/compute/docs/autoscaler) based on HTTPS load,
ensuring that enough instances are in service to handle user traffic. To handle massive, expected traffic spikes, scheduled scaling
activities will drastically increase the number of instances in service to avoid a scenario where autoscaling is not able to keep up with demand.
The instance groups will be configured to have read-only access to the various components of the data layer.

An admin compute instance will also be deployed in us-west2, that will have write access to the various data
layers of this project. This will enable admins to manage the site from a node that is not serving user traffic.

## Data Layer

#### [Cloud SQL](https://cloud.google.com/sql)
For the WordPress backend needs, the MySQL database will be deployed on Cloud SQL. For redundancy, The Cloud SQL 
instance in the primary region (us-west2) will be deployed in an [HA configuration](https://cloud.google.com/sql/docs/mysql/high-availability).
The instance will also have one or more [read replicas](https://cloud.google.com/sql/docs/mysql/replication) deployed 
in us-west2 and one or more cross-region read replicas deployed in asia-northeast1. 
The replicas will serve to enhance performance of the site, as well as prevent malicious activity from performing any writes to the database.

#### [Cloud Storage](https://cloud.google.com/storage#section-7)
To offload media content storage, WordPress will be configured with a [plugin](https://wordpress.org/plugins/gcs/)
that stores all media files in a Cloud Storage bucket. As this will be deployed in the primary region (us-west2),

#### [Cloud Filestore](https://cloud.google.com/filestore)
To provide persistent, highly-available storage of the WordPress configuration, a Cloud Filestore instance will be
mounted to each compute instance running the site. User-facing compute resources will have read-only access to this
file system. The primary Filestore instance will reside in us-west2 with a TBD mechanism replicating data to a 
secondary instance in asia-northeast1.

## Network Layer

#### [VPC](https://cloud.google.com/vpc/docs/overview)
*just about out of time. doh!*

The VPC will be deployed as a multi-region VPC. An array of private subnets (one for each zone in each region) will be provisioned
for the compute instance groups. A similar array of private subnets will also be provisioned for the Cloud SQL instance(s).


