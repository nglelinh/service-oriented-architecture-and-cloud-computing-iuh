# Lecture_6.2_Docker_kukenetes


## Slide 1

### Containerization


## Slide 2

### Popular Container Frameworks


Most popular, Full-featured
Requires root
Windows, macOS, and Linux


Less popular, More limited
Daemonless and rootless
Windows, macOS, and Linux


Popular on HPC Clusters
Only for Linux
(formerly “Singularity”)


## Slide 3

### Dockerfile


Contains layers of instructions for configuring the system and installing libraries and files.

Specifies a base layer, which is useful for “picking up where someone left off”


## Slide 4

### Dockerfile Layers


Docker uses layers to save on image space and on build complexity.

Each layer contains the information that changes the layer before it.

When you rebuild, Docker will cache the unchanged layers.


## Slide 9

### Docker Commands


Pull an Image from a Container Registry
docker pull <url>/<user>/<image>:<tag>
docker pull nvidia/cuda:12.6.1-cudnn-devel-ubuntu22.04
docker pull nvcr.io/nvidia/pytorch:22.04-py3


Build an Image Locally
docker build -t <image tag> .
docker build -t reactwebapp .


Run a Container Locally
docker run <image name> <other options>
docker run -p 8888:8888 -v /data/local:/home/jovyan/work jupyter/base-notebook
docker run --name cuda_gpu_1 --gpus all -t nvidia/cuda


Execute Command in a Container
docker run/exec <image name> <commands>
docker run --name mycontainer1 -it mycontainer /bin/bash
docker exec -it mycontainer /bin/bash


Get started | Docker Docs


## Slide 10

### Volumes (and sharing other resources)


Host Machine


Docker System


Filesystem


Docker Area


Memory


CPUs


GPUs


Container


bind mount


volume


tmpfs
mount


Network


veth


nvidia
container toolkit


(shared with host)


## Slide 11

### Docker Compose


Docker Compose allows you to specify multiple images that can communicate with one another.
Networking (vNets)
Volumes (databases)

docker compose up -d
docker compose down

Less used in cloud native environments where PaaS options are more common.


services:
  backend:
    build: backend
    ports:
      - 80:80
      - 9229:9229
      - 9230:9230
    ...
  db:
    image: mariadb:10.6.4-focal
    ...
  frontend:
    build: frontend
    ports:
    - 3000:3000
    ...


Directory Structure


compose.yaml File


## Slide 12

### Container Storage


Azure Container Registry


DockerHub


GitHub Packages


## Slide 13

### Container Registry Commands


Login to the Container Registry
az acr login --name <registry name>
az acr login --name crdsba6190deveastus001


Tag the Image with the URL, your name, and tag
docker tag <image name> <registry name>.azurecr.io/<image name>:<tag>
docker tag instructor_sklearn crdsba6190deveastus001.azurecr.io/instructor_sklearn:latest


Push the Image to the Container Registry
docker push <registry name>.azurecr.io/<image name>:<tag>
docker push crdsba6190deveastus001.azurecr.io/instructor_sklearn:latest


## Slide 14

### What is / Why Kubernetes?


## Slide 15

### Orchestrator


manage and organize both hosts and docker containers running on a cluster

main issue - resource allocation - where can a container be scheduled, to fulfill its requirements (CPU/RAM/disk) + how to keep track of nodes and scale


## Slide 16

### Some orchestrator tasks


manage networking and access
track state of containers
scale services
do load balancing
relocation in case of unresponsive host
service discovery
attribute storage to containers
...


## Slide 17

### Orchestrator options


Kubernetes – open-source, product of CNCF
Apache Mesos – cluster management tool, with container orchestration being only one of the things it can do, originally through a plugin called Marathon
Docker Swarm – integrated in docker container platform


## Slide 18

### What is Kubernetes?


Learn more: https://kubernetes.io/


## Slide 19

### What Does “Kubernetes” Mean?


Greek for “pilot” or “Helmsman of a ship”


## Slide 20

### Self Healing


Kubernetes will ALWAYS try and steer the cluster to its desired state.

Me: “I want 3 healthy instances of redis to always be running.”
Kubernetes: “Okay, I’ll ensure there are always 3 instances up and running.”
Kubernetes: “Oh look, one has died. I’m going to attempt to spin up a new one.”


## Slide 21

### Architecture


## Slide 25

### Core Objects


## Slide 26

### Namespaces


Namespaces are a logical cluster or environment, and are the primary method of partitioning a cluster or scoping access.


apiVersion: v1 kind: Namespace metadata:
name: prod labels:
app: MyBigWebApp


$ kubectl get ns --show-labels NAME      STATUS   AGE
default     Active   11h
kube-public  Active   11h kube-system  Active   11h prod      Active   6s


LABELS
<none>
<none>
<none> app=MyBigWebApp


## Slide 27

### Kubernetes and containers


Can you deploy a container in Kubernetes? NO (not directly)

Why not? Because the smallest deployable unit of computing is not a container, but ...


## Slide 28

### Pod


smallest deployable unit of computing in Kubernetes

colocated multiple apps(containers) into a single atomic unit, scheduled onto a single machine. 
Networking: All containers in the same Pod share the same IP address and can communicate with each other via localhost. They also share the same network ports.

Storage: They can share data using volumes mounted to the Pod.

upon creation, statically allocated to a certain node


## Slide 29

### Pod


each container runs in its own cgroup (CPU + RAM allocation), but they share some namespaces and filesystems, such as:

IP address and port space

same hostname

IPC channels for communication


## Slide 30

### Example Pod Specification


Specifies the name of the pod, its image, and other information.
Volumes
Commands
Secrets
Resource Requests
CPUs
GPUs
Memory
Tolerations


apiVersion: v1
kind: Pod
metadata:
  name: instructor-test-01
spec:
  restartPolicy: Never
  containers:
  - name: instructor-sklearn
    image: crdsba6190deveastus001.azurecr.io/instructor_sklearn:latest
    volumeMounts:
      - name: datalake
        mountPath: "/mnt/datalake/"
        readOnly: false
    # command: ["/bin/bash", "-c"]
    # args: ["./run.py"]
    command: [ "/bin/bash", "-c", "--" ]
    args: [ "while true; do sleep 30; done;" ]
    # resources:
    #   limits:
    #     memory: "2Gi"
    #     cpu: "200m"
  imagePullSecrets:
    - name: acr-secret
  volumes:
    - name: datalake
      persistentVolumeClaim:
        claimName: pvc-datalake-class-blob


## Slide 31

### Pod Template


Workload Controllers manage instances of Pods based off a provided template.
Pod Templates are Pod specs with limited metadata.


Controllers use Pod Templates to make actual pods.


apiVersion: v1 kind: Pod metadata:
name: pod-example labels:
app: nginx
spec:


template: metadata:
labels:
app: nginx spec:
containers:
- name: nginx image: nginx


## Slide 32

### So, why a pod and not container directly?


all or nothing approach for a group of symbiotic containers, that need to be kept together at all times

pod considered running if all containers are scheduled and running

Each Pod represents a single logical application.

Can you deploy a container in Kubernetes? Yes, inside a pod!


## Slide 33

### When to have multiple containers inside a pod?


when it's impossible for them to work on different machines (sharing local filesystem or using IPC)

when one of them facilitates communication with the other without altering it (adapter)

when one of them offers support for the other (logging/monitoring)

when one of them configures the other


## Slide 34

### Pod scheduling


scheduler tries to scatter replicas for reliability and are never moved (immutability)

what happens when physical node dies? pod needs to be deleted in order to be rescheduled


## Slide 35

### Pod health checks


process health check - main process is always running and has not exited (for each container)

liveness probe - application specific, determines if application actually does what the probe knows it should do

readiness probe - on start, it might take a while until the application fully loads and can process requests as expected


## Slide 36

### ConfigMaps and Secrets


configure pods, make images more reusable

live update on change (application needs to be able to reload)

secrets mounted as ram disk => not written to actual filesystem


## Slide 37

### Labels and annotations


labels - key/value pairs attached to objects arbitrarily, providing foundation for grouping objects

annotations - key/value pairs designed to hold non-identifying information that can be leveraged by tools and libraries

metadata needed by the system to provide identification, grouping and higher-level features


## Slide 38

### Labels


key-value pairs that are used to identify, describe and group together related sets of objects or resources.
NOT characteristic of uniqueness.
Have a strict syntax with a slightly limited character set*.


*  https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#syntax-and-character-set


