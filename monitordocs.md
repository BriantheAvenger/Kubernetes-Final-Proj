---
title: "Kubernetes Dashboard setup"
date: 2019-05-07T15:53:17-07:00
draft: false
layout: 'posts'
tags: ["Metrics", "Monitor", "Dashboard"]
---
# How to setup Helm along with Tiller
Helm helps us to put in place metrics and we install tiller so that it setups the server side of the metrics part.
Installation of Helm through the use of brew Note: you may have to install brew if its not already included
```
brew install kubernetes-helm
```
Next we can go ahead and create a namespace for tiller which is supposed to be the server side of helm. After this pretty much issues start arising with the way that certain things are being ran so this will pretty much be the last time working with tiller as i hoped on over to kubernetes dashboard for a much more clearer representation of things plus it was more easier to implement. I would as well reccomend using prometheus with helm but it didnt cross my mind until I had already started on kubernetes dashboard. The following will create the kubernetes dashboard:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

Apart from the dashboard we as well need heapster which will give us options on what metrics we can display. The second line allows to install needed backend settings for our cluster. And the third line applies the cluster role binding:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/heapster.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/influxdb.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/rbac/heapster-rbac.yaml
```
Heapster isnt as fully supported as prometheus is I did not realize until nearly completing it but that is the main reason why i would reccomend prometheus next time instead of the kubernetes dashboard. 
We first have to create a yaml file that will allow for eks account access note that this will grant full permissions to the cluster. The last part is just applying that role as similar as to what has been done in the first steps of setting up monitoring.
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: eks-admin
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: eks-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: eks-admin
  namespace: kube-system
```
```
kubectl apply -f eks-admin-service-account.yaml
```


The following is what gets spit out I have modified part of the token by deleting some of it and adding other characters just as a means to try and stay secure incase anyone happens to come across this documentation
```
Name:         eks-admin-token-zfm8c
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name=eks-admin
              kubernetes.io/service-account.uid=4acb2382-758f-11w9-6f79-123b01168c28

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSwitoanj2235946IiJ9.eyJpc3wieurjojr8394iOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc27dhrku94dW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZWtzLWFkbWluIiwia3ViZXJuZXRlcy5jrjoiowlapurnpby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiNWJjYjIzODItNzI0Zi0xMWU5LThiOTgtMDIzZTAxMTQ4ZjI4Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmVrcy1hZG1pbiJ9.SrpF6KjDzdpbH7lTkuKO4aEbxEi6I8Fd3rP6MpkBt_ZwEGo8b1_kGFORPtKVN3JfLGSrbZNsVxRpZBqzRo69B8Nf3pXnA6fEUcwHwPxcCl164TGQkXR4rb61ki25hVipH2tzNWArUMM2ewQ2wf3XYLnLP5-2io9mYb1qbI_7T6YCfuV6IhEBNfxKG01qtCey8Mnw47hpSpfK9bymA-fqH_gkFX8Sw8djkeu4k3dj3sFD6KBOXtDtmaiXDIt1zmDvqdk2mRl8ylcIccqyx3Q
ca.crt:     1025 bytes
namespace:  11 bytes
```
Next we will apply kubernetes proxy and follow the link to our dashboard note that after applying the command you will need to copy paste your token and choose to sign in through the token option so that you may see a graphical representation.
```
kubectl proxy
```