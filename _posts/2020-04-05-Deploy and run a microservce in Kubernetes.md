---
title: "Deploy and Run a microservice in Kubernetes"
toc: true
toc_label: "Content"
toc_sticky: true
tags:
  - Deploy and Run a microservice in Kubernetes

date: 2020-04-05
header:
  teaser: /assets/images/nature.jpg
excerpt: "This blog covers little basics of Kubernetes & Container Orchestration in general and then details out how to deploy a Flask-based microservice (along with Postgres and React.js) to a Kubernetes cluster"
---

This blog covers little basics of Kubernetes & Container Orchestration in general and then details out how to deploy a Flask-based microservice (along with Postgres and React.js) to a Kubernetes cluster.

Dependencies:

- Docker Engine - CE: Client: v19.03.8, Server: v19.03.8
- Kubectl: v1.18.0
- Minikube: v1.9.2
- Oracle VirtualBox: v6.1


{% if page.toc %}
  {% include toc.html %}
{% endif %}

## Objectives

1. Understand what is container orchestration
1. Pros and cons of using Kubernetes over other orchestration tools like Docker Swarm and Elastic Container Service (ECS)
1. Kubernetes Architecture
1. Kubernetes Objects - Pod, Service, Volumes, Namespace etc.
1. Configure a Kubernetes cluster to run locally with Minikube
1. Use Kubernetes Secrets to manage sensitive information
1. Set up a volume to hold Postgres data within a Kubernetes cluster
1. Run Flask, Gunicorn, Postgres, and React on Kubernetes
1. Expose Flask and React to external users via an Ingress

## What is Container Orchestration?
Container orchestration is automated configuration, coordination, and management of Containers.
As you move from deploying containers on a single machine to deploying them across a number of machines, you'll need an orchestration tool to manage (and automate) the arrangement, coordination, and availability of the containers across the entire system.

Orchestration tools helps in:

1. Cross-server container communication
1. Horizontal scaling
1. Service discovery
1. Load balancing
1. Security/TLS
1. Zero-downtime deploys
1. Rollbacks
1. Logging
1. Monitoring