## Slide 39

### Selectors use labels to filter or select objects, and are used throughout Kubernetes.


apiVersion: v1 kind: Pod metadata:
name: pod-label-example labels:
app: nginx
env: prod spec:
containers:
- name: nginx
image: nginx:stable-alpine
ports:
- containerPort: 80 nodeSelector:
gpu: nvidia


## Slide 40

### apiVersion: v1 kind: Pod metadata:
name: pod-label-example labels:
app: nginx env: prod
spec:
containers:
- name: nginx
image: nginx:stable-alpine ports:
- containerPort: 80 nodeSelector:
gpu: nvidia


Selector Example


## Slide 41

### Equality based selectors allow for simple filtering (=,==, or !=).


Set-based selectors are supported on a limited subset of objects.
However, they provide a method of filtering on a set of values, and supports multiple operators including: in, notin, and exist.


selector: matchExpressions:
- key: gpu operator: in values: [“nvidia”]


selector: matchLabels:
gpu: nvidia


## Slide 42

### Network Communication


## Slide 43

### Communication challenges


between pods - using hardcoded IPs would be the wrong way to do it, as pods might be rescheduled on different nodes and change IPs

from outside - keep track of all pods that provide a certain service and loadbalance between them


## Slide 44

### Service discovery


find which processes are listening at which addresses for which services

do it quickly and reliably, with low-latency, storing richer definitions of what those services are

public DNS isn't dynamic enough to deal with the amount of updates


## Slide 45

### Service


Abstraction which defines a logical set of Pods (selected using label selector), that provide the same functionality (same microservice)

Different types, for different types of exposure provided by the service

Durable resource
static cluster IP
static namespaced DNS name
<service name>.<namespace>. svc.cluster.local


## Slide 46

### Services


Target Pods using equality based selectors.
Uses kube-proxy to provide simple load-balancing.
kube-proxy acts as a daemon that creates local entries in the host’s iptables for every service.


## Slide 47

### Service Types


There are 4 major service types:
ClusterIP (default)
NodePort
LoadBalancer
ExternalName


## Slide 48

### ClusterIP Service


used for intra-cluster communication

special IP that will load-balance across all of the pods identified by service selector (which is a label selector)

allocated on create and cannot be changed until deletion of service, irrespective of number of pods


## Slide 49

### ClusterIP Service


internal Kubernetes DNS service allows service name to be used to access pods - 
my-svc.my-namespace.svc.cluster.local

if using readiness probes, only ready pods will be loadbalanced


## Slide 50

### Cluster IP Service


Name: Selector: Type:
IP:
Port: TargetPort:
Endpoints:


example-prod app=nginx,env=prod ClusterIP 10.96.28.176
<unset>	80/TCP 80/TCP
10.255.16.3:80,
10.255.16.4:80


/ # nslookup example-prod.default.svc.cluster.local

Name:	example-prod.default.svc.cluster.local
Address 1: 10.96.28.176 example-prod.default.svc.cluster.local


## Slide 51

### ClusterIP Service Without Selector


## Slide 52

### NodePorts Service


NodePort services extend the ClusterIP service.

Used to access pod from outside of cluster, exposes a port on every node’s IP

System picks a port and on all cluster nodes, traffic on that port is sent to service to loadbalance to pods

Port can either be statically defined, or dynamically taken from a range between 30000- 32767.


## Slide 53

### NodePort Service


example-prod app=nginx,env=prod NodePort 10.96.28.176
<unset>	80/TCP 80/TCP


Name: Selector: Type:
IP:
Port: TargetPort:
NodePort: Endpoints:


<unset>	32410/TCP 10.255.16.3:80,
10.255.16.4:80


## Slide 54

### LoadBalancer Service


LoadBalancer services extend NodePort.

implements the needed external support, to make access to pods easier in cloud environments

cloud providers provide support inside Kubernetes, to provision whatever is needed inside their environment to access service directly

Ex: IP gets allocated in Google, URL in AWS


## Slide 55

### LoadBalancer Service


example-prod app=nginx,env=prod LoadBalancer 10.96.28.176

172.17.18.43
<unset>	80/TCP 80/TCP


<unset>	32410/TCP


Name: Selector: Type:
IP:
LoadBalancer Ingress:
Port: TargetPort: NodePort: Endpoints:


10.255.16.3:80,
10.255.16.4:80


## Slide 56

### Fundamental Networking Rules


All containers within a pod can communicate with each other unimpeded.
All Pods can communicate with all other Pods without NAT.
All nodes can communicate with all Pods (and vice- versa) without NAT.
The IP that a Pod sees itself as is the same IP that others see it as.


## Slide 57

### Fundamentals Applied


Container-to-Container
Containers within a pod exist within the same network namespace and share an IP.
Enables intrapod communication over localhost.
Pod-to-Pod
Allocated cluster unique IP for the duration of its life cycle.
Pods themselves are fundamentally ephemeral.


## Slide 58

### Fundamentals Applied


Pod-to-Service
managed by kube-proxy and given a persistent cluster unique IP
exists beyond a Pod’s lifecycle.
External-to-Service
Handled by kube-proxy.
Works in cooperation with a cloud provider or other external entity (load balancer).


## Slide 59

### Coordination


## Slide 60

### Managing multiple pods


even though the pod is the building computing block for Kubernetes, working directly with Pods is tedious

Kubernetes abstracts different needs, to make it easier


## Slide 61

### Reconciliation loop


start with a desired state

observe current state and take action to try to make it match the desired one

goal-driven, self-healing system


## Slide 62

### ReplicaSet


makes sure a given number of identical pods are up at any time

does not "own" the pods it manages - selected with labels

can adopt existing pods with those labels

quarantine containers - remove label!

by default, on delete it deletes the pods, but can be set not to (--cascade=false)


## Slide 63

### ReplicaSet


replicas: The desired number of instances of the Pod.
selector:The label selector for the ReplicaSet will manage ALL Pod instances that it targets; whether it’s desired or not.


apiVersion: apps/v1 kind: ReplicaSet metadata:
name: rs-example spec:
replicas: 3
selector: matchLabels:
app: nginx env: prod
template:
<pod template>


## Slide 64

### ReplicaSet


$ kubectl describe rs rs-example Name:	rs-example Namespace:	default
Selector:	app=nginx,env=prod Labels:	app=nginx
env=prod Annotations: <none>
Replicas:	3 current / 3 desired
Pods Status: 3 Running / 0 Waiting / 0 Succeeded / 0 Failed Pod Template:
Labels: app=nginx
env=prod Containers:
nginx:
Image:	nginx:stable-alpine
Port:     80/TCP Environment: <none> Mounts:    <none>
Volumes:	<none> Events:
Type	Reason	Age	From	Message
----	------	---- ----	-------
Normal SuccessfulCreate 16s	replicaset-controller Created pod: rs-example-mkll2
Normal SuccessfulCreate 16s	replicaset-controller Created pod: rs-example-b7bcg Normal SuccessfulCreate 16s	replicaset-controller Created pod: rs-example-9l4dt


apiVersion: apps/v1 kind: ReplicaSet metadata:
name: rs-example spec:
replicas: 3
selector: matchLabels:
app: nginx env: prod
template:
metadata: labels:
app: nginx env: prod
spec:
containers:
- name: nginx
image: nginx:stable-alpine ports:
- containerPort: 80


$ kubectl get pods
NAME	READY
rs-example-9l4dt  1/1 rs-example-b7bcg  1/1 rs-example-mkll2  1/1


STATUS	RESTARTS	AGE
Running	0	1h
Running	0	1h
Running	0	1h


## Slide 65

### DaemonSet


makes sure a pod is executed on each physical node

usual usage - agent, logging, monitoring

can select nodes using node labels and node selector

can perform rollingUpdate on pods


## Slide 66

### Jobs


with ReplicaSets and DaemonSets, if pod's process exit (even with 0), it gets restarted, to keep consistency

Job manages Pods that need to run and exit with 0 (restarted until they do so)

configured with number of allowed parallel pods and number of expected completions


## Slide 67

### Job patterns


one shot - pods = 1, completion = 1

parallelism - pods >= 1, completion >= 1 (nr. pods will never be > completion)

work queue - pods >= 1, completion not set (pods will run until they all finish, no new ones created after one finishes)


## Slide 68

### Application versioning


## Slide 69

### Application versioning


there comes a time when a new version needs to be released

usually with no service downtime

check new version works before going through with the full release

