---
title: "Kubernetes Cluster setup with EKS"
date: 2019-05-07T15:53:17-07:00
draft: false
layout: 'posts'
tags: ["Aws EKS", "Setup", "Nodes"]
---
# Documentation for setting up Amazon EKS along with Cloud Formation
## Initial setup
Prior to the start of setting everything up you must make sure to have a dedidacted eks service role in place which can be done through the IAM category of the console and you will as well need a vpc with at least three subnets that will get created by default. To start off with the main part we need to install kubectl so that we can configure EKS with it next we apply the appropriate settings. And third, I end up copying adding path where im gonna be working in and as well adding it to the environmental variables in the .bashrc file:
```
curl -o aws-iam-authenticator https://amazon-eks.s3-us-west-2.amazonaws.com/1.12.7/2019-03-27/bin/linux/amd64/aws-iam-authenticator
chmod +x ./aws-iam-authenticator
mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
```
## Amazon EKS setup
Most of this was done through point and click. When setting up the cluster you can name it whatever you want but keep track of the name as it will be important later on for the nodes. The main thing to pay attention to is that you select the correct vpc that you created and choose the subnets that come with it as well as make sure that you create and attach a specific security group for this cluster.
## Kube Configuration File
The next part of this plan is to go ahead and create a kubectl configuration file. we first have to make sure that aws cli is installed and then we can go ahead and run the update-kubeconfig command to update the settings with our cluster. Right after I ran the svc command so that it can display the cluster that I have up and running 
```
aws eks --region us-west-2 update-kubeconfig --name eks_cit481clus
kube get svc
```
## Creating worker nodes
I will continue setting up the rest of the kubernetes cluster by adding a worker node. Once again I will be using cloudformation and installing a template. First step is to create another stack and choose to load a specified template by inputting the following url:
```
https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-02-11/amazon-eks-nodegroup.yaml
```
The next step is to go ahead and specify the details the main important thing to note is that the cluster name must be exactly named as the cluster when we first created it in the earlier part. Along with that the cluster control plane secuirty group should have a dedicated security group for this rundown I will be using the security group that was created earlier, which should be made available on the drop down menu. The Node group name along with all the autoscaling settings is up to personal preference. For my instance type I went ahead with a t2.small as opposed to a spot instance.

Since I am using Kubernetes version 1.1 I went ahead and used the following ami id to specify things:
```
ami-05ecac759c81e0b0c
```
Note that for the next step of the details page it is required to have an ssh key pair which should be a .pem file ssh provides your own way of creating one or you can go to the EC2 seection of the aws console to create one. The next part will not work unless a security key pair is in place for the worker node. The vpc and subnets being used should be the ones that we created earlier the same rule applys with the three subnets that we created in the vpc.

In order to enable the worker nodes I went ahead and used a yaml file that had been prebuilt below is the link to downloading it:
```
curl -o aws-auth-cm.yaml https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-02-11/aws-auth-cm.yaml
```
Replace the rolearn with the one that is your own and it should work.
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: arn:aws:iam::647338590706:role/cit481-worker-nodes
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes
```
Lastly, we will need to apply our changes and we can as well display our nodes after applying them with the second watch flag. 
```
kubectl apply -f aws-auth-cm.yaml

kubectl get nodes --watch
```