This is where [Kubernetes](https://kubernetes.io/) fits in along with a number of other orchestration tools  - like [Docker Swarm](https://docs.docker.com/engine/swarm/), [ECS](https://aws.amazon.com/ecs/) and [Mesos](http://mesos.apache.org/).

Which one should you use?

- use *Kubernetes* if you need to manage large, complex clusters
- use *Docker Swarm* if you are just getting started and/or need to manage small to medium-sized clusters
- use *ECS* if you're already using a number of AWS services

| Tool         | Pros                                          | Cons                                    |
|--------------|-----------------------------------------------|-----------------------------------------|
| Kubernetes   | large community, flexible, most features, hip | complex setup, high learning curve, hip |
| Docker Swarm | easy to set up, perfect for smaller clusters  | limited by the Docker API               |
| ECS          | fully-managed service, integrated with AWS    | vendor lock-in                          |

There's also a number of Kubernetes services offered by deifferent vendors

1. [Elastic Container Service](https://aws.amazon.com/eks/) (EKS)
1. [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) (GKE)
1. [Azure Kubernetes Service](https://azure.microsoft.com/en-us/services/kubernetes-service/) (AKS)

## Kubernetes
Kubernetes is a portable, extensible, open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation.
The name Kubernetes originates from Greek, meaning helmsman or pilot. Google open-sourced the Kubernetes project in 2014

## Kubernetes Architecture (Components of Kubernetes)
- The *master* node, which hosts the *Kubernetes Control Plane* that controls and managaes the whole Kubernetes system.
 It runs 3 processes: kube-apiserver, kube-controller-manager and kube-scheduler.
- Worker *nodes* that run the actual applications you deploy. It runs 2 processes: kubelet (which communicates with  the Kuberenetes master) and kube-proxy.


<img src="/images/k8s/kubernetes-components.png" style="max-width:80%;padding-bottom:20px;" alt="Kubernetes Components">
To knowmore about each component, refer: https://kubernetes.io/docs/concepts/overview/components/

## Kubernetes Concepts
To work with Kubernetes, you use *Kubernetes API objects* to describe your cluster’s desired state: what applications or other workloads you want to run, what container images they use, the number of replicas, what network and disk resources you want to make available, and more. You set your desired state by creating objects using the Kubernetes API, typically via the command-line interface, *kubectl*

The basic Kubernetes objects include:
1. **[Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/)** is a logical, tightly-coupled group of application containers that run on a Node. Containers in a Pod are deployed together and share resources (like data volumes and network addresses). Multiple Pods can run on a single Node.
1. **[Service](https://kubernetes.io/docs/concepts/services-networking/service/)** is a logical set of Pods that perform a similar function. It enables load balancing and service discovery. It's an abstraction layer over the Pods; Pods are meant to be ephemeral while services are much more persistent.
1. **[Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)** are used to persist data beyond the life of a container. They are especially important for stateful applications like Redis and Postgres.
    - A *[PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)* defines a storage volume independent of the normal Pod-lifecycle. It's managed outside of the particular Pod that it resides in.
    - A *[PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)* is a request to use the PersistentVolume by a user.
1. **[Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)**: Kubernetes supports multiple virtual clusters backed by the same physical cluster. These virtual clusters are called namespaces    
    
Kubernetes also contains higher-level abstractions that rely on controllers to build upon the basic objects, and provide additional functionality and convenience features. These include:

1. **[Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)** are used to describe the desired state of Kubernetes. They dictate how Pods are created, deployed, and replicated.
1. **[DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)** A DaemonSet ensures that all (or some) Nodes run a copy of a Pod.
1. **[StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)**: Manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods.
1. **[ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)**: 
Before diving in, let's look at some of the basic building blocks that you have to work with from the [Kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/): A ReplicaSet’s purpose is to maintain a stable set of replica Pods running at any given time
1. **[Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/)**: A Job creates one or more Pods and ensures that a specified number of them successfully terminate

Other objects/metadata for the objects include:
1. **[Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)** Labels are key/value pairs that are attached to objects, such as pods.
1. **[Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)** An API object that manages external access to the services in a cluster, typically HTTP. Ingress may provide load balancing, SSL termination and name-based virtual hosting

Now, It's time to look at the App deployment part, which is the main purpose of this blog.

## App Overview
**DevopsTree**: Flask + React + Postgres app (currently, it only displays the information. You cannot edit/update).
It displays major tools & technolgies used in Devops & it's dependencies in a Tree pattern.


<img src="/images/k8s/devopstree.gif" style="max-width:80%;padding-bottom:20px;" alt="DevopsTree App">


## Project Setup

Clone the Github repo: [DevopsTree-Kubernetes](https://github.com/vinaydhegde/DevopsTree-Kubernetes)

```sh
$ git clone https://github.com/vinaydhegde/DevopsTree-Kubernetes ~/DevopsTree-Kubernetes
```

Below is the code structure you see when you clone the repo :

```sh
├── README.md
├── k8s
│   ├── devopstree-secret.yml
│   ├── devopstree-deployment-postgres.yml
│   ├── devopstree-service-postgres.yml
│   ├── devopstree-minikube-ingress.yml
│   ├── devopstree-pv.yml
│   ├── devopstree-pvc.yml
│   ├── devopstree-deployment-flask.yml
│   ├── devopstree-service-flask.yml
│   ├── devopstree-deployment-react.yml
│   └── devopstree-service-react.yml
└── services
    ├── client
    │   ├── Dockerfile
    │   ├── Dockerfile-k8s
    │   ├── README.md
    │   ├── node_modules
    |   ├── public
    │   ├── package.json
    │   ├── src
    │   │   ├── App.js
    │   │   ├── index.js
    │   │   ├── utils.js
    │   │   ├── Tree
    │   │       ├── index.js
    ├── db
    │   ├── create_database.sql
    │   └── dockerfile
    └── server
        ├── dockerfile
        ├── entrypoint.sh
        ├── manage.py
        ├── project
        │   ├── __init__.py
        │   ├── api
        │   │   ├── __init__.py
        │   │   ├── devopstree.py
        │   │   └── models.py
        │   └── config.py
        └── requirements.txt
```

## Minikube

[Minikube](https://kubernetes.io/docs/setup/minikube/) is a tool which allows developers to use and run a Kubernetes cluster locally. It's a great way to quickly get a cluster up and running so you can start interacting with the Kubernetes API.
```sh
# For Ubuntu 
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

The command above is to install Minikube on Ubuntu. You can follow the official [quickstart](https://github.com/kubernetes/minikube#quickstart) guide to get Minikube installed for other OS platforms.

Along with this, you also need to install,

1. A [Hypervisor](https://kubernetes.io/docs/tasks/tools/install-minikube/#install-a-hypervisor) (like [VirtualBox](https://www.virtualbox.org/wiki/Downloads) or [HyperKit](https://github.com/moby/hyperkit)) to manage virtual machines

1. [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) to deploy and manage apps on Kubernetes
```sh
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
$ chmod +x ./kubectl
$ sudo mv ./kubectl /usr/local/bin/kubectl
$ kubectl version --client
```

Start Minikube & create a cluster
```sh
$ minikube start
```

Access Minikube [dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/):
```sh
$ minikube dashboard
```

<img src="/images/k8s/minikube-dashboard.png" style="max-width:80%;padding-bottom:20px;" alt="Minikube Dashboard">

> Note: Config files will be located in the *~/.kube* directory and all the virtual machine bits will be in the *~/.minikube* directory.

Now let's start creating objects via the Kubernetes API.

## Creating Objects

To create a new [object](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) in Kubernetes, you must provide a "spec" that describes its desired state.


Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devopstree-react
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: devopstree-react
    spec:
      containers:
      - name: devopstree-react
      - image: vinaydhegde/devopstree-react:latest
```

Required Fields:

1. `apiVersion` - [Kubernetes API](https://kubernetes.io/docs/reference/#api-reference) version
1. `kind` - the type of object you want to create
1. `metadata` - info about the object so that it can be uniquely identified
1. `spec` - desired state of the object

In the above example, this spec will create a new Deployment called *devopstree-react* with a single replica (Pod). Take note of the `containers` section. Here, we specified the Docker image to pull from the docker registry.

We are going to create the following objects in Kubernetes:

<img src="/images/k8s/objects.png" style="max-width:80%;padding-bottom:20px;" alt="objects">

## Volume: db-pv

As, containers are ephemeral, we need to configure a volume, via a [PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes) and a [PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims), to store the Postgres data outside of the Pod.

Take note of the YAML file in *k8s/devopstree-pv.yml*:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: db-pv
  labels:
    type: local
spec:
  capacity:
    storage: 2Gi
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/data/db-pv"
```

This configuration will create a [hostPath](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath) PersistentVolume at "/data/db-pv" within the Node. The size of the volume is 2 gibibytes with an access mode of [ReadWriteOnce](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes), which means that the volume can be mounted as read-write by a single node.

> Note: Kubernetes only supports using a hostPath on a single-node cluster.

Create the volume:

```sh
$ kubectl apply -f ./k8s/devopstree-pv.yml
```

View details:

```sh
$ kubectl get pv

NAME    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
db-pv   2Gi        RWO            Retain           Available           standard                5s
```

You can also see this object in the dashboard:

<img src="/images/k8s/pv.png" style="max-width:80%;padding-bottom:20px;" alt="peristent-volume">

*k8s/devopstree-pvc.yml*:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc
  labels:
    type: local
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  volumeName: db-pv
  storageClassName: standard
```

## Volume Claim: dp-pvc

```sh
$ kubectl apply -f ./k8s/devopstree-pvc.yml
```

View details:

```sh
$ kubectl get pvc

NAME     STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
db-pvc   Pending   db-pv    0                         standard       9s
```

<img src="/images/k8s/pvc.png" style="max-width:80%;padding-bottom:20px;" alt="peristent-volume-claim">

## Secrets: postgres-credentials

[Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) are used to handle sensitive info such as passwords, API tokens, and SSH keys. We'll set up a Secret to store our Postgres database credentials.

*k8s/devopstree-secret.yml*:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-credentials
type: Opaque
data:
  user: ZGV2b3BzdHJlZS1kZW1v
  password: ZGVtb0AxMjM=
```

The `user` and `password` fields are base64 encoded strings ([security via obscurity](https://en.wikipedia.org/wiki/Security_through_obscurity)):

For example:
```sh
$ echo -n "<your_password_to_encode>" | base64
eW91cl9wYXNzd29yZF90b19lbmNvZGU=
```

> Note: Any user with access to the cluster will be able to read the values in plaintext. Use `vault` if you want to encrypt secrets in transit and at rest.

Add the Secrets object:

```sh
$ kubectl apply -f ./k8s/devopstree-secret.yml
```

<img src="/images/k8s/secret.png" style="max-width:80%;padding-bottom:20px;" alt="Secret object">

## POSTGRES

## Deployment: devopstree-deployment-postgres

With the volume and database credentials set up in the cluster, we can now configure the Postgres database itself.

*k8s/devopstree-deployment-postgres.yml*:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    name: database
  name: devopstree-postgres
spec:
  progressDeadlineSeconds: 2147483647
  replicas: 1
  selector:
    matchLabels:
      service: devopstree-postgres
  template:
    metadata:
      creationTimestamp: null
      labels:
        service: devopstree-postgres
    spec:
      containers:
      - name: devopstree-postgres
        image: postgres:12.2-alpine
        env:
          - name: POSTGRES_USER
            valueFrom:
              secretKeyRef:
                name: postgres-credentials
                key: user
          - name: POSTGRES_PASSWORD
            valueFrom:
              secretKeyRef:
                name: postgres-credentials
                key: password
        volumeMounts:
        - mountPath: /var/lib/postgresql/data
          name: postgres-volume-mount
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - name: postgres-volume-mount
        persistentVolumeClaim:
          claimName: db-pvcc
```

Let's understand what the above configuration does:

1. `metadata`
    - The `name` field defines the Deployment name - `devopstree-postgres`
    - `labels` define the labels for the Deployment - `name: database`
1. `spec`
    - `replicas` define the number of Pods to run - `1`
    - `template`
        - `metadata`
            - `labels` indicate which labels should be assigned to the Pod - `service: devopstree-postgres`
        - `spec`
            - `containers` define the containers associated with each Pod
            - `volumes` define the volume claim - `postgres-volume-mount`
            - `restartPolicy` defines the [restart policy](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/) - `Always`

Here, the Pod name is also `devopstree-postgres` but it can be different than the service name (You will see name prefix *devopstree* in many places in almost all configuration files. So don't get confused). 
The docker image name is `postgres:12.2-alpine`, which will be pulled from Docker Hub. The database credentials, from the Secret object, are passed in as well.

Finally, when applied, the volume claim will be mounted into the Pod. The claim is mounted to "/var/lib/postgresql/data" - the default location - while the data will be stored in the PersistentVolume, "/data/db-pv".

Create the Deployment:

```sh
$ kubectl create -f ./k8s/devopstree-deployment-postgres.yml
```

Status:

```sh
$ kubectl get deployments

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
devopstree-postgres   0/1     1            0           17s
```

## Service: devopstree-service-postgres

*k8s/devopstree-service-postgres.yml*:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: devopstree-postgres
  labels:
    service: devopstree-postgres
spec:
  selector:
    service: devopstree-postgres
  type: ClusterIP
  ports:
  - port: 5432
```

Let's understand what the above configuration does:

1. `metadata`
    - The `name` field defines the Service name - `devopstree-postgres`
    - `labels` define the labels for the Service - `name: database`
1. `spec`
    - `selector` defines the Pod label and value that the Service applies to - `service: devopstree-postgres`
    - `type` defines the [type](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types) of Service - `ClusterIP`
    - `ports`
        - `port` defines the port exposed to the cluster


> Note: `selector` in the above Service configuration, refers to the Deployment we created earlier. Here, we have set the same name for both deployment & service. But it can be different.

Since the [Service type](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types) is `ClusterIP`, it's not exposed externally, so it's *only* accessible from within the cluster by other objects.

Create the service:

```sh
$ kubectl create -f ./k8s/devopstree-service-postgres.yml
```

Create a postgres database called`devopstree`, using the Pod name:

```sh
$ kubectl get pods

NAME                                   READY   STATUS    RESTARTS   AGE
devopstree-postgres-7bbcc445dc-lrzx9   1/1     Running   0          24m

$ kubectl exec devopstree-postgres-7bbcc445dc-lrzx9 --stdin --tty -- createdb -U devopstree-demo devopstree
```
>Note: Here, `devopstree-demo` is the user name which we login to postgres DB.
'devopstree' is the postgres DB name.

Verify the creation:

```sh
kubectl exec devopstree-postgres-7bbcc445dc-lrzx9 --stdin --tty -- psql -U devopstree-demo devopstree

psql (12.2)
Type "help" for help.

postgress=# \l
                                                List of databases
      Name       |      Owner      | Encoding |  Collate   |   Ctype    |            Access privileges            
-----------------+-----------------+----------+------------+------------+-----------------------------------------
 devopstree      | devopstree-demo | UTF8     | en_US.utf8 | en_US.utf8 | 
 devopstree-demo | devopstree-demo | UTF8     | en_US.utf8 | en_US.utf8 | 
 postgres        | devopstree-demo | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0       | devopstree-demo | UTF8     | en_US.utf8 | en_US.utf8 | =c/"devopstree-demo"                   +
                 |                 |          |            |            | "devopstree-demo"=CTc/"devopstree-demo"
 template1       | devopstree-demo | UTF8     | en_US.utf8 | en_US.utf8 | =c/"devopstree-demo"                   +
                 |                 |          |            |            | "devopstree-demo"=CTc/"devopstree-demo"
(5 rows)

postgres=# \q
```
> You can also get the Pod name using the command:
>
> ```sh
> $ kubectl get pod -l service=devopstree-postgres -o jsonpath="{.items[0].metadata.name}"
> ```
>
> Assign the value to a variable and then create the database:
> ```sh
> $ POD_NAME=$(kubectl get pod -l service=devopstree-postgres -o jsonpath="{.items[0].metadata.name}")
> $ kubectl exec $POD_NAME --stdin --tty -- createdb -U devopstree-demo devopstree
> ```

```sh
$ kubectl get service

NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
devopstree-postgres   ClusterIP   10.100.166.248   <none>        5432/TCP   14m
kubernetes            ClusterIP   10.96.0.1        <none>        443/TCP    151m

$ kubectl get deployments

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
devopstree-postgres   1/1     1            1           40m
```


## Flask

## Deployment: devopstree-deployment-flask

Take a moment to review the Flask project structure along with the *dockerfile* and the *entrypoint.sh* files:

1. "services/server"
1. *services/server/dockerfile*
1. *services/server/entrypoint.sh*

*k8s/devopstree-deployment-flask.yml*:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    name: devopstree-flask
  name: devopstree-flask
spec:
  progressDeadlineSeconds: 2147483647
  replicas: 1
  selector:
    matchLabels:
      app: devopstree-flask
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: devopstree-flask
    spec:
      containers:
      - env:
        - name: FLASK_ENV
          value: development
        - name: APP_SETTINGS
          value: project.config.DevelopmentConfig
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              key: user
              name: postgres-credentials
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              key: password
              name: postgres-credentials
        image: vinaydhegde/devopstree-flask:latest
        imagePullPolicy: Always
        name: flask
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
```

This is similar to the Postgres Deployment spec. The major difference here is that, we are using my pre-built & pre-pushed image on [Docker Hub](https://hub.docker.com/), `vinaydhegde/flask-kubernetes`, or build and push your own.

For example:

```sh
$ docker build -t <YOUR_DOCKER_HUB_NAME>/devopstree-flask ./services/server
$ docker push <YOUR_DOCKER_HUB_NAME>/devopstree-flask
```

If you use your own, make sure you replace `vinaydhegde` with your Docker Hub name in *k8s/devopstree-deployment-flask.yml* as well.

> Note: if you don't want to push the image to a Docker registry, after you build the image locally, you can set the `image-pull-policy` flag to `Never` to always use the local image.

Create the Deployment:

```sh
$ kubectl create -f ./k8s/devopstree-deployment-flask.yml
```


<img src="/images/k8s/flask-deployment.png" style="max-width:80%;padding-top:20px;padding-bottom:20px;" alt="falsk deployment">

This will immediately spin up a new Pod:

<img src="/images/k8s/flask-pod.png" style="max-width:80%;padding-top:20px;padding-bottom:20px;" alt="falsk deployment">

## Service: devopstree-service-flask

*kubernetes/devopstree-service-flask.yml*:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: devopstree-flask
  labels:
    service: devopstree-flask
spec:
  selector:
    app: devopstree-flask
  ports:
  - port: 5000
    targetPort: 5000
```

> To understand about the `targetPort` and how it relates to the `port`, refer [Services](https://kubernetes.io/docs/concepts/services-networking/service/#defining-a-service) guide.

Create the service:

```sh
$ kubectl create -f ./k8s/devopstree-service-flask.yml
```

Now, check whether the Pod you created through deployment is associated with the Service:

<img src="/images/k8s/flask-service-with-pod.png" style="max-width:80%;padding-top:20px;padding-bottom:20px;" alt="service & pod">

Apply the migrations and bootsrap the database:

```sh
$ kubectl get pods

NAME                                   READY   STATUS    RESTARTS   AGE
devopstree-flask-64988ffb68-8v82n      1/1     Running   0          12m
devopstree-postgres-7bbcc445dc-lrzx9   1/1     Running   0          62m
```

```h
$ kubectl exec devopstree-flask-64988ffb68-8v82n --stdin --tty -- python manage.py recreate_db
$ kubectl exec devopstree-flask-64988ffb68-8v82n --stdin --tty -- python manage.py bootstrap_db
```

Verify:

```sh
$ kubectl exec devopstree-postgres-7bbcc445dc-lrzx9 --stdin --tty -- psql -U devopstree-demo

devopstree-demo=# \c devopstree
You are now connected to database "devopstree" as user "devopstree-demo".
devopstree=# select * from devopstree;
```
>Note: Here, both database & table name is: devopstree


## Ingress: minikube-ingress

To enable traffic to access the Flask API inside the cluster, you can use either a NodePort, LoadBalancer, or Ingress:

1. A [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#nodeport) exposes a Service on an open port on the Node.
1. As the name implies, a [LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer) creates an external load balancer that points to a Service in the cluster.
1. Unlike the previous two methods, an [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) is not a type of Service; instead, it sits on top of the Services as an entry point into the cluster.

> For more, review the official [Publishing Services](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types) guide.

*k8s/devopstree-minikube-ingress.yml*:

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: minikube-ingress
  annotations:
spec:
  rules:
  - host: devops-tree
    http:
      paths:
      - path: /
        backend:
          serviceName: devopstree-react
          servicePort: 8080
      - path: /devopstree
        backend:
          serviceName: devopstree-flask
          servicePort: 5000
```

Here, we defined the following HTTP rules:

1. `/` - routes requests to the React Service (which we are going to setup in the coming steps)
1. `/devopstree` - routes requests to the Flask Service

Enable the Ingress [addon](https://github.com/kubernetes/minikube/tree/master/deploy/addons/ingress):

```sh
$ minikube addons enable ingress
```

Create the Ingress object: 'minikube-ingress'

```sh
$ kubectl apply -f ./k8s/devopstree-minikube-ingress.yml
```

Update the */etc/hosts* file to route requests from the host we defined, `devops-tree`, to the Minikube instance.

Add an entry to */etc/hosts*:

```sh
$ echo "$(minikube ip) devops-tree" | sudo tee -a /etc/hosts
```

Try it out:

1. http://devops-tree/devopstree

It should list the JSON elements.


## React

## Deployment: devopstree-deployment-react

Review the `React` project along with the associated Dockerfiles:

1. "services/client"
1. */services/client/Dockerfile*
1. */services/client/Dockerfile-k8s*

*k8s/devopstree-deployment-react.yml*:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    name: devopstree-react
  name: devopstree-react
spec:
  progressDeadlineSeconds: 2147483647
  replicas: 1
  selector:
    matchLabels:
      app: devopstree-react
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: devopstree-react
    spec:
      containers:
      - image: vinaydhegde/devopstree-react:latest
        imagePullPolicy: Always
        name: devopstree-react
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
```

Again, you can either use my docker image or build your own & push it to Docker Hub:

```sh
$ docker build -t <YOUR_DOCKERHUB_NAME>/devopstree-react ./services/client \
    -f ./services/client/Dockerfile-k8s
$ docker push <YOUR_DOCKERHUB_NAME>/devopstree-react
```

Create the Deployment:

```sh
$ kubectl create -f ./k8s/devopstree-deployment-react.yml
```

Check whether a Pod is created along with the Deployment:

```sh
$ kubectl get deployments devopstree-react

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
devopstree-react   1/1     1            1           5m21s

$ kubectl get pods

NAME                                   READY   STATUS    RESTARTS   AGE
devopstree-flask-64988ffb68-8v82n      1/1     Running   0          48m
devopstree-postgres-7bbcc445dc-lrzx9   1/1     Running   0          98m
devopstree-react-6c8b7c8cc6-t2xsn      1/1     Running   0          5m57s
```

> Verify, Pod and Deployment were created successfully in the dashboard.

<img src="/images/k8s/react-deployment.png" style="max-width:80%;padding-top:20px;padding-bottom:20px;" alt="react deployment">

<img src="/images/k8s/react-pod.png" style="max-width:80%;padding-top:20px;padding-bottom:20px;" alt="react pod">


## Service: devopstree-service-react

*k8s/devopstree-service-react.yml*:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: devopstree-react
  labels:
    service: devopstree-react
  name: devopstree-react
spec:
  selector:
    app: devopstree-react
  ports:
  - port: 8080
    targetPort: 8080
```

Create the service:

```sh
$ kubectl create -f ./k8s/devopstree-service-react.yml
```
<img src="/images/k8s/react-service.png" style="max-width:80%;padding-top:20px;padding-bottom:20px;" alt="react service">


Yes.. finally! we are DONE.

You can access the app at: [http://devops-tree/](http://devops-tree)

<img src="/images/k8s/devopstree.png" style="max-width:80%;padding-top:20px;padding-bottom:20px;" alt="DevopsTree app">


## Scaling

Kubernetes makes it easy to scale, adding additional Pods as necessary, when the traffic load becomes too much for a single Pod to handle.

For example, let's add another Flask Pod to the cluster:

```sh
$ kubectl scale deployment devopstree-flask --replicas=2
```

Confirm:

```sh
$ kubectl get deployments devopstree-flask

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
devopstree-flask   2/2     2            2           62m
```

```sh
$ kubectl get pods -o wide

NAME                                   READY   STATUS    RESTARTS   AGE     IP            NODE       
devopstree-flask-64988ffb68-8v82n      1/1     Running   0          63m     172.18.0.7    minikube
devopstree-flask-64988ffb68-j5drv      1/1     Running   0          2m25s   172.18.0.10   minikube
devopstree-postgres-7bbcc445dc-lrzx9   1/1     Running   0          112m    172.18.0.6    minikube
devopstree-react-6c8b7c8cc6-t2xsn      1/1     Running   0          20m     172.18.0.9    minikube
```

## Helpful Commands

| Command                     | Explanation                                          |
|-----------------------------|------------------------------------------------------|
| `minikube get-k8s-versions` | Lists the supported Kubernetes versions              |
| `minikube start`            | Starts a local Kubernetes cluster                    |
| `minikube ip`               | Displays the IP address of the cluster               |
| `minikube dashboard`        | Opens the Kubernetes dashboard in your browser       |
| `kubectl version`           | Displays the Kubectl version                         |
| `kubectl cluster-info`      | Displays the cluster info                            |
| `kubectl get nodes`         | Lists the Nodes                                      |
| `kubectl get pods`          | Lists the Pods                                       |
| `kubectl get deployments`   | Lists the Deployments                                |
| `kubectl get services`      | Lists the Services                                   |
| `minikube stop`             | Stops a local Kubernetes cluster                     |
| `minikube delete`           | Removes a local Kubernetes cluster                   |