Kubernetes has an abstraction for this!


## Slide 70

### Deployment


manages replica set through time and versions for pod spec

scale != version update

using health checks, makes sure a new version works

allows rollbacks to older versions (keeps track of changes)


## Slide 71

### Deployment


revisionHistoryLimit: The number of previous iterations of the Deployment to retain.
strategy: Describes the method of updating the Pods based on the type. Valid options are Recreate or RollingUpdate.
Recreate: All existing Pods are killed before the new ones are created.
RollingUpdate: Cycles through updating the Pods according to the parameters: maxSurge and maxUnavailable.


apiVersion: apps/v1 kind: Deployment metadata:
name: deploy-example spec:
replicas: 3
revisionHistoryLimit: 3 selector:
matchLabels: app: nginx env: prod
strategy:
type: RollingUpdate
rollingUpdate: maxSurge: 1
maxUnavailable: 0
template:
<pod template>


## Slide 72

### Deployment


$ kubectl create deployment test --image=nginx
$ kubectl set image deployment test nginx=nginx:1.9.1 --record

$ kubectl rollout history deployment test deployments "test"
REVISION	CHANGE-CAUSE


1
2


<none>
kubectl set image deployment test nginx=nginx:1.9.1 --record=true


$ kubectl annotate deployment test kubernetes.io/change-cause="image updated to 1.9.1"

$ kubectl rollout undo deployment test
$ kubectl rollout undo deployment test --to-revision=2

$ kubectl rollout history deployment test deployments "test"
REVISION	CHANGE-CAUSE


2
3


kubectl set image deployment test nginx=nginx:1.9.1 --record=true
<none>


kubectl scale deployment test --replicas=10 kubectl rollout pause deployment test kubectl rollout resume deployment test


## Slide 73

### Deployment strategies - recreate


all previous pods are destroyed and new pods are created

quickest

downtime while new pods start

in case of problems and rollback, even more downtime


## Slide 74

### Deployment strategies – rolling update


configured with max unavailable and max surge

max unavailable = number of pods that can be doing updates/rollbacks at a time, from the number of replicas

max surge = number of additional pods to be used for update/rollback


## Slide 75

### RollingUpdate Deployment


$ kubectl get replicaset
NAME	DESIRED |  | CURRENT | READY | AGE
mydep-6766777fff | 3 | 3 | 3 | 5h
 |  |  |  | 
$ kubectl get pods NAME | READY | STATUS | RESTARTS | AGE
mydep-6766777fff-9r2zn | 1/1 | Running | 0 | 5h
mydep-6766777fff-hsfz9 | 1/1 | Running | 0 | 5h
mydep-6766777fff-sjxhf | 1/1 | Running | 0 | 5h


Updating pod template generates a new ReplicaSet revision.


R1 pod-template-hash:
676677fff
R2 pod-template-hash:
54f7ff7d6d


## Slide 76

### RollingUpdate Deployment


$ kubectl get replicaset
NAME	DESIRED |  | CURRENT | READY | AGE
mydep-54f7ff7d6d | 1 | 1 | 1 | 5s
mydep-6766777fff | 2 | 3 | 3 | 5h
 |  |  |  | 
$ kubectl get pods NAME | READY | STATUS | RESTARTS | AGE
mydep-54f7ff7d6d-9gvll | 1/1 | Running | 0 | 2s
mydep-6766777fff-9r2zn | 1/1 | Running | 0 | 5h
mydep-6766777fff-hsfz9 | 1/1 | Running | 0 | 5h
mydep-6766777fff-sjxhf | 1/1 | Running | 0 | 5h


New ReplicaSet is initially scaled up based on maxSurge.


R1 pod-template-hash:
676677fff
R2 pod-template-hash:
54f7ff7d6d


## Slide 77

### RollingUpdate Deployment


$ kubectl get replicaset
NAME	DESIRED |  | CURRENT | READY | AGE
mydep-54f7ff7d6d | 2 | 2 | 2 | 8s
mydep-6766777fff | 2 | 2 | 2 | 5h
 |  |  |  | 
$ kubectl get pods NAME | READY | STATUS | RESTARTS | AGE
mydep-54f7ff7d6d-9gvll | 1/1 | Running | 0 | 5s
mydep-54f7ff7d6d-cqvlq | 1/1 | Running | 0 | 2s
mydep-6766777fff-9r2zn | 1/1 | Running | 0 | 5h
mydep-6766777fff-hsfz9 | 1/1 | Running | 0 | 5h


Phase out of old Pods managed by
maxSurge and maxUnavailable.


R1 pod-template-hash:
676677fff
R2 pod-template-hash:
54f7ff7d6d


## Slide 78

### RollingUpdate Deployment


$ kubectl get replicaset
NAME	DESIRED |  | CURRENT | READY | AGE
mydep-54f7ff7d6d | 3 | 3 | 3 | 10s
mydep-6766777fff | 0 | 1 | 1 | 5h
 |  |  |  | 
$ kubectl get pods NAME | READY | STATUS | RESTARTS | AGE
mydep-54f7ff7d6d-9gvll | 1/1 | Running | 0 | 7s
mydep-54f7ff7d6d-cqvlq | 1/1 | Running | 0 | 5s
mydep-54f7ff7d6d-gccr6 | 1/1 | Running | 0 | 2s
mydep-6766777fff-9r2zn | 1/1 | Running | 0 | 5h


Phase out of old Pods managed by
maxSurge and maxUnavailable.


R1 pod-template-hash:
676677fff
R2 pod-template-hash:
54f7ff7d6d


## Slide 79

### RollingUpdate Deployment


$ kubectl get replicaset
NAME	DESIRED |  | CURRENT | READY | AGE
mydep-54f7ff7d6d | 3 | 3 | 3 | 13s
mydep-6766777fff | 0 | 0 | 0 | 5h
 |  |  |  | 
$ kubectl get pods NAME | READY | STATUS | RESTARTS | AGE
mydep-54f7ff7d6d-9gvll | 1/1 | Running | 0 | 10s
mydep-54f7ff7d6d-cqvlq | 1/1 | Running | 0 | 8s
mydep-54f7ff7d6d-gccr6 | 1/1 | Running | 0 | 5s


Phase out of old Pods managed by
maxSurge and maxUnavailable.


R1 pod-template-hash:
676677fff
R2 pod-template-hash:
54f7ff7d6d


## Slide 80

### RollingUpdate Deployment


$ kubectl get replicaset
NAME	DESIRED |  | CURRENT | READY | AGE
mydep-54f7ff7d6d | 3 | 3 | 3 | 15s
mydep-6766777fff | 0 | 0 | 0 | 5h
 |  |  |  | 
$ kubectl get pods NAME | READY | STATUS | RESTARTS | AGE
mydep-54f7ff7d6d-9gvll | 1/1 | Running | 0 | 12s
mydep-54f7ff7d6d-cqvlq | 1/1 | Running | 0 | 10s
mydep-54f7ff7d6d-gccr6 | 1/1 | Running | 0 | 7s


Updated to new deployment revision completed.


R1 pod-template-hash:
676677fff
R2 pod-template-hash:
54f7ff7d6d


## Slide 81

### Horizontal Pod Autoscaling


built-in feature

automatically shrink/increase based on certain parameters

works with heapster pod, that gathers information from containers 

works on top of replicaSets as well as Deployments


## Slide 82

### Stateful


## Slide 83

### stateless -> stateFULL


