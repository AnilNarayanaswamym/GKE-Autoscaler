What You’ll Learn
In this Lab you will learn how to:
Troubleshoot customer issues more effectively
Traverse logs
Mitigate problems
Lab 1: Cluster autoscaler
Objectives

Successfully troubleshoot the simulated workload issue:
Deploy the workload
Observe the described issue
Understand the problem
Remediate the problem
Task 1. Create cluster

In this section, you:
Create a GKE cluster in a specific network/subnetwork

Setting up your environment
In the browser window where you have logged into the Cloud Console, click this link to load a Cloud Shell

Alternatively, you can copy the URL below to open the Cloud Shell:
https://console.cloud.google.com/cloudshell/

Run this command to create a network and subnetwork
# create network
gcloud compute networks create gke-clusters-network --subnet-mode=custom
 
# create subnetwork
gcloud compute networks subnets create gke-subnetwork --network=gke-clusters-network --region=us-central1 --range=10.1.0.0/28

Run this command to create a GKE cluster where all VMs will reside in the network/subnetwork you just created.
# create cluster
gcloud beta container clusters create "standard-cluster-1" --zone "us-central1-a" \
--no-enable-basic-auth \
--cluster-version "1.13.11-gke.14" \
--machine-type "n1-standard-1" \
--image-type "COS" \
--disk-type "pd-standard" \
--disk-size "100" \
--num-nodes "3" \
--enable-cloud-logging \
--enable-cloud-monitoring \
--enable-ip-alias \
--network=gke-clusters-network \
--subnetwork=gke-subnetwork \
--default-max-pods-per-node "30" \
--addons HorizontalPodAutoscaling,HttpLoadBalancing \
--enable-autorepair \
--scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append"
Create additional node pools
# create additional node pools
gcloud beta container node-pools create node-pool-1 --node-version=1.13.11-gke.14 \
--num-nodes=3 \
--machine-type=n1-standard-1 \
--disk-size=100 \
--disk-type=pd-standard \
--node-version=1.13.11-gke.14 \
--scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append \
 --image-type=COS \
 --max-pods-per-node=30 \
 --enable-autorepair \
 --enable-autoupgrade \
 --cluster=standard-cluster-1 \
 --zone=us-central1-a \
 --enable-autoscaling \
 --max-nodes=15 \
 --min-nodes=3 \

# delete default node pool
gcloud container node-pools delete default-pool --cluster=standard-cluster-1 --zone us-central1-a


Task 2. Deploy problematic workload

Deploy both YAML in the cluster you just created

	workload
$ kubectl apply -f https://storage.googleapis.com/breakfix/myapp.yaml

Task 3. Identify issue and resolve it

When you deploy the YAML file, the cluster autoscaler will create new nodes automatically for you. If you modify the YAML for 30 replicas, you will get unschedulable pods and the cluster autoscaler is unable to scale properly. Why?
