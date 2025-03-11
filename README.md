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
In the event of a disaster (or a mock up), considering the 'supposed to be the live cluster' is down, we change the replicas to 1 for the inactive cluster and merge the PR. Argo CD picks up the changes and deploys the changes and the Route 53 directs the traffic to the running cluster. `make sure you replicate this Route 53 routing`

## Can you talk about scaling in kubernetes with some examples
Kubernetes can resize clusters depending the current load. If we are hosting a microservice in a Pod, them if that pod, occasionally, has to work too hard, especially during the month end where due to trade expiries, traders place more transactions into the system. That mocroservice running in a pod are automatically replicated. For to be able to replicate itself, the microservice has to be written in such a way. In one of our examples, we have a web frontend and a API gateway, which is the single point of entry from the frontend. Trade orders and deals are sent my trade capture microservice to activeMQ and the trade status microservice sends and reads the trade data from the backend database. The frontend routes the read requests to the trade status microservice to get the current status of the every deal. SO we cannot replicate the trade capture the m/s as that will push duplicate data to the queue. There are some stateless m/s like service which reads data from the DB.(mongo replication is possible with statefulsets).API gateway is stateless so they are great candidate for replication. We set HPA, which is a kubernetes controller which keeps monitoring the resource utilization say CPU utilization and if it is more than the threshold, it modifies the deployment and changes the replica counts so that a new pod is spawned. HPA monitors the resource util via the metric server. The threshold depends on the request we have set on the pod. We set up HPA as individual rules for each of the deployments.
![image](https://github.com/user-attachments/assets/ad28ffe0-0d9f-4356-abc0-d867d3b2c001)
![image](https://github.com/user-attachments/assets/f8fccc14-bb90-41ef-9259-180d06bcd6e0)
![image](https://github.com/user-attachments/assets/0bf65b5f-2f45-4fda-966d-a5fb28f02b5e)
![image](https://github.com/user-attachments/assets/45b7405d-a00f-4e90-86e3-dad024be3f76)
We need to create separate HPA for each one of the deployments.
![image](https://github.com/user-attachments/assets/721e8d6e-ee68-43ec-99fb-f40ad1c9626a)
![image](https://github.com/user-attachments/assets/0900def0-9315-4fac-b47a-31f013388046)

## How is readiness probe linked to HPA
When a new pod comes up, the program inside the pod may take sometime to come up but as soon as the pod comes into exixtence, the service thinks that the pod is ready and it starts diverting the traffic to the pod. In this case we will get 503 error. This is why we set readiness probe in the pod or deployment definition.
![image](https://github.com/user-attachments/assets/fbb535da-4f27-44ef-99af-eed074aecfb9)
![image](https://github.com/user-attachments/assets/a22c450b-f95d-4816-97e7-49b6862d2fd4)
![image](https://github.com/user-attachments/assets/180f2e4e-218c-4bf5-8bf7-251189af66d6)
![image](https://github.com/user-attachments/assets/e3876ce6-43d3-4a0e-9b76-2e0279f20d4f)
![image](https://github.com/user-attachments/assets/b2a24880-e5ec-4168-a950-b6dc692b66de)
Liveness probe runs for the entire duration of the pod and if the probe fails, the container inside the pod is restarted.
![image](https://github.com/user-attachments/assets/c13fe073-2ade-47e7-a20c-79030f493a3f)
If the pod is not a http webserver, we can set a command to check the readiless -
![image](https://github.com/user-attachments/assets/ac0eb97e-3404-42f6-bc34-3294ec27b873)
We can also set up TCP probes.

## What is the role of the Scheduler
Scheduler runs in the master node and its purpose is to look for Nodes suitable for hosting new pods and then schedule it. Some times the Pods also gets evicted due to the QoS and then the scheduler reschedules the Pod in another Node.
![image](https://github.com/user-attachments/assets/8f3c0f22-d270-471b-8522-f97bd9b25ee9)
![image](https://github.com/user-attachments/assets/d4519b56-8093-4112-bbf7-ba000b66b0e9)
Eviction algo - BestEffort, Burstable, Guaranteed
Pod priorities are for the new pods to be scheduled.
![image](https://github.com/user-attachments/assets/34d5495e-c150-46ee-a64c-e5c048ad21dd)
![image](https://github.com/user-attachments/assets/4b493960-8fde-4437-a079-e3021355352f)

## How have you implemented cluster autoscaller
Depending on the Nodes which we choose, say m5.large, it has 4 vCPUs. If we have set a request of 500m for the pod specification in the deployment and if the replicas are set to 20 then 20*500=10,000m=10 vCPUs

![image](https://github.com/user-attachments/assets/d809cdfe-3433-49eb-a101-311a9947f787)

Even though, in the above example, there are no traffic, the ClusterAS will kick in and will spin up 4 more m5.large.
Under the hood the Nodes are ec2s. ec2s auto scales with the help og auto scaling groups. So the ClusterAS tells the ASG to provision more Nodes.

`eksctl create cluster --name my-cluster --version 1.15 --managed --asg-access (by default it creates 2 m5.large)
Autoscaler version and kubernetes version must match.
If the ASG is limited to scale no. of Nodes, then ClusterAS cannot scale. So, we need to edit the ASG to set the right min and max limits.
ClusterAS runs as a deployment(pod) in the kube-system namespace

### Steps:
- EKS cluster running
- ASG in the Nodes
- IAM permissions for ASG
- create SA with IRSA for the ClusterAS
- YAML
```apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    app: cluster-autoscaler
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
      annotations:
        cluster-autoscaler.kubernetes.io/safe-to-evict: "false"
    spec:
      serviceAccountName: cluster-autoscaler
      containers:
        - name: cluster-autoscaler
          image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.29.0  # Match your k8s version
          resources:
            limits:
              cpu: 100m
              memory: 300Mi
            requests:
              cpu: 100m
              memory: 300Mi
          command:
            - ./cluster-autoscaler
            - --cloud-provider=aws
            - --cluster-name=<cluster-name>
            - --scan-interval=10s
            - --balance-similar-node-groups
            - --skip-nodes-with-system-pods=false
            - --expander=least-waste
            - --logtostderr=true
            - --v=4
          env:
            - name: AWS_REGION
              value: <region>
  ```


## ESO

![image](https://github.com/user-attachments/assets/fb1abfe3-922c-478e-969a-84c3320a9040)


![image](https://github.com/user-attachments/assets/63297504-61b8-4c8e-a42b-1e22c185f442)

![image](https://github.com/user-attachments/assets/766b9659-475b-4701-aecb-1aeadfba5ed5)

![image](https://github.com/user-attachments/assets/402c2f30-e7a9-459b-9c23-c343dcd6e02a)















