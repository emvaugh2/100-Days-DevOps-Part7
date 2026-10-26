# 100 Days of DevOps with KodeKloud - Days 61 - 70

We're now in the 60s! This is still mostly Kubernetes (K8s) labs but we'll end off running some basic Jenkins labs. Lets get it started. 

## Day 70: Configure Jenkins User Access



## Day 69: Install Jenkins Plugins



## Day 68: Set Up Jenkins Server

Just in case I run out of time today. Just need to keep my streak. 

We'll knock out a few of these tomorrow. Or Monday. 

Alright fareal next time. 

Knocking out 5 of these today. 

## Day 67: Deploy Guest Book App on Kubernetes

THIS IS OUR FINAL K8s LAB! We learned a lot, didn't we? Lets finish it up. I pretty much got this all the way by myself. The labeling was a little off and I had an indentation error but outside of those things, I was good to go. I did ask AI what order I should deploy all of these in so I left them in the right order for future use. 

NICE. GREEN CHECK. You're a K8s expert at this point. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
  labels:
    app: redis-master

spec:
  replicas: 1

  selector:
    matchLabels:
      app: redis-master

  template:
    metadata:
      labels:
        app: redis-master

    spec:
      containers:
        - name: master-redis-datacenter
          image: redis
          resources:
            requests:
              cpu: "100m"
              memory: "100Mi"
          ports:
            containerPort: 6379

```

```
apiVersion: v1
kind: Service
metadata:
  name: redis-master

spec:
  type: ClusterIP

  selector:
    app: redis-master

  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
  labels:
    app: redis-slave

spec:
  replicas: 2

  selector:
    matchLabels:
      app: redis-slave

  template:
    metadata:
      labels:
        app: redis-slave

    spec:
      containers:
        - name: slave-redis-datacenter
          image: gcr.io/google_samples/gb-redisslave:v3
          env:
            - name: GET_HOSTS_FROM
              value: "dns"
          resources:
            requests:
              cpu: "100m"
              memory: "100Mi"
          ports:
            containerPort: 6379


```

```
apiVersion: v1
kind: Service
metadata:
  name: redis-slave

spec:
  type: ClusterIP

  selector:
    app: redis-slave

  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379
```

```
apiVersion: v1
kind: Service
metadata:
  name: redis-follower

spec:
  type: ClusterIP

  selector:
    app: redis-slave

  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: frontend

spec:
  replicas: 3

  selector:
    matchLabels:
      app: frontend

  template:
    metadata:
      labels:
        app: frontend

    spec:
      containers:
        - name: php-redis-datacenter
          image: gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff
          env:
            - name: GET_HOSTS_FROM
              value: "dns"
          resources:
            requests:
              cpu: "100m"
              memory: "100Mi"
          ports:
            containerPort: 80


```

```
apiVersion: v1
kind: Service
metadata:
  name: frontend

spec:
  type: NodePort

  selector:
    app: frontend

  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30009
```

## Day 66: Deploy MySQL on Kubernetes

We needed to deploy a MySQL instance using different Kubernetes manifests. We needed to included environment variables in our deployment file while also creating a Secrets manifest to store the values of the environment variable keys. We also need a persistent volume and volume claim to store at data from the container. Lastly, we need to create a Service manifest to expose port 30002 while still serving ports 3306. 

My biggest learning lesson here was supplying the Secrets values to the deployment. I've never done that before and didn't know what it was supposed to look like so I leaned on AI for this. I used my manifests from my previous labs to piece this deployment together. I also didn't know the ports for MySQL. I thought it was 1433 but it was 3306. Everything else I got correct though so that's also a confidence booster. 

Green check. Lets go. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
  labels:
    app: mysql

spec:
  replicas: 1

  selector:
    matchLabels:
      app: mysql

  template:
    metadata:
      labels:
        app: mysql

    spec:
      containers:
        - name: mysql-container
          image: mysql:latest

          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-root-pass
                  key: password

            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-db-url
                  key: database

            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: username

            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: password

          ports:
            - containerPort: 3306

          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql

      volumes:
        - name: mysql-storage
          persistentVolumeClaim:
            claimName: mysql-pv-claim

```


```
apiVersion: v1
kind: Service
metadata:
  name: mysql

spec:
  type: NodePort

  selector:
    app: mysql

  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
      nodePort: 30007
```
```
apiVersion: v1
kind: Secret
metadata:
  name: mysql-root-pass
type: Opaque
stringData:
  password: "YUIidhb667"
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-user-pass
type: Opaque
stringData:
  username: "kodekloud_gem"
  password: "LQfKeWWxWD"
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-db-url
type: Opaque
stringData:
  database: "kodekloud_db2"


```

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim

spec:
  storageClassName: manual

  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 250Mi
```

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  storageClassName: manual
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/var/lib/mysql"
```



## Day 65: Deploy Redis Deployment on Kubernetes

Lets get this deployment underway! So I'm going to rely on my previous deployment templates to get this started. We also need to create a ConfigMap so this will be the first time for that. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
  labels:
    app: redis
spec:
  replicas: 1

  selector:
    matchLabels:
      app: redis

  template:
    metadata:
      labels:
        app: redis

    spec:
      containers:
        - name: redis-container
          image: redis:alpine
          ports: 
            - containterPort: 6379
          resources:
            requests:
              cpu: "1"
          volumeMounts:
            - name: data
              mountPath: /redis-master-data

            - name: redis-config
              mountPath: /redis-master
            
      volumes:
        - name: data
          emptyDir: {}

        - name: redis-config
          configMap:
            name: my-redis-config
            items:
              - key: redis-config
                path: redis.conf 

```

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-redis-config

data:
  redis-config: |
    maxmemory 2mb
    maxmemory-policy allkeys-lru
```

UPDATE: Okay I did the best I could. I've never worked with a ConfigMap and the K8s documentation wasn't clear. Going to lean on AI to see what I need to fix. I had to make some changes to the volumes: section regarding the ConfigMap section. I still don't understand what it's doing but I'll ask for an explanation later. Apparently, we need to deploy the ConfigMap first and then the deployment. Lets do that. Got the green check mark. Lets go over what we did regarding the ConfigMap part. 

AI explanations:
- ConfigMap - a K8s API object that stores non-sensitve (so no Secrets) configuration separately from your container image. It stores configuration key-value pairs and the Pods can consume these are environment variables.
- ConfigMap manifests use the `data:` section. The key is redis-config. The values are maxmemory 2mb and maxmemroy-policy allkeys-lru.
- volume: name: redis-config is just the volume's internal name.
- The configMap part connects the Pod to this ConfigMap object.
- The items: section says retrieve the key called redis-config (remember, that key has two values) and expose that key as a file named redis.conf. So basically it's just placing those values into that file.
- Create the ConfigMap first before the deployment.
- Manifest - a file containing the declarative definition of one or more K8s resources. Remember, deployments, pods, services, configmaps, secrets, etc are all K8s OBJECTS. The manifests are the files that describe the objects so we call everything a manifest. 

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
