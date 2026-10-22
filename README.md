# 100 Days of DevOps with KodeKloud - Days 61 - 70

We're now in the 60s! This is still mostly Kubernetes (K8s) labs but we'll end off running some basic Jenkins labs. Lets get it started. 

## Day 70: Configure Jenkins User Access



## Day 69: Install Jenkins Plugins



## Day 68: Set Up Jenkins Server



## Day 67: Deploy Guest Book App on Kubernetes



## Day 66: Deploy MySQL on Kubernetes



## Day 65: Deploy Redis Deployment on Kubernetes



## Day 64: Fix Python App Deployed on Kubernetes Cluster

This lab is all about troubleshooting. You already know the gameplan. Lets see as many logs as we can find. Everything looks good on the deployment side (well mostly everything). After running `kubectl get pods`, I'm seeing there's an issue with the image. The image is `poroko/flash-app-demo`. It can't find this image. I don't even know if this is a real image or what the actual image should be. After rereading the prompt, it's supposed to be using `poroko/flash-dem-app` so we need to swap the wording. We'll fix that using `kubectl edit deployment`. Lets focus on the service though. It's supposed to be using nodePort 32345 and the targetPort should be the python flask app's default port. I'm assuming this is TCP 5000 based on the pod saying `Port: 5000/TCP`. Okay I used `kubectl edit svc` to change the targetPort to 5000. After I checked both edits for a running status, I pressed the app button on KodeKloud and I received a message saying `Hello World Pyvo 1!`

I GOT THE GREEN CHECK. Heck yeah!

## Day 63: Deploy Iron Gallery App on Kubernetes

We need to create an app named Iron Gallary on K8s. It needs it's own namespace and deployment. It'll actually be two deployments so lets just get that started. We'll use one of our previous deployment manifests. Use the command `kubectl create namespace <name>` for the namespace as well. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-gallery-deployment-xfusion
  namespace: iron-namespace-xfusion
  labels:
    run: iron-gallery
spec:
  replicas: 1
  selector:
    matchLabels:
      run: iron-gallery
  template:
    metadata:
      labels:
        run: iron-gallery
    spec:
      containers:
        - name: iron-gallery-container-xfusion
          image: kodekloud/irongallery:2.0
          resources:
            limits:
              memory: "100Mi"
              cpu: "50m"
          volumeMounts:
            - name: config
              mountPath: /usr/share/nginx/html/data

            - name: images
              mountPath: /usr/share/nginx/html/uploads
            
      volumes:
        - name: config
          emptyDir: {}

        - name: images
          emptyDir: {}

```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-db-deployment-xfusion
  namespace: iron-namespace-xfusion
  labels:
    db: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      db: mariadb
  template:
    metadata:
      labels:
        db: mariadb
    spec:
      containers:
        - name: iron-db-container-xfusion
          image: kodekloud/irondb:2.0
          env:
            - name: MYSQL_DATABASE
              value: "database_host"
            -  name: MYSQL_ROOT_PASSWORD
              value: "letsgeteverythingwedeserve5678"
            - name: MYSQL_PASSWORD
              value: "ILoveGodDoYouLoveGod?9876"
            -  name: MYSQL_USER
              value: "edward_1234"
          volumeMounts:
            - name: db
              mountPath: /var/lib/mysql
      volumes:
        - name: db
          emptyDir: {}
```

```
apiVersion: v1
kind: Service
metadata:
  name: iron-db-service-xfusion
  namespace: iron-namespace-xfusion
spec:
  selector:
    db: mariadb
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  type: ClusterIP
```

```
apiVersion: v1
kind: Service
metadata:
  name: iron-gallery-service-xfusion
  namespace: iron-namespace-xfusion
spec:
  type: NodePort
  selector:
    run: iron-gallery
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 32678
```

Okay apparently I needed to fix three things. Everywhere in my gallery deployment output, I need to change the `app:` text to `run:`. I next needed to change my volumes section. Both sections need to be condensed into one section and not two parts. Lastly, I had the same issue with the DB manifest files. I had to switch `app:` to `db:`. I really need to spend a few days just writing out these templates myself and learning what each part means because this is consistently messing me up. I also decided to put the namespace in the manifest files as well. I didn't know what I was able to do that but it makes the apply commands a little shorter. 

I'm going to deploy these using the deployment files first and then the service files. Then we'll see if we get the Iron Gallery installation page. Well that worked! Green check. 

## Day 62: Manage Secrets in Kubernetes

I'm completely new to Kubernetes secrets so I'm going to lean on AI for this lab. I'm also very sleepy so this will be my last lab for the day. 

We need to create a generic secret named blog and we have the password in a file called blog.txt. We need to create a Pod for this as well. We need to use the sleep command for the container as well. I believe I can do all of this myself besides the secret part. 

Lets create the secret. We'll use the command `kubectl create secret generic blog \  --from-file=/opt/blog.txt` to do so. I used the `kubectl get secret` command to get the name of the secret. Now, lets create the Pod manifest using the details in the lab and the secret we just created. 

```
apiVersion: v1
kind: Pod

metadata:
  name: secret-nautilus

spec:
  containers:
    - name: secret-container-nautilus
      image: fedora:latest

      command:
        - /bin/bash
        - -c
        - sleep infinity

      volumeMounts:
        - name: blog-secret-volume
          mountPath: /opt/demo
          readOnly: true

  volumes:
    - name: blog-secret-volume
      secret:
        secretName: blog
```



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
