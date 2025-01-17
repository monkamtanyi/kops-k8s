## kops-kubernetes-cluster-configuration
## Landmark Technologies,  -    Landmark Technologies 
## Tel: +1 437 215 2483,   -     +1 437 215 2483 
## mylandmarktech@gaIL.com,  -    www.mylandmarktech.com ,

## Setting up Kubernetes (K8s) Cluster on AWS Using KOPS

1.kops is a software use to create production ready k8s cluster in a cloud provider like AWS.

2. kOPS SUPPORTS MULTIPLE CLOUD PROVIDERS

3. Kops compete with managed kubernestes services like EKS, AKS and GKE

4. Kops is cheaper than the others.

5. Kops create production ready K8S.

6. KOPS create resources like: LoadBalancers, ASG, Launch Configuration, woker node Master node (CONTROL PLANE.

7. KOPS is IaaC

#!/bin/bash
## 1) Create Ubuntu EC2 instance in AWS

## 2a) create kops user
``` sh
 sudo adduser kops
 sudo echo "kops  ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/kops
 sudo su - kops
 ```
 ##  2b) install AWSCLI using the apt package manager
  ```sh
 sudo apt update
 sudo apt install awscli -y 
 ```
 ## or 2b1) install AWSCLI using the script below this is the one that worked
 ```sh
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws --version
 ```
## 3) Install kops software on an ubuntu instance by running the commands below:
 	kops@master:~$
        sudo apt install wget -y
 	sudo wget https://github.com/kubernetes/kops/releases/download/v1.22.0/kops-linux-amd64
 	sudo chmod +x kops-linux-amd64
 	sudo mv kops-linux-amd64 /usr/local/bin/kops
        kops version
 
## 4) Install kubectl kubernetes client if it is not already installed
```sh
 kubectl get all   #check if kubectls is installed
 sudo curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
 sudo chmod +x ./kubectl
 sudo mv ./kubectl /usr/local/bin/kubectl
```
## 5) Create an IAM role from AWS Console or CLI with the below Policies. 

	AmazonEC2FullAccess 
	AmazonS3FullAccess
	IAMFullAccess 
	AmazonVPCFullAccess

Then Attach IAM role to ubuntu server from Console Select KOPS Server --> Actions --> Instance Settings --> Attach/Replace IAM Role --> Select the role which
You Created. --> Save.

## 6) create an S3 bucket
## Execute the commands below in your KOPS control Server. use unique s3 bucket name. If you get bucket name exists error.
	aws s3 mb s3://classkops
	aws s3 ls # to verify
	
 ## 6b) create an S3 bucket    Set e't varaibles.
	Expose environment variable:
    # Add env variables in bashrc
    
       vi .bashrc
	# Give Unique Name And S3 Bucket which you created. Paste below in .bashrc
	export NAME=team2clas.k8s.local
	export KOPS_STATE_STORE=s3://classkops
 #info about my clustre wil be stored in d s3 bucket created.
 
      source .bashrc   #refresh d file
	
### 7) Create sshkeys before creating cluster
```sh
ssh-keygen -t rsa -b 4096  #-b for base
click enter to d end.
```

# 8) Create kubernetes cluster definitions on S3 bucket
```
#ls .ssh   // copy key and replace d end code of line 3 (kops create secret...)
kops create cluster --zones us-east-1a --networking weave --master-size t2.medium --master-count 1 --node-size t2.medium --node-count=2 team2clas.k8s.local 
# copy the sshkey into your cluster to be able to access your kubernetes node from the kops server
kops create secret --name team2clas.k8s.local sshpublickey admin -i ~/.ssh/id_rsa.pub  #id key copied id_rsa.pub
#kops create secret --name ${NAME} sshpublickey admin -i ~/.ssh/id_rsa.pub
 
```
Play around w d suggestions in 10a
kops get cluster
kops edit cluster class.k8s.lacal  #use for editing if need be



# 9) Initialise your kops kubernetes cluser by running the command below
```sh
kops update cluster team2clas.k8s.local --yes    #will take a while to update.
```
# 10a) Validate your cluster(KOPS will take some time to create cluster ,Execute below commond after 3 or 4 mins)

wait for about 10min b4 u run d valiate command

sudo kops validate cluster
	   
	   Suggestions:
 * validate cluster: kops validate cluster --wait 10m
 * list nodes: kubectl get nodes --show-labels
 * ssh to the master: ssh -i ~/.ssh/id_rsa ubuntu@api.class.k8s.local
 * the ubuntu user is specific to Ubuntu. If not using Ubuntu please use the appropriate user based on your OS.
 * read about installing addons at: https://kops.sigs.k8s.io/operations/addons.

## 10b - Export the kubeconfig file to manage your kubernetes cluster from a remote server. For this demo, Our remote server shall be our kops server 
```sh
 kops export kubecfg $NAME --admin # did not wk prof did sudo su - kops  ran
kops@master:~$
```
## 11a) To list nodes and pod to ensure that you can make calls to the kubernetes apiSAerver and run workloads
	  kubectl get nodes 
 kubectl get po        #no resource in default ns
assume we want to change our current namespace from default to dev
kubectl create ns dev
chenge d default ns context to dev
kubectl config set-context --current --namespace=dev
kubectl get po

### 11b) Alternative you can ssh into your kubernetes master server using the command below and manage your cluster from the master
    sh -i ~/.ssh/id_rsa ubuntu@ipAddress
    ssh -i ~/.ssh/id_rsa ubuntu@18.222.139.125
    ssh -i ~/.ssh/id_rsa ubuntu@172.20.58.124

### 11b. Alternative, Enable PasswordAuthentication in the master server and assign passwd
```sh
sudo sed -i "/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
sudo service sshd restart
sudo passwd ubuntu
```

### 11c) To list nodes

	  kubectl get nodes 
 
## 12) To Delete Cluster

   kops delete cluster --name=${NAME} --state=${KOPS_STATE_STORE} --yes  
   
====================================================================================================


13 # IF you want to SSH to Kubernetes Master or Nodes Created by KOPS. You can SSH From KOPS_Server

sh -i ~/.ssh/id_rsa ubuntu@ipAddress
ssh -i ~/.ssh/id_rsa ubuntu@3.90.203.23
  
``
