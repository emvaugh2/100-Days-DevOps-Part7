# 100 Days of DevOps with KodeKloud - Days 61 - 70

We're now in the 60s! This is still mostly Kubernetes (K8s) labs but we'll end off running some basic Jenkins labs. Lets get it started. 

## Day 70: Configure Jenkins User Access
## Day 69: Install Jenkins Plugins
## Day 68: Set Up Jenkins Server
## Day 67: Deploy Guest Book App on Kubernetes
## Day 66: Deploy MySQL on Kubernetes
## Day 65: Deploy Redis Deployment on Kubernetes
## Day 64: Fix Python App Deployed on Kubernetes Cluster
## Day 63: Deploy Iron Gallery App on Kubernetes
## Day 62: Manage Secrets in Kubernetes



## Day 61: Init Containers in Kubernetes

Okay for this lab, we need to create a deployment that has an initContainer and a main container with the specifications listed in the lab. I'm going to pull some generic templates and use those to get started. I'll have AI check it afterwards. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ic-deploy-xfusion
  labels:
    app: ic-xfusion
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ic-xfusion
  template:
    metadata:
      labels:
        app: ic-xfusion
    spec:
      volumes:
        - name: ic-volume-xfusion
          emptyDir: {}

      initContainers:
        - name: ic-msg-xfusion
          image: fedora:latest
          command:
            - /bin/bash
            - -c
            - |
              echo Init Done - Welcome to xFusionCorp Industries > /ic/blog
          volumeMounts:
            - name: ic-volume-xfusion
              mountPath: /ic

      containers:
        - name: ic-main-xfusion
          image: fedora:latest
          command:
            - /bin/bash
            - -c
            - |
              while true; do cat /ic/blog; sleep 5; done
          volumeMounts:
            - name: ic-volume-xfusion
              mountPath: /ic
```

Unforunately Google kind of just gave me the entire template when I was searching for a website to put it all together. I'll just use one of these going forward. Lab completed. The takeway is initContainers run to completion before the main container starts. They're useful for when you need something prepared for the main container before starting it. I would think about this along the lines of a multi-stage Dockerfile. You build all the necessary tools to get the image started and throw away what you no longer need to keep things lightweight. 
