# Notes, Thoughts, and Questions

## Initial thoughts

### Necessary components:
- CDN
- WAF
- Geo-routing
- S3-equivalent storage for media content (with replication)
- EFS-equivalent storage for WP file system (with backups)
- RDS-equivalent MySQL backend (with snapshots)
- Multi-zone VPC x2 (1 for SF, 1 for Tokyo)
  - Public subnets for externally facing resources
  - Private subnets for backend and admin resources
- Compute resources for running WP
  - Compute instances in ASGs or
  - Regional GKE cluster (overkill?)

## After some investigation

### Additional component notes:
- CDN, WAF, and geo-routing seem to all fall under the external LB component
- This project can utilize a single, multi-region VPC
- While a fun solution, GKE likely adds too much complexity to the project

## Questions:
- Is there any overwhelming benefit to implementing a GKE-based solution?
- Is there a need for a non-prod system?
  - How would updates be propagated to prod systems?

## Scaling notes:
- Spikes on 1st and 15th bring in 1,000,000 users over 24hrs
  - 1,000,000/24 = 41,667 users per hour
  - ~42x normal hourly traffic
- Need clarification on what "adding 10,000 active users per month" means
  - 10,000/30 days/24 hours = ~14 additional users per hour (not significant)

## Compute Instance Notes:
- Mount Filestore as RO volume in each node of instance groups; RW for admin node
- Nodes will have IAM role that has read-only access to Cloud Storage

## ~~K8s Notes:~~
- ~~Internet-exposed pods should~~
  - ~~use persistent storage that is ROX~~
  - ~~connect to read-only DBs~~
  - ~~have read-only access to Cloud Storage~~

## Things to monitor, tune, consider:
- CDN
  - Expiration time for cached content; increase or decrease?
- Cloud Armor
  - Where is malicious traffic originating?
  - Are custom rules necessary?
- Cloud SQL
  - Is one read replica in each region sufficient?
- Cloud Storage
  - What is the latency for compute resources in asia-northeast1
  pulling media from the us-west2 bucket?
- Compute
  - What instance size should be used?
  - How big does the instance group need to be to meet initial traffic demand?
  - For normal traffic and monthly increases, what load-balancing threshold should cause scale-ups?
  - What scheduling mechanisms are available to scale up for 1st and 15th spikes?
