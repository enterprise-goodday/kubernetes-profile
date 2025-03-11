# All about kubernetes job and descriptions of projects

## Begin introduction
I have transitioned to cloud and kubernetes a bit late in my career. I have 18 years of experience and I strted working with cloud and kubernetes in the year 2019/20.
Before that I was working on support projects leading huge team for supporting 50+ application and I was also SME for Oracle based application. I also did data migration using PL/SQL.
The firt Kubernetes project I got into was an SRE role where we were responsible for the end to end reliability, uptime and support of the applications hosted on kubernetes.
Part of my responsibilities were monitor kubernetes cluster, toubleshoot for any production issues, coordinate with Dev and Ops teams for issue resolutions, organizing issue post-mortems, finding RCA, document findings and learninga and maintain playbooks.
This continued until I got a chance to play the role of a release and deployment manager. The role was in the US and I had to cover for the US business and also I took responsibilities of release planning and deployments.
Deployments where done just before the closing hours of the trading business of US and other businesses from rest of the world would have been alredy shut.
After this I trasitioned to a full fledged kubernetes developement role because the client had many requirements of migrating many applications hosted in VM.
These includes application written in Nodejs, React, Springboot and .Net. I was actively involved in helping the dev teams containerize their applications and then hosting them in Kubernetes in Dev, Test and Prod.
Some of the sub modules I developed are creating PostgreSQL, Redis and Kafka servers in Kubernetes and set HA and Scaling. Also set up failover and DR for applications.
I also worked closely withe the DevOps/GitOps team and wrote Helm charts for deployments.

## Question on DR
### How do you set the DR environemnt and what strategy you follow
We did not check for the usual book of the RTO/RPO as the applications are in their early stage but we just had to set the DRs as for the compliance we have to do a yearly DR test.
In one large application, we had 2 clusters - one is east-us and other in north-eu. We labelled them as BLUE and GREEN.
Only 1 cluster is live and running at a given point of time. So, this is a warm standby mode where the inactive cluster is ready except it has replicas set to 0 so no workloads are running there.
We have 1 instance of SQL Server running, with a global replica. So the Pods from both servers point to the same SQL DB.
In the event of a disaster (or a mock up), considering the 'supposed to be the live cluster' is down, we change the replicas to 1 for the inactive cluster and merge the PR. Argo CD picks up the changes and deploys the changes and the Route 53 directs the traffic to the running cluster
