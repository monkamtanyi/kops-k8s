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
 
IAM>role>aws service>service use case>ec2>create>select the above permission policies>next>role name= kops35>create role.
Then Attach IAM role to ubuntu server from Console Select KOPS Server --> Actions --> Instance Settings --> Attach/Replace IAM Role --> Select the role which
You Created. --> Save.
aws s3 ls # to verify

## 6) create an S3 bucket
## Execute the commands below in your KOPS control Server. use unique s3 bucket name. If you get bucket name exists error.
	aws s3 mb s3://classkops
	aws s3 ls # to verify
	
 ## 6b) create an S3 bucket    Set e't varaibles.
	Expose environment variable:
    # Add env variables in bashrc
    
       vi .bashrc
	# Give Unique Name And S3 Bucket which you created. Paste below in .bashrc
	export NAME=team2class.k8s.local    #name your cluster
	export KOPS_STATE_STORE=s3://classkops
 #info about my clustre wil be stored in d s3 bucket created.

 head .bashrc
 echo $NAME
 
      source .bashrc   #refresh d file
  head .bashrc
 echo $NAME    
	
### 7) Create sshkeys before creating cluster
```sh
ssh-keygen -t rsa -b 4096  #-b for base    #ssh-keygen
click enter to d end.
```

# 8) Create kubernetes cluster definitions on S3 bucket
```
#ls .ssh   // copy key and replace d end code of line 3 (kops create secret...)
kops create cluster --zones us-east-1a --networking weave --master-size t2.medium --master-count 1 --node-size t2.medium --node-count=2 team2class.k8s.local 
# copy the sshkey into your cluster to be able to access your kubernetes node from the kops server
kops create secret --name team2clas.k8s.local sshpublickey admin -i ~/.ssh/id_rsa.pub  #id key copied id_rsa.pub #ls .ssh  to see keys
#kops create secret --name ${NAME} sshpublickey admin -i ~/.ssh/id_rsa.pub

ls .ssh  to see keys
```
Play around w d suggestions in 10a
kops get cluster
kops edit cluster class.k8s.lacal  #use for editing if need be



# 9) Initialise your kops kubernetes cluser by running the command below
```sh
kops update cluster team2clas.k8s.local --yes    #will take a while to update.
prof used kops update cluster ${NAME} --yes
```
# 10a) Validate your cluster(KOPS will take some time to create cluster ,Execute below commond after 3 or 4 mins)


sudo kops validate cluster --wait 10min
	   
	   Suggestions:
 * validate cluster: kops validate cluster --wait 10m
 * list nodes: kubectl get nodes --show-labels
 * ssh to the master: ssh -i ~/.ssh/id_rsa ubuntu@api.class.k8s.local
 * the ubuntu user is specific to Ubuntu. If not using Ubuntu please use the appropriate user based on your OS.
 * read about installing addons at: https://kops.sigs.k8s.io/operations/addons.

## 10b - Export the kubeconfig file to manage your kubernetes cluster from a remote server. For this demo, Our remote server shall be our kops server 
```sh

 kops export kubecfg $NAME --admin # did not wk prof did:  sudo su - kops b4 running this command 
kops@master:~$
```
## 11a) To list nodes and pod to ensure that you can make calls to the kubernetes apiSAerver and run workloads
	  kubectl get nodes 
 kubectl get po        #no resource in default ns
assume we want to change our current namespace context from default to dev
kubectl create ns dev
chenge d default ns context to dev
kubectl config set-context --current --namespace=dev
kubectl get po   
NEXT Prof deployed this app w is a complete manifest file:
Goto XXXX below
yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy

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

ssh -i ~/.ssh/id_rsa ubuntu@ipAddress
ssh -i ~/.ssh/id_rsa ubuntu@3.90.203.23
****************************************************************************************************************************************
XXXX
suppose u were gven this ticket to deploy aa production ready k8s cluster using kops.
Answer: kops is a software used to deploy producation ready k8s cluster.
github documentation: https://github.com/LandmarkTechnology/kops-k8s

we have this script below to deploy an appl.
https://github.com/LandmakTechnology/kubernetes-manifests/blob/main/SpringBoot-Mongo-DynamicPV.yml 

main manifest file repo https://github.com/monkamtanyi/kubernetes-manifests
check in SpringBoot-Mongo-DynamicPV.yml

this is a springapp deployment, replica 2, a load balancer service in aws, persistentvolumeClaim,
replicaset for mongodb, service for mongodb with clusterIp.

kubectl get pv
no resource found.
kubectl get sc
there is no pv found here. but we have some dynamic storage classes. with this dynamic storage classes
even without creating a pv, it will create a pvc, it will automatically create a pv for us.

raw file copied below
========
vi app.yml    paste and save.


# this deploymt is a Complete Manifest, Where in a single yml, we define 
#Deployment & Service for SpringApp & PVC(with default  StorageClass),
#ReplicaSet & Service For Mongo.)

