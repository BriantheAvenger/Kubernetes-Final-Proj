---
title: "Simple Nginx Pods Setup"
date: 2019-05-07T15:53:17-07:00
draft: false
layout: 'posts'
tags: ["Pods", "Setup"]
---
# Pods setup
I started my setup of pods by launching 3 replicas of nginx deployments which would allow me to ensure that there would be 3 pods. Below is the yaml part that I ended up using to launch them:
```
controllers/nginx-deployment.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```
I went ahead and decided to put in place a much more enforced autoscaling policy by inputting the following into the terminal:
```
kubectl autoscale deployment.v1.apps/nginx-deployment --min=10 --max=15 --cpu-percent=80
```