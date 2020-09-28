# kubernetes_my_demo
# Kubernetes on AWS using Kops

Set Environment variables:
	export S3_BUCKET_NAME=maitrihome-state-store	(Bucket Name)


### 1. Launch Linux EC2 instance in AWS (Kubernetes Client)
	- Amazon Linux
	- Connect to default security group.
	- Connect to security group (SSH Port 22) for putty to connect
### 2. Create a IAM User and attach below mentioned 5 policies:
	Kops need permissions to access:
		AmazonEC2FullAccess
		AmazonRoute53FullAccess
		AmazonS3FullAccess
		IAMFullAccess
		AmazonVPCFullAccess
		
	Connect to EC2 on putty-	
	Configure aws-cli with AWS_ACCESS_KEY and AWS_SECRET_ACCESS_KEY and REGION of the user created in step above:

	aws configure
		
		
### 3. Install Kops on EC2

curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64

chmod +x kops-linux-amd64

sudo mv kops-linux-amd64 /usr/local/bin/kops


### 4. Install kubectl

curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

### 5. Create S3 bucket in AWS and do versioning
S3 bucket is used by kubernetes to persist cluster state, lets create s3 bucket using aws cli
**Note:**  Make sure you choose bucket name that is uniqe accross all aws accounts

	aws s3api create-bucket --bucket ${S3_BUCKET_NAME} --region ap-south-1 --create-bucket-configuration LocationConstraint=ap-south-1
	
	aws s3api put-bucket-versioning --bucket maitrihome-state-store --versioning-configuration Status=Enabled

### 6. Create private hosted zone in AWS Route53
 1. Head over to aws Route53 and create hostedzone
 2. Choose name for example (maitrihome.in.in)
 3. Choose type as privated hosted zone for VPC
 4. Select default vpc in the region you are setting up your cluster
 5. Hit create

### 7 Configure environment variables.
Open .bashrc file 
```
	vi ~/.bashrc
```
Add following content into .bashrc, you can choose any arbitary name for cluster and make sure buck name matches the one you created in previous step.

export KOPS_CLUSTER_NAME=maitrihome.in				(Cluster Name same as DNS)
export KOPS_STATE_STORE=s3://maitrihome-state-store

Then running command to reflect variables added to .bashrc
```
	source ~/.bashrc
```
### 8. Create ssh key pair
This keypair is used for ssh into kubernetes cluster

ssh-keygen

### 9. Create a Kubernetes cluster definition.
```sh
kops create cluster \
--state=${KOPS_STATE_STORE} \
--node-count=2 \
--master-size=t2.micro \
--node-size=t2.micro \
--zones=ap-south-1a,ap-south-1b \
--name=${KOPS_CLUSTER_NAME} \
--dns private \
--master-count 1
```

### 10. Create kubernetes cluster

kops update cluster --yes

Above command may take some time to create the required infrastructure resources on AWS. Execute the validate command to check its status and wait until the cluster becomes ready

************* ABOVE STEP WILL TAKE ATLEAST 10 MINS*************

kops validate cluster

For the above above command, you might see validation failed error initially when you create cluster and it is expected behaviour, you have to wait for some more time and check again.

### 11. To connect to the master
```sh
ssh admin@api.javahome.in
```
# Destroy the kubernetes cluster
```sh
kops delete cluster  --yes
```

## Update Nodes and Master in the cluster
We can change numner of nodes and number of masters using following commands
```
   kops edit ig nodes change minSize and maxSize to 0
   kops get ig- to get master node name
   kops edit ig - change min and max size to 0
   kops update cluster --yes
 
```
# Optional (Create terraform scripts through kops)

```
  https://github.com/kubernetes/kops/blob/master/docs/terraform.md

```