apiVersion: apps/v1
kind: Deployment
metadata:
  name: springapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: springapp
  template:
    metadata:
      name: springapppod
      labels:
        app: springapp
    spec:
      containers:
      - name: springappcontainer
        image: mylandmarktech/spring-boot-mongo
        ports:
        - containerPort: 8080
        env:
        - name: MONGO_DB_USERNAME
          value: devdb
        - name: MONGO_DB_PASSWORD
          value: devdb@123
        - name: MONGO_DB_HOSTNAME
          value: mongo 
---
apiVersion: v1
kind: Service
metadata:
  name: springapp
spec:
  selector:
    app: springapp
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodbpvc 
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mongodbrs
spec:
  selector:
    matchLabels:
      app: mongodb
  template:
     metadata:
       name: mongodbpod
       labels:
         app: mongodb
     spec:
       volumes:
       - name: pvc
         persistentVolumeClaim:
           claimName: mongodbpvc     
       containers:
       - name: mongodbcontainer
         image: mongo
         ports:
         - containerPort: 27017
         env:
         - name: MONGO_INITDB_ROOT_USERNAME
           value: devdb
         - name: MONGO_INITDB_ROOT_PASSWORD
           value: devdb@123
         volumeMounts:
         - name: pvc
           mountPath: /data/db   
---
apiVersion: v1                                       ###
kind: Service
metadata:
  name: mongo
spec:
  type: ClusterIP
  selector:
    app: mongodb
  ports:
  - port: 27017
    targetPort: 27017

kubectl get all     # there is no deployment in d cluster.
kubectl get all        no service
check us -east-1a, N. birginia, to see what objects has been created.
goto aws acc>instances running>1 master, 2 wkernodes were created.
once we did kops create cluster, it created all of these objects for us.

It also created auto scaling gps>u will see an autoscaling gp for master and wkeer node
it also created some load balancers 2 prof said it seems we have 2 kops cluster. tried to del one but left it.
he del lb for class35(avoid cost)

~$ kubectl apply -f app.yml
deployment.apps/springapp created
service/springapp created
persistentvolumeclaim/mongodbpvc created
replicaset.apps/mongodbrs created
service/mongo created

kubectl get pod   all pods are running
kubectl get svc
a service is created n it has created a load balancer in aws.
gto LB in aws, refresh, u will see another lb created.

kops@master:~$ kubectl get sc
NAME                      PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
default                   kubernetes.io/aws-ebs   Delete          Immediate              false                  110m
gp2                       kubernetes.io/aws-ebs   Delete          Immediate              false                  110m
kops-csi-1-21 (default)   ebs.csi.aws.com         Delete          WaitForFirstConsumer   true                   110m
kops-ssd-1-17             kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   true                   110m
kops@master:~$
sc - we have some dynamic storage classes sc
with this dynamic storage classes storage classes even without creating a pv, if 
we create a pvc it will automatically create a pv for us.


copy LB url and take to d browser
we can access our appl. yes.

is d appl communicating with our db?
fill info and submit. yes it is.

Is our data persisting, check it out
kubectl get pod



kubectl get pvc   # pvc was created

kubectl get pv        # pv was created

kubectl describe pod mongodbrs-nfggp  (from get po)

our db pod was created in d dev ns, it is scheduled on one of d node, it has a label, has a poort Ip.
it is controlled by a replicaset called mongodbrs, image is mongo from dkhub,e't var, devdb, devdb@123

data is mounted as follows - /data/db from pvc (rw) read/write perm. that's why users can write data.

PV vs PVC
PV piece of storage created in ur cluster.
PVC when that volume PV is being used by a cluster - a pod claims d vol or is making use of d vol.



nginx-ingress contoller:
==========================
https://github.com/LandmakTechnology/nginx-ingress
go thro nginx-ingress video


kind: Pod
apiVersion: v1
metadata:
  name: web
  labels:
    app: web  
spec:
  containers:
  - name: web
    image : mylandmarktech/hello
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: websvc
spec:
  selector:
    app: app   
  ports:
  - port: 80
    targetPort: 80

---
kind: Pod
apiVersion: v1
metadata:
  name: webapp   
  labels:
    app: webapp     
spec:
  containers:
  - name: web
    image : mylandmarktech/java-web-app  
    ports:
    - containerPort: 8080  
---
apiVersion: v1
kind: Service
metadata:
  name: webappsvc
spec:
  selector:
    app: webapp   
  ports:
  - port: 80
    targetPort: 8080 
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress-rule
spec:
  ingressClassName: nginx
  rules:
  - host: mylandmarktech.net
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: springappsvc
            port:
              number: 80
  - host: hello.mylandmarktech.net
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: websvc 
            port:
              number: 80
  - host: web.mylandmarktech.net
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: webappsvc
            port:
              number: 80

apiVersion: v1  
kind: Pod 
metadata:
  name: app  
  namespace: dev  
  labels:  
    app: fe  
    name: paul  
spec:  

kubernetes 13--AUGUST 13, 2023
ResourceQuota,   
LimitRange,
StatefulSets  
nginx-ingress

  
``