not all applications are stateless (most aren't)

state = unique pod identity (not interchange-able anymore) + persistence of data (even when rescheduled on different nodes)

Kubernetes comes to the rescue again!


## Slide 84

### StatefulSet


each replica gets a persistent hostname with unique id

created in order of index low -> high

delete in order of index high -> low

usage example - database


## Slide 85

### StatefulSet


apiVersion: apps/v1 kind: StatefulSet metadata:
name: sts-example spec:
replicas: 2
revisionHistoryLimit: 3 selector:
matchLabels:
app: stateful serviceName: app
updateStrategy:
type: RollingUpdate rollingUpdate:
partition: 0
template:
metadata: labels:
app: stateful
<continued>


<continued>
spec:
containers:
name: nginx
image: nginx:stable-alpine ports:
containerPort: 80
volumeMounts:
name: www
mountPath: /usr/share/nginx/html volumeClaimTemplates:
- metadata:
name: www spec:
accessModes: [ "ReadWriteOnce" ] storageClassName: standard resources:
requests:
storage: 1Gi


## Slide 86

### StatefulSet


apiVersion: apps/v1 kind: StatefulSet metadata:
name: sts-example
spec:
replicas: 2
revisionHistoryLimit: 3 selector:
matchLabels:
app: stateful serviceName: app updateStrategy: type: RollingUpdate rollingUpdate:
partition: 0 template:
<pod template>


revisionHistoryLimit: The number of previous iterations of the StatefulSet to retain.
serviceName: The name of the associated headless service; or a service without a ClusterIP.


## Slide 87

### Headless service


no need for loadbalancing and a single service IP

service with clusterIP: none

used in order to create DNS records for replicas, which can be used to uniquely identify them (0.svc..., 1.svc...)


## Slide 88

### Headless Service


/ # dig sts-example-0.app.default.svc.cluster.local +noall +answer

; <<>> DiG 9.11.2-P1 <<>> sts-example-0.app.default.svc.cluster.local +noall +answer
;; global options: +cmd
sts-example-0.app.default.svc.cluster.local. 20 IN A 10.255.0.2


apiVersion: v1 kind: Service metadata:
name: app spec:
clusterIP: None
selector:
app: stateful
ports:
- protocol: TCP port: 80
targetPort: 80


$ kubectl get pods


STATUS	RESTARTS

Running	0

Running	0


NAME	READY
AGE
sts-example-0	1/1 11m
sts-example-1	1/1 11m


<StatefulSet Name>-<ordinal>.<service name>.<namespace>.svc.cluster.local


/ # dig app.default.svc.cluster.local +noall +answer

; <<>> DiG 9.11.2-P1 <<>> app.default.svc.cluster.local +noall +answer
;; global options: +cmd
app.default.svc.cluster.local. 2 IN A	10.255.0.5
app.default.svc.cluster.local. 2 IN A	10.255.0.2


/ # dig sts-example-1.app.default.svc.cluster.local +noall +answer

; <<>> DiG 9.11.2-P1 <<>> sts-example-1.app.default.svc.cluster.local +noall +answer
;; global options: +cmd
sts-example-1.app.default.svc.cluster.local. 30 IN A 10.255.0.5


## Slide 89

### Headless Service


/ # dig sts-example-0.app.default.svc.cluster.local +noall +answer

; <<>> DiG 9.11.2-P1 <<>> sts-example-0.app.default.svc.cluster.local +noall +answer
;; global options: +cmd
sts-example-0.app.default.svc.cluster.local. 20 IN A 10.255.0.2


apiVersion: v1 kind: Service metadata:
name: app spec:
clusterIP: None
selector:
app: stateful
ports:
- protocol: TCP port: 80
targetPort: 80


$ kubectl get pods


NAME AGE


READY

sts-example-0	1/1 11m
sts-example-1	1/1


STATUS	RESTARTS

Running	0

Running	0


11m


<StatefulSet Name>-<ordinal>.<service name>.<namespace>.svc.cluster.local


/ # dig app.default.svc.cluster.local +noall +answer

; <<>> DiG 9.11.2-P1 <<>> app.default.svc.cluster.local +noall +answer
;; global options: +cmd
app.default.svc.cluster.local. 2 IN A	10.255.0.5
app.default.svc.cluster.local. 2 IN A	10.255.0.2


/ # dig sts-example-1.app.default.svc.cluster.local +noall +answer

; <<>> DiG 9.11.2-P1 <<>> sts-example-1.app.default.svc.cluster.local +noall +answer
;; global options: +cmd
sts-example-1.app.default.svc.cluster.local. 30 IN A 10.255.0.5


## Slide 90

### Headless Service


/ # dig sts-example-0.app.default.svc.cluster.local +noall +answer

; <<>> DiG 9.11.2-P1 <<>> sts-example-0.app.default.svc.cluster.local +noall +answer
;; global options: +cmd
sts-example-0.app.default.svc.cluster.local. 20 IN A 10.255.0.2


apiVersion: v1 kind: Service metadata:
name: app spec:
clusterIP: None
selector:
app: stateful
ports:
- protocol: TCP port: 80
targetPort: 80


$ kubectl get pods


NAME AGE


READY

sts-example-0	1/1 11m
sts-example-1	1/1


STATUS	RESTARTS

Running	0

Running	0


11m


<StatefulSet Name>-<ordinal>.<service name>.<namespace>.svc.cluster.local


/ # dig app.default.svc.cluster.local +noall +answer

; <<>> DiG 9.11.2-P1 <<>> app.default.svc.cluster.local +noall +answer
;; global options: +cmd
app.default.svc.cluster.local. 2 IN A	10.255.0.5
app.default.svc.cluster.local. 2 IN A	10.255.0.2


/ # dig sts-example-1.app.default.svc.cluster.local +noall +answer

; <<>> DiG 9.11.2-P1 <<>> sts-example-1.app.default.svc.cluster.local +noall +answer
;; global options: +cmd


sts-example-1.app.default.svc.cluster.local. 30 IN A 10.255.0.5


## Slide 91

### CronJob


An extension of the Job Controller, it provides a method of executing jobs on a cron-like schedule.


CronJobs within Kubernetes use UTC ONLY.


## Slide 92

### CronJob


schedule: The cron schedule for the job.
successfulJobHistoryLimit: The number of successful jobs to retain.
failedJobHistoryLimit: The number of failed jobs to retain.


apiVersion: batch/v1beta1 kind: CronJob
metadata:
name: cronjob-example spec:
schedule: "*/1 * * * *" successfulJobsHistoryLimit: 3
failedJobsHistoryLimit: 1 jobTemplate:
spec:
completions: 4
parallelism: 2 template:
<pod template>


## Slide 93

### CronJob


$ kubectl describe cronjob cronjob-example Name:	cronjob-example
Namespace:	default
Labels:	<none>
Annotations:	<none>
Schedule:	*/1 * * * *
Concurrency Policy:	Allow
Suspend:	False
Starting Deadline Seconds: <unset>
Selector:	<unset>
Parallelism:	2
Completions:	4
Pod Template: Labels: <none>
Containers:
hello:
Image: alpine:latest Port:	<none> Command:
/bin/sh
-c Args:
echo hello from $HOSTNAME!
Environment:	<none>
Mounts:	<none>
Volumes:	<none>
Last Schedule Time: Mon, 19 Feb 2018 09:54:00 -0500 Active Jobs:	cronjob-example-1519052040 Events:
Type	Reason

----	------
Normal SuccessfulCreate Normal SawCompletedJob Normal SuccessfulCreate
Normal SawCompletedJob Normal SuccessfulCreate

Age	From	Message
---- ----	-------
3m	cronjob-controller Created job cronjob-example-1519051860
2m	cronjob-controller Saw completed job: cronjob-example-1519051860 2m	cronjob-controller Created job cronjob-example-1519051920
1m	cronjob-controller Saw completed job: cronjob-example-1519051920 1m	cronjob-controller Created job cronjob-example-1519051980


apiVersion: batch/v1beta1 kind: CronJob
metadata:
name: cronjob-example
spec:
schedule: "*/1 * * * *" successfulJobsHistoryLimit: 3
failedJobsHistoryLimit: 1 jobTemplate:
spec:
completions: 4
parallelism: 2
template:
spec:
containers:
- name: hello
image: alpine:latest
command: ["/bin/sh", "-c"]
args: ["echo hello from $HOSTNAME!"]
restartPolicy: Never


$ kubectl get jobs |  |  | 
NAME | DESIRED | SUCCESSFUL | AGE
cronjob-example-1519053240 | 4 | 4 | 2m
cronjob-example-1519053300 | 4 | 4 | 1m
cronjob-example-1519053360 | 4 | 4 | 26s


## Slide 94

### Health checks


initialDelaySeconds: Number of seconds after the container has started before liveness or readiness probes are initiated.
periodSeconds: How often (in seconds) to perform the probe. Default to 10 seconds. Minimum value is 1.
timeoutSeconds: Number of seconds after which the probe times out. Defaults to 1 second. Minimum value is 1.
successThreshold: Minimum consecutive successes for the probe to be considered successful after having failed.
Defaults to 1. Must be 1 for liveness. Minimum value is 1.
failureThreshold: When a Pod starts and the probe fails, Kubernetes will try failureThreshold times before giving up. Giving up in case of liveness probe means restarting the Pod. In case of readiness probe the Pod will be marked Unready. Defaults to 3. Minimum value is 1.


apiVersion: v1 kind: Pod metadata:
labels:
test: liveness
name: liveness-readiness-http spec:
containers:
- name: liveness-readiness-http
image: k8s.gcr.io/ liveness-readiness-http livenessProbe:
httpGet:
path: /healthz port: 8080
initialDelaySeconds: 5
periodSeconds: 10
timeoutSeconds: 4
failureThreshold: 5 readinessProbe:
httpGet:
path: /healthz port: 8080
initialDelaySeconds: 100
periodSeconds: 10
timeoutSeconds: 4
failureThreshold: 2


## Slide 96

### Helm


## Slide 97

### What is Helm?


package manager for Kubernetes

provides higher-level abstraction (Chart) to configure full-fledged applications

manage complexity, easy upgrades, simple sharing of full application setups, safe rollbacks


## Slide 98

### How does Helm work?


Helm CLI + Tiller server in Kubernetes (which is a controller)

CLI responsible for management + requests for releases of charts on Kubernetes

Tiller - listens for requests, combines chart + configuration = release, install release, track release


## Slide 99

### Helm++


Helm release controller - current Lentiq way to manage applications

expose HelmRelease as a CRD (custom resource definition) in Kubernetes, to work directly with Kubernetes to manage apps


## Slide 100

### Architecture Overview


Control Plane Components


## Slide 101

### Control Plane Components


kube-apiserver
etcd
kube-controller-manager
kube-scheduler
cloud-controller-manager


## Slide 102

### kube-apiserver


Provides a forward facing REST interface into the kubernetes control plane and datastore.
All clients and other applications interact with kubernetes strictly through the API Server.
Acts as the gatekeeper to the cluster by handling authentication and authorization, request validation, mutation, and admission control in addition to being the front-end to the backing datastore.


## Slide 103

### etcd


etcd acts as the cluster datastore.
Purpose in relation to Kubernetes is to provide a strong, consistent and highly available key-value store for persisting cluster state.
Stores objects and config information.


## Slide 104

### etcd


Uses “Raft Consensus” among a quorum of systems to create a fault-tolerant consistent “view” of the cluster.


https://raft.github.io/


Image Source


## Slide 105

### kube-controller-manager


Monitors the cluster state via the apiserver and steers the cluster towards the desired state.
Node Controller: Responsible for noticing and responding when nodes go down.
Replication Controller: Responsible for maintaining the correct number of pods for every replication controller object in the system.
Endpoints Controller: Populates the Endpoints object (that is, joins Services & Pods).
Service Account & Token Controllers: Create default accounts and API access tokens for new namespaces.


## Slide 106

### kube-scheduler


Component on the master that watches newly created pods that have no node assigned, and selects a node for them to run on.
Factors taken into account for scheduling decisions include individual and collective resource requirements, hardware/software/policy constraints, affinity and anti- affinity specifications, data locality, inter-workload interference and deadlines.


## Slide 107

### cloud-controller-manager


Node Controller: For checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding
Route Controller: For setting up routes in the underlying cloud infrastructure
Service Controller: For creating, updating and deleting cloud provider load balancers
Volume Controller: For creating, attaching, and mounting volumes, and interacting with the cloud provider to orchestrate volumes


## Slide 108

### Architecture Overview


Node Components


## Slide 109

### Node Components


kubelet
kube-proxy
Container Runtime Engine


## Slide 110

### kubelet


An agent that runs on each node in the cluster. It makes sure that containers are running in a pod.
The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy.


## Slide 111

### kube-proxy


Manages the network rules on each node.
Performs connection forwarding or load balancing for Kubernetes cluster services.


## Slide 112

### Container Runtime Engine


A container runtime is a CRI (Container Runtime Interface) compatible application that executes and manages containers.
Containerd (docker)
Cri-o
Rkt
Kata (formerly clear and hyper)
Virtlet (VM CRI compatible runtime)


## Slide 114

### Architecture Overview


Security


## Slide 115

### User


Pod


Authentication


Authonization


Admission Control


Kubernetes API Server


Access Control Diagram


## Slide 116

### Authentication


X509 Client Certs (CN used as user, Org fields as group) No way to revoke them!! – wip ©
Static Password File (password,user,uid,"group1,group2,group3")
Static Token File (token,user,uid,"group1,group2,group3")
Bearer Token (Authorization: Bearer 31ada4fd-ade)
Bootstrap Tokens (Authorization: Bearer 781292.db7bc3a58fc5f07e)
Service Account Tokens (signed by API server’s private TLS key or specified by file)


## Slide 117

### Role - Authorization


## Slide 118

### RoleBinding - Authorization


## Slide 119

### RoleBinding - Authorization


## Slide 120

### Admission Control


AlwaysPullImages
DefaultStorageClass
DefaultTolerationSeconds
DenyEscalatingExec
EventRateLimit
ImagePolicyWebhook
LimitRanger/ResourceQuota
PersistentVolumeClaimResize
PodSecurityPolicy


## Slide 123

### Request/Response


{


"apiVersion": "authentication.k8s.io/v1beta1",
"kind": "TokenReview", "status": {
"authenticated": true,
"user": { "username":
"janedoe@example.com", "uid": "42",
"groups": [
"developers", "qa"
]
}
}
}


{
"apiVersion":
"authentication.k8s.io/v1beta1", "kind": "TokenReview",
"spec": {
"token": "(BEARERTOKEN)"
}
}


## Slide 124

### Concepts and Resources


Storage


Volumes
Persistent Volumes
Persistent Volume Claims
StorageClass


## Slide 125

### Persistence of data


how to define the physical location of data and how much should be allocated to each ?

how to actually allocate and link certain data to specific pods ?

abstracted and decoupled through PersistentVolume subsystem


## Slide 126

### PersistentVolume


abstraction of a piece of storage in the cluster that can be used

lifecycle independent of any individual pod that uses it

can be manually put by an operator or can be provisioned dynamically (usually in cloud services)


## Slide 127

### PersistentVolumeClaim


request for storage by an user

storage equivalent of a pod

links a persistent volume to a pod for the pod's lifetime

doesn't affect the persistent volume upon pod deletion (unless explicitly specified)


## Slide 128

### Storage


Pods by themselves are useful, but many workloads require exchanging data between containers, or persisting some form of data.

For this we have Volumes, PersistentVolumes, PersistentVolumeClaims, and StorageClasses.


## Slide 129

### StorageClass


Storage classes are an abstraction on top of an external storage resource (PV)
Work hand-in-hand with the external storage system to enable dynamic provisioning of storage by eliminating the need for the cluster admin to pre-provision a PV


## Slide 130

### StorageClass


provisioner: Defines the ‘driver’ to be used for provisioning of the external storage.
parameters: A hash of the various
configuration parameters for the provisioner.
reclaimPolicy: The behaviour for
the backing storage when the PVC is deleted.
Retain - manual clean-up
Delete - storage asset deleted by provider


kind: StorageClass apiVersion: storage.k8s.io/v1 metadata:
name: standard
provisioner: kubernetes.io/gce-pd parameters:
type: pd-standard
zones: us-central1-a, us-central1-b
reclaimPolicy: Delete


## Slide 131

### Available StorageClasses


AWSElasticBlockStore
AzureFile
AzureDisk
CephFS
Cinder
FC
Flocker
GCEPersistentDisk
Glusterfs


iSCSI
Quobyte
NFS
RBD
VsphereVolume
PortworxVolume
ScaleIO
StorageOS
Local


Internal Provisioner


## Slide 132

### Volumes


Storage that is tied to the Pod’s Lifecycle.
A pod can have one or more types of volumes attached to it.
Can be consumed by any of the containers within the pod.
Survive Pod restarts; however their durability beyond that is dependent on the Volume Type.


## Slide 133

### Volume Types


awsElasticBlockStore
azureDisk
azureFile
cephfs
configMap
csi
downwardAPI
emptyDir
fc (fibre channel)


flocker
gcePersistentDisk
gitRepo
glusterfs
hostPath
iscsi
local
nfs
persistentVolume
Claim


projected
portworxVolume
quobyte
rbd
scaleIO
secret
storageos
vsphereVolume


Persistent Volume Supported


## Slide 134

### Volumes


volumes: A list of volume objects to be attached to the Pod. Every object within the list must have it’s own unique name.
volumeMounts: A container specific list referencing the Pod volumes by name, along with their desired mountPath.


apiVersion: v1 kind: Pod metadata:
name: volume-example spec:
containers:
name: nginx
image: nginx:stable-alpine volumeMounts:
- name: html
mountPath: /usr/share/nginx/html ReadOnly: true
name: content
image: alpine:latest command: ["/bin/sh", "-c"] args:
- while true; do
date >> /html/index.html; sleep 5;
done volumeMounts:
- name: html
mountPath: /html volumes:
- name: html emptyDir: {}


## Slide 135

### Volumes


volumes: A list of volume objects to be attached to the Pod. Every object within the list must have it’s own unique name.


volumeMounts: A container specific list referencing the Pod volumes by name, along with their desired mountPath.


apiVersion: v1 kind: Pod metadata:
name: volume-example spec:
containers:
name: nginx
image: nginx:stable-alpine volumeMounts:
- name: html
mountPath: /usr/share/nginx/html ReadOnly: true
name: content
image: alpine:latest command: ["/bin/sh", "-c"] args:
- while true; do
date >> /html/index.html; sleep 5;
done volumeMounts:
- name: html
mountPath: /html volumes:
- name: html emptyDir: {}


## Slide 136

### Volumes


volumes: A list of volume objects to be attached to the Pod. Every object within the list must have it’s own unique name.


volumeMounts: A container specific list referencing the Pod volumes by name, along with their desired mountPath.


apiVersion: v1 kind: Pod metadata:
name: volume-example spec:
containers:
name: nginx
image: nginx:stable-alpine volumeMounts:
- name: html
mountPath: /usr/share/nginx/html ReadOnly: true
name: content
image: alpine:latest command: ["/bin/sh", "-c"] args:
- while true; do
date >> /html/index.html; sleep 5;
done volumeMounts:
- name: html
mountPath: /html volumes:
- name: html emptyDir: {}


## Slide 137

### Persistent Volumes


A PersistentVolume (PV) represents a storage resource.
PVs are a cluster wide resource linked to a backing storage provider: NFS, GCEPersistentDisk, RBD etc.
Generally provisioned by an administrator.
Their lifecycle is handled independently from a pod
CANNOT be attached to a Pod directly. Relies on a
PersistentVolumeClaim


## Slide 138

### PersistentVolumeClaims


A PersistentVolumeClaim (PVC) is a namespaced
request for storage.
Satisfies a set of requirements instead of mapping to a storage resource directly.
Ensures that an application’s ‘claim’ for storage is portable across numerous backends or providers.


## Slide 139

### apiVersion: v1
kind: PersistentVolume metadata:
name: nfsserver
spec:
capacity:
storage: 50Gi volumeMode: Filesystem accessModes:
ReadWriteOnce
ReadWriteMany
persistentVolumeReclaimPolicy: Delete storageClassName: slow
mountOptions:
hard
nfsvers=4.1
nfs:
path: /exports server: 172.22.0.42


PersistentVolume


capacity.storage: The total amount of available storage.
volumeMode: The type of volume, this can be either Filesystem or Block.
accessModes: A list of the supported methods of accessing the volume. Options include:
ReadWriteOnce
ReadOnlyMany
ReadWriteMany


## Slide 140

### PersistentVolume


persistentVolumeReclaimPolicy: The behaviour for PVC’s that have been deleted. Options include:
Retain - manual clean-up
Delete - storage asset deleted by provider.
storageClassName: Optional name of the storage class that PVC’s can reference. If provided, ONLY PVC’s referencing the name consume use it.
mountOptions: Optional mount options for the PV.


apiVersion: v1
kind: PersistentVolume metadata:
name: nfsserver
spec:
capacity:
storage: 50Gi volumeMode: Filesystem accessModes:
ReadWriteOnce
ReadWriteMany
persistentVolumeReclaimPolicy: Delete storageClassName: slow
mountOptions:
hard
nfsvers=4.1
nfs:
path: /exports server: 172.22.0.42


## Slide 141

### PersistentVolumeClaim


accessModes: The selected method of accessing the storage. This MUST be a subset of what is defined on the target PV or Storage Class.
ReadWriteOnce
ReadOnlyMany
ReadWriteMany
resources.requests.storage: The desired amount of storage for the claim
storageClassName: The name of the desired Storage Class


kind: PersistentVolumeClaim apiVersion: v1
metadata:
name: pvc-sc-example spec:
accessModes:
- ReadWriteOnce resources:
requests: storage: 1Gi
storageClassName: slow


## Slide 142

### PVs and PVCs with Selectors


kind: PersistentVolume apiVersion: v1 metadata:
name: pv-selector-example labels:
type: hostpath
spec:
capacity: storage: 2Gi
accessModes:
- ReadWriteMany hostPath:
path: "/mnt/data"


kind: PersistentVolumeClaim apiVersion: v1
metadata:
name: pvc-selector-example spec:
accessModes:
- ReadWriteMany resources:
requests: storage: 1Gi
selector:
matchLabels: type: hostpath


## Slide 143

### PVs and PVCs with Selectors


kind: PersistentVolume apiVersion: v1 metadata:
name: pv-selector-example labels:
type: hostpath
spec:
capacity: storage: 2Gi
accessModes:
- ReadWriteMany hostPath:
path: "/mnt/data"


kind: PersistentVolumeClaim apiVersion: v1
metadata:
name: pvc-selector-example spec:
accessModes:
- ReadWriteMany resources:
requests: storage: 1Gi


selector: matchLabels:
type: hostpath


## Slide 144

### PV Phases


Available
PV is ready and available to be consumed.


Bound
The PV has been bound to a claim.


Released
The binding PVC has been deleted, and the PV is pending reclamation.


Failed
An error has been encountered.


## Slide 145

### StorageClass


pv: pvc-9df65c6e-1a69-11e8-ae10-080027a3682b


uid: 9df65c6e-1a69-11e8-ae10-080027a3682b


2. StorageClass provisions request through API with
1. PVC makes a request of	external storage system. the StorageClass.


3. External storage system creates a PV strictly satisfying the PVC request.


4. provisioned PV is bound to requesting PVC.


## Slide 146

### Persistent Volumes and Claims


Cluster Users


Cluster Admins


## Slide 147

### Lab - github.com/mrbobbytables/k8s-intro-tutorials/blob/master/storage


Working with Volumes


## Slide 148

### Concepts and Resources


Configuration


ConfigMap
Secret


## Slide 149

### Configuration


Kubernetes has an integrated pattern for decoupling configuration from application or container.

This pattern makes use of two Kubernetes components: ConfigMaps and Secrets.


## Slide 150

### ConfigMap


Externalized data stored within kubernetes.
Can be referenced through several different means:
environment variable
a command line argument (via env var)
injected as a file into a volume mount
Can be created from a manifest, literals, directories, or files directly.


## Slide 151

### ConfigMap


data: Contains key-value pairs of ConfigMap contents.


apiVersion: v1 kind: ConfigMap metadata:
name: manifest-example data:
state: Michigan
city: Ann Arbor content: |
Look at this,
its multiline!


## Slide 152

### ConfigMap Example


apiVersion: v1 kind: ConfigMap metadata:
name: manifest-example data:
city: Ann Arbor
state: Michigan


$ kubectl create configmap literal-example \
--from-literal="city=Ann Arbor" --from-literal=state=Michigan
configmap “literal-example” created


$ cat info/city Ann Arbor
$ cat info/state Michigan
$ kubectl create configmap file-example --from-file=cm/city --from-file=cm/state
configmap "file-example" created


All produce a ConfigMap with the same content!


$ cat info/city Ann Arbor
$ cat info/state Michigan
$ kubectl create configmap dir-example --from-file=cm/
configmap "dir-example" created


## Slide 153

### ConfigMap Example


apiVersion: v1 kind: ConfigMap metadata:
name: manifest-example data:
city: Ann Arbor
state: Michigan


$ kubectl create configmap literal-example \
--from-literal="city=Ann Arbor" --from-literal=state=Michigan
configmap “literal-example” created


$ cat info/city Ann Arbor
$ cat info/state Michigan
$ kubectl create configmap file-example --from-file=cm/city --from-file=cm/state
configmap "file-example" created


All produce a ConfigMap with the same content!


$ cat info/city Ann Arbor
$ cat info/state Michigan
$ kubectl create configmap dir-example --from-file=cm/
configmap "dir-example" created


## Slide 154

### ConfigMap Example


apiVersion: v1 kind: ConfigMap metadata:
name: manifest-example data:
city: Ann Arbor
state: Michigan


$ kubectl create configmap literal-example \
--from-literal="city=Ann Arbor" --from-literal=state=Michigan
configmap “literal-example” created


$ cat info/city Ann Arbor
$ cat info/state Michigan
$ kubectl create configmap file-example --from-file=cm/city --from-file=cm/state
configmap "file-example" created


All produce a ConfigMap with the same content!

--from-file=cm/

$ cat info/city Ann Arbor
$ cat info/state Michigan
$ kubectl create configmap dir-example configmap "dir-example" created


## Slide 155

### ConfigMap Example


apiVersion: v1 kind: ConfigMap metadata:
name: manifest-example data:
city: Ann Arbor
state: Michigan


$ kubectl create configmap literal-example \
--from-literal="city=Ann Arbor" --from-literal=state=Michigan
configmap “literal-example” created

--from-file=cm/city --from-file=cm/state

$ cat info/city Ann Arbor
$ cat info/state Michigan
$ kubectl create configmap file-example configmap "file-example" created


All produce a ConfigMap with the same content!


$ cat info/city Ann Arbor
$ cat info/state Michigan
$ kubectl create configmap dir-example --from-file=cm/
configmap "dir-example" created


## Slide 156

### Secret


Functionally identical to a ConfigMap.
Stored as base64 encoded content.
Encrypted at rest within etcd (if configured!).
Stored on each worker node in tmpfs directory.
Ideal for username/passwords, certificates or other 	sensitive information that should not be stored in a 	container.


## Slide 157

### Secret


type: There are three different types of secrets within Kubernetes:
docker-registry - credentials used to authenticate to a container registry
generic/Opaque - literal values
from different sources
tls - a certificate based secret
data: Contains key-value pairs of base64 encoded content.


apiVersion: v1 kind: Secret metadata:
name: manifest-secret type: Opaque
data:
username: ZXhhbXBsZQ== password: bXlwYXNzd29yZA==


## Slide 158

### Secret Example


apiVersion: v1 kind: Secret metadata:
name: manifest-example type: Opaque
data:
username: ZXhhbXBsZQ== password: bXlwYXNzd29yZA==


$ kubectl create secret generic literal-secret \
--from-literal=username=example \
--from-literal=password=mypassword
secret "literal-secret" created


$ cat secret/username example
$ cat secret/password mypassword
$ kubectl create secret generic file-secret --from-file=secret/username --from-file=secret/password
Secret "file-secret" created


All produce a Secret with the same content!


$ cat info/username example
$ cat info/password mypassword
$ kubectl create secret generic dir-secret --from-file=secret/ Secret "file-secret" created


## Slide 159

### Secret Example


apiVersion: v1 kind: Secret metadata:
name: manifest-example type: Opaque
data:
username: ZXhhbXBsZQ== password: bXlwYXNzd29yZA==

--from-literal=username=example \
--from-literal=password=mypassword

$ kubectl create secret generic literal-secret \
>
>
secret "literal-secret" created


$ cat secret/username example
$ cat secret/password mypassword
$ kubectl create secret generic file-secret --from-file=secret/username --from-file=secret/password
Secret "file-secret" created


All produce a Secret with the same content!


$ cat info/username example
$ cat info/password mypassword
$ kubectl create secret generic dir-secret --from-file=secret/ Secret "file-secret" created


## Slide 160

### Secret Example


apiVersion: v1 kind: Secret metadata:
name: manifest-example type: Opaque
data:
username: ZXhhbXBsZQ== password: bXlwYXNzd29yZA==


$ kubectl create secret generic literal-secret \
--from-literal=username=example \
--from-literal=password=mypassword
secret "literal-secret" created


$ cat secret/username example
$ cat secret/password mypassword
$ kubectl create secret generic file-secret --from-file=secret/username --from-file=secret/password
Secret "file-secret" created


All produce a Secret with the same content!

--from-file=secret/

$ cat info/username example
$ cat info/password mypassword
$ kubectl create secret generic dir-secret Secret "file-secret" created


## Slide 161

### Secret Example


apiVersion: v1 kind: Secret metadata:
name: manifest-example type: Opaque
data:
username: ZXhhbXBsZQ== password: bXlwYXNzd29yZA==


$ kubectl create secret generic literal-secret \
--from-literal=username=example \
--from-literal=password=mypassword
secret "literal-secret" created

--from-file=secret/username --from-file=secret/password

$ cat secret/username example
$ cat secret/password mypassword
$ kubectl create secret generic file-secret Secret "file-secret" created


All produce a Secret with the same content!


$ cat info/username example
$ cat info/password mypassword
$ kubectl create secret generic dir-secret --from-file=secret/ Secret "file-secret" created


## Slide 162

### Injecting as Environment Variable


env:
- name: CITY valueFrom:
configMapKeyRef:
name: manifest-example key: city


apiVersion: batch/v1 kind: Job
metadata:
name: cm-env-example spec:
template:
spec:
containers:
- name: mypod
image: alpine:latest command: [“/bin/sh”, “-c”]
args: [“printenv CITY”]






restartPolicy: Never


env:
- name: USERNAME
valueFrom: secretKeyRef:
name: manifest-example key: username


apiVersion: batch/v1 kind: Job
metadata:
name: secret-env-example spec:
template:
spec:
containers:
- name: mypod
image: alpine:latest command: [“/bin/sh”, “-c”]
args: [“printenv USERNAME”]






restartPolicy: Never


## Slide 163

### Injecting as Environment Variable


image: alpine:latest
command: [“/bin/sh”, “-c”]
args: [“printenv CITY”]
env:


apiVersion: batch/v1 kind: Job
metadata:
name: cm-env-example spec:
template:
spec:
containers:
- name: mypod




- name: CITY valueFrom:
configMapKeyRef:
name: manifest-example key: city
restartPolicy: Never


image: alpine:latest
command: [“/bin/sh”, “-c”]
args: [“printenv USERNAME”]
env:


apiVersion: batch/v1 kind: Job
metadata:
name: secret-env-example spec:
template:
spec:
containers:
- name: mypod




- name: USERNAME
valueFrom: secretKeyRef:
name: manifest-example key: username
restartPolicy: Never


## Slide 164

### Injecting in a Command


apiVersion: batch/v1 kind: Job
metadata:
name: cm-cmd-example spec:
template:
spec:
containers:
- name: mypod
image: alpine:latest


command: [“/bin/sh”, “-c”]
args: [“echo Hello ${CITY}!”]


env:
- name: CITY valueFrom:
configMapKeyRef:
name: manifest-example


key: city


restartPolicy: Never


apiVersion: batch/v1 kind: Job
metadata:
name: secret-cmd-example spec:
template:
spec:
containers:
- name: mypod
image: alpine:latest


command: [“/bin/sh”, “-c”]
args: [“echo Hello ${USERNAME}!”]


env:
- name: USERNAME
valueFrom: secretKeyRef:
name: manifest-example


key: username


restartPolicy: Never


## Slide 165

### Injecting in a Command


image: alpine:latest
command: [“/bin/sh”, “-c”]
args: [“echo Hello ${CITY}!”]
env:


apiVersion: batch/v1 kind: Job
metadata:
name: cm-cmd-example spec:
template:
spec:
containers:
- name: mypod




- name: CITY valueFrom:
configMapKeyRef:
name: manifest-example key: city
restartPolicy: Never


image: alpine:latest
command: [“/bin/sh”, “-c”]
args: [“echo Hello ${USERNAME}!”]
env:


apiVersion: batch/v1 kind: Job
metadata:
name: secret-cmd-example spec:
template:
spec:
containers:
- name: mypod




- name: USERNAME
valueFrom: secretKeyRef:
name: manifest-example key: username
restartPolicy: Never


## Slide 166

### Injecting as a Volume


apiVersion: batch/v1 kind: Job
metadata:
name: cm-vol-example spec:
template:
spec:
containers:
- name: mypod
image: alpine:latest command: [“/bin/sh”, “-c”]
args: [“cat /myconfig/city”] volumeMounts:
- name: config-volume mountPath: /myconfig
restartPolicy: Never |  | 
 | volumes:
- name: config-volume
configMap:
name: manifest-example | 


apiVersion: batch/v1 kind: Job
metadata:
name: secret-vol-example spec:
template:
spec:
containers:
- name: mypod
image: alpine:latest command: [“/bin/sh”, “-c”]
args: [“cat /mysecret/username”] volumeMounts:
- name: secret-volume mountPath: /mysecret
restartPolicy: Never |  | 
 | volumes:
- name: secret-volume
secret:
secretName: manifest-example | 


## Slide 167

### Injecting as a Volume


apiVersion: batch/v1 kind: Job
metadata:
name: cm-vol-example spec:
template:
spec:
containers:
- name: mypod
image: alpine:latest


command: [“/bin/sh”, “-c”] args: [“cat /myconfig/city”]


volumeMounts:
- name: config-volume mountPath: /myconfig
restartPolicy: Never


volumes:
- name: config-volume
configMap:
name: manifest-example


apiVersion: batch/v1 kind: Job
metadata:
name: secret-vol-example spec:
template:
spec:
containers:
- name: mypod
image: alpine:latest


command: [“/bin/sh”, “-c”]
args: [“cat /mysecret/username”]


volumeMounts:
- name: secret-volume


mountPath: /mysecret


restartPolicy: Never


volumes:
- name: secret-volume
secret:
secretName: manifest-example


## Slide 168

### Concepts and Resources


Metrics and Monitoring


Metrics server
HPA (horizontal pod autoscaler)
Prometheus
Grafana (dashboards)
Fluentd (log shipping)


## Slide 169

### Metrics API Server


Metric server collects metrics such as CPU and Memory by each pod and node from the Summary API, exposed by Kubelet on each node.
Metrics Server registered in the main API server through Kubernetes aggregator, which was introduced in Kubernetes 1.7


## Slide 170

### HPA


## Slide 171

### Horizontal Pod Autoscaling


$ kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10 deployment "php-apache" autoscaled


$ kubectl get hpa


NAME	REFERENCE
php-apache	Deployment/php-apache/scale


TARGET
58'.


CURRENT
305--.


MINPODS


MAXPODS	AGE
10	3m


$ kubectl get deployment php-apache
NAME	DESIRED	CURRENT	UP-TO-DATE	AVAILABLE	AGE
php-apache	7	7	7	7	19m


apiVersion: extensions/vlbetal kind: HorizontalPodAutoscaler metadata:
name: php-apache
namespace: default spec:
scaleRef:
kind: Deployment name: php-apache subresource: scale
minReplicas: 1
maxReplicas: 10 cpuUtilization:
targetPercentage: 50


Tips

Scale ouVin

TriggeredScaleUp (GCE, AWS, will add more)

Support for custom metrics


annotations:
alpha/target .custom-metrics. podautoscaler. kubernetes. io:  '{" i terns": [ {•name": "qps",  "value" : "10"} I}'


## Slide 172

### Short-lived jobs


pushmetrics at exit


0	kubernetes


Service discovery

file sd


Pushgateway


:discover targets


Prometheus server


pull --------
metrics

----r	Retrieval 'I ---GSDB	-··GTTP
server
,J

0

Prometheus alerting
...---------


..--{ pagerduty ]


Alertmanager

•
-------------{

E_m_ai_l


]

-notify   {

e_tc		]


push alerts


PromQL


t
il=H·tfM·i


I


.....


,


Prometheus
	webUI


'


Jobs/ exporters


Node


Grafana


Prometheus targets


Data visualization and export


!................,


API clients


## Slide 173

### ,.


I


I


I


'


'►


'r


I


,


I I
'I


I


'I


I I I


'I


I


r
I
'►


'i


I I


'I


I


I


I I I


I


►


I
f


I


I
I
I I I I

-	--:

I I I I I I


I I


I I I I I I


I I I I I I I I
I
I


,I


ellasticseairch


:9200


kibana


I
I


'I


\


I
f


I

•

...

•
•

.,

•

y----	J


Nodes

-

 |  | - -- | - - - | -
 |  |  |  | 

-

.,,,,,


## Slide 175

### Kubernetes and Big Data


## Slide 176

### APPS CREATED FROM INTERFACE


## Slide 177

### APPS CREATED FROM INTERFACE


user@cloudshell:~ (demo)$ kubectl get helmreleases --namespace=demo
NAME                AGE
internal-sparksql  24m
jupyter                17m
spark                  12m

user@cloudshell:~ (demo)$ kubectl get statefulsets --namespace=demo
NAME                                                       DESIRED CURRENT AGE
demo-internal-sparksql-bdl-sparksql-master       1               1        24m
demo-internal-sparksql-bdl-sparksql-worker      1               1        24m
demo-jupyter-bdl-jupyter                                  1               1        17m
demo-spark-bdl-spark-master                           1               1        13m
demo-spark-bdl-spark-worker                          1               1        13m

user@cloudshell:~ (demo)$ kubectl get pods --namespace=demo
NAME                                                         READY   STATUS    RESTARTS   AGE
demo-internal-sparksql-bdl-sparksql-master-0   2/2        Running           2          24m
demo-internal-sparksql-bdl-sparksql-worker-0   1/1       Running           0          24m
demo-jupyter-bdl-jupyter-0                               1/1       Running           0          18m
demo-spark-bdl-spark-master-0                        2/2       Running           0          13m
demo-spark-bdl-spark-worker-0                       1/1       Running           0          13m


## Slide 178

### APPS CREATED FROM INTERFACE


user@cloudshell:~ (demo)$ kubectl get services --namespace=demo
NAME                                                                   TYPE                CLUSTER-IP      EXTERNAL-IP     PORT(S)                                                                                       AGE
demo-internal-sparksql-bdl-sparksql-master             ClusterIP           None                <none>               4040/TCP,8080/TCP,7077/TCP,10000/TCP                                      24m
demo-internal-sparksql-bdl-sparksql-master-public    LoadBalancer    10.31.252.242    34.80.170.123       80:32438/TCP,4040:30330/TCP,7077:30605/TCP,10000:32755/TCP  24m
demo-internal-sparksql-bdl-sparksql-worker             ClusterIP          None                <none>                8081/TCP                                                                                    24m
demo-internal-sparksql-bdl-sparksql-worker-public    LoadBalancer   10.31.254.85       34.80.191.210      8081:32181/TCP                                                                           24m
demo-jupyter-bdl-jupyter                                         ClusterIP          None                <none>               8888/TCP                                                                                    18m
demo-jupyter-bdl-jupyter-public                                LoadBalancer   10.31.253.99       34.80.208.250     8888:30753/TCP                                                                          18m
demo-spark-bdl-spark-master                                  ClusterIP          None                <none>               4040/TCP,8080/TCP,7077/TCP                                                      13m
demo-spark-bdl-spark-master-public                         LoadBalancer    10.31.249.147    35.201.134.214    80:31575/TCP,4040:32010/TCP,7077:30910/TCP                            13m
demo-spark-bdl-spark-worker                                 ClusterIP           None               <none>                8081/TCP                                                                                   13m
demo-spark-bdl-spark-worker-public                        LoadBalancer    10.31.243.213    35.201.202.111     8081:31535/TCP                                                                         13m

user@cloudshell:~ (demo)$ kubectl get pvc --namespace=demo
NAME                                                                    STATUS    VOLUME                                                        CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-demo-internal-sparksql-bdl-sparksql-master-0      Bound     pvc-f37593fa-43e4-11e9-90c0-42010a8c0026     15Gi               RWO                   standard              1h
data-demo-internal-sparksql-bdl-sparksql-worker-0     Bound      pvc-f376aa77-43e4-11e9-90c0-42010a8c0026    15Gi               RWO                   standard              1h
data-demo-jupyter-bdl-jupyter                                    Bound     pvc-d9e302f5-43e5-11e9-90c0-42010a8c0026   15Gi                RWO                   standard             1h
data-demo-spark-bdl-spark-master-0                           Bound     pvc-7e301d28-43e6-11e9-90c0-42010a8c0026   15Gi               RWO                   standard             1h
data-demo-spark-bdl-spark-worker-0                          Bound     pvc-7e314394-43e6-11e9-90c0-42010a8c0026   15Gi               RWO                    standard             1h


## Slide 179

### UPLOAD DATA VIA JUPYTER


## Slide 180

### CONNECT TO SPARK AND RUN QUERY


## Slide 181

### SPARK UI SHOWS CONNECTION FROM JUPYTER


## Slide 182

### kubectl Commands


Apply Kubernetes Pod
kubectl apply -f <pod yaml file>
kubectl apply -f example_pod.yml


Execute Command in Pod
kubectl exec -it <pod_name> -- <command>
kubectl exec -it instructor-test-01 -- /bin/bash


Get Azure Kubernetes Service Credentials
az aks get-credentials --resource-group rg-ahab-dev-eastus-001 --name kub-ahab-dev-eastus-001 --overwrite-existing
