#Kubernetes Installation pre-requisites:

1. Kops 						: The utility used to install the kubernetes cluster
2. Kubectl 						: The kubernetes command line tool used to manage the cluster
3. Domain configured at Route53 : All the URL entries are created under this zone id which are also updated dynamically by the cluster.
4. S3 Bucket				    : Used to store the Cluster configuration that is used by kops.
5. AWS Cli						

Procedure:
1. Create a S3 bucket.

2. AWS Cli Installation:
apt-get install -y awscli;
aws configure set aws_access_key_id AKIAJG4VCVUPCQLZSQ;
aws configure set aws_secret_access_key Wzd/nWCym7B6pkNYR4;
aws configure set default.region ap-south-1;


3. Kops Installation:
#Reference URL: https://github.com/kubernetes/kops

wget https://github.com/kubernetes/kops/releases/download/1.8.0/kops-linux-amd64
chmod +x kops-linux-amd64
mv kops-linux-amd64 /usr/local/bin/kops

4. Kubectl Installation:
#Reference URL: https://kubernetes.io/docs/tasks/tools/install-kubectl/

curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl;
chmod +x ./kubectl;
mv ./kubectl /usr/local/bin/kubectl;

5. Kops uses an id_rsa.pub file to be used for the created instances which i have used as below:
cp /home/ubuntu/.ssh/authorized_keys /root/.ssh/id_rsa.pub

6. Export the variables for the Route53 hosted zone and the S3 bucket as below:
export KOPS_STATE_STORE=s3://rvkubernetes.tk    #S3 bucket that we created in the step 1
export DNS_ZONE_PUBLIC_ID=Z1A3BXE5N40766 		#Zone ID of the Route53 hosted zone

7. Create the cluster configuration as below:

kops create cluster --zones=ap-south-1a rvkubernetes.tk

8. Cluster configuration has been created and stored in the s3 bucket. To edit the node/master instance type and the auto-scaling configuration, we need to edit the configurations with below commands

 * edit your node instance group with command: kops edit ig --name=rvkubernetes.tk nodes
 * edit your master instance group with command: kops edit ig --name=rvkubernetes.tk master-ap-south-1a
 
9. After making the desired changes....apply the configuration to create the cluster with below command:
 kops update cluster rvkubernetes.tk --yes
 
10. It will take 7-10 minutes to build the cluster post which we can use the below commands to verify the cluster status:
  * validate cluster with command: kops validate cluster 
  Or Use: kubectl get nodes

  #### Notes:-
 * The above procedure creates a separate VPC and builds the entire cluster inside that VPC. 
 * Two IAM roles are created, one for master and one for node.
 * Two additional volumes are created and attached to the k8 master for storing the data to be used for disaster recovery. Whenever the master instances 	  changes  or gets terminated, these two volumes get attached with the new master instance automatically. In a HA k8 cluster, each master nodes in the different AZs have their own additional volumes.
  
  


----------------------------
#Dashboard Installation:
----------------------------
#Reference URL: https://github.com/kubernetes/kops/blob/master/docs/addons.md

kubectl create -f https://raw.githubusercontent.com/kubernetes/kops/master/addons/kubernetes-dashboard/v1.7.1.yaml

The login credentials are:

    Username: admin
    Password: get by running "kops get secrets kube --type secret -oplaintext" or "kubectl config view --minify"
	
The Dashboard URL pattern is like below:

https://api.$domain/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy/#!/namespace?namespace=_all

As In our case:

https://api.rvkubernetes.tk/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy/#!/namespace?namespace=_all

