# Lecture_2.1_Computing_Models


## Slide 1

### Models for Distributed Systems & Cloud Computing


## Slide 2

### Client-Server
Peer-to-Peer
Services provided by multiple servers
Proxy servers and caches
Mobile code and mobile agents
Thin clients and mobile devices
Event Driven Architechture


2


Distributed System Models


## Slide 3

### 3


1. Client-Server Model:


Clients request services, servers provide them. 
Examples: Web applications, database servers.


## Slide 4

### 4


2. Peer-to-peer Systems


All nodes are equal and share resources directly. 
Examples: BitTorrent, blockchain.


## Slide 5

### 5


3. A Service by Multiple Servers


Presentation tier, application tier, data tier. 
Widely used in web applications.


## Slide 6

### 6


4. Web Proxy Server


A web proxy server is an intermediary server that sits between a client (e.g., a browser) and the internet. It processes client requests by forwarding them to the intended server and then returning the server's response to the client. Web proxies enhance privacy, security, and performance.


## Slide 7

### 7


5. Web Applets


## Slide 8

### Thin


Client


Application


Process


Network computer or PC


Compute server


network


8


6. Thin Clients and Compute Servers


A thin client is a lightweight computing device that depends heavily on a server for computational power, applications, and data storage.
Minimal hardware requirements. No local storage or minimal local processing. Connects to a central server or cloud to perform tasks.
Web browsers as thin clients for SaaS applications. 
Devices in virtual desktop infrastructure (VDI).


## Slide 9

### 7. Event Driven Architecture


Asynchronous communication based on events. 
Examples: IoT systems, real-time analytics.


## Slide 10

### Cloud Computing: Origins


Computer Scientist John McCarthy in 1961: “If computers of the kind I have advocated become the computers of the future, then computing may someday be organized as a public utility just as the telephone system is
... The computer utility could become the basis of a new and important industry.”
	1963: J.C.R. Licklider, a director at Advanced Research Projects Agency (ARPA), introduced the concept of Intergalactic Computer Network.
	1960 Leonard Kleinrock (@ARPANET): “As of now, computer networks are still in their infancy. But as they [...] become more sophisticated, we will probably see the spread of computer utilities.”


## Slide 11

### Cloud Computing: Origins


(Fast Forward 30 years):
	Late 1990s: Salesforce.com pioneered remotely provisioned services.
	2002 Amazon.com launched Amazon Web Services (AWS) platform.
	2006: The term Cloud Computing emerged commercially. Amazon launched Elastic Compute Cloud (EC2).


2009: Google offered Google App Engine.


## Slide 12

### Technology Innovations that inspired Cloud Computing


Virtualization: enabled to decouple applications from the underlying hardware, drivers, OS. Enabled sharing of the underlying processing capabilities by multiple users that need their own platform/hardware requirement. Enabled maintaining images for a standardized service and spawning as many copies of them as demanded, . . .


## Slide 13

### Essential Characteristics of
Cloud Computing


NIST lists 5 essential chatacteristics (defining properties) for cloud computing:


Ubiquitous (Broad) Network Access
On-demand Self-Service
Rapid Elasticity or Expansion
Resource Pooling
Measured Service


## Slide 14

### Essential Characteristics


Ubiquitous Network Access:
The property that the cloud service (the IT resource that it provides) be widely accessible.
This requires the cloud to support a range of transport protocols, interfaces, and security technologies.


## Slide 15

### Essential Characteristics


On-demand Self-Service:
 The property that the cloud consumer can access and scale the offered cloud-based resources independently (through a graphical or command-line interface) and be able to control them at will, with no need for any decision-input/approval by the cloud provider or any disruptive impact on the cloud’s operation. 
The cloud should also have the option to automate the process of resource provisioning whenever needed (at the runtime).


## Slide 16

### Essential Characteristics


Rapid Elasticity:
Elasticity (scalability):
the degree to which a system is able to adapt to workload changes by provisioning and de-provisioning resources in an autonomic manner, such that at each point in time the available resources match the current demand as closely as possible.1
There are two types of scaling:
  Horizontal: Allocation/Releasing of resources of the same type, e.g. nodes, servers, instances, etc.
  Vertical: Adding/removing capacity to/from (or replacing) an existing IT resource.


.


1Herbst, Nikolas Roman; Samuel Kounev; Ralf Reussner.
Elasticity in Cloud Computing: What It Is, & What It Is  Not ICAC 2013.


## Slide 17

### Essential Characteristics


Horizontal Scaling: Scaling Out / Scaling In


Vertical Scaling Scaling Up / Scaling Down


## Slide 18

### Essential Characteristics


Resource Pooling:

 Aggregation of computing resources such as storage, processing, memory, and network bandwidth across different physical machines so that they can be used by multiple users and dynamically assigned and released per demand.


## Slide 19

### Essential Characteristics


Measured Service:

 Cloud systems transparently monitor, control and report the usage of resources (e.g., storage, processing, bandwidth, and active user accounts) according to the abstraction level of the service, providing transparency to both the provider and consumer of the service.


## Slide 20

### Useful terminologies


Q: What are IT resources?
A: Any asset that provides an IT service. This includes physical/virtualized resources like 
Servers (computers where applications reside), Databases (computers where data resides in a structured way), Workstations or Clients (where the applications to communicate with and use the servers reside), network devices (access points, switches, routers, bridges, backbone, backhaul), or software (firmware, Drivers, OS, middleware, Applications), or human resources (IT/mgmt expertise)
Q:What’s the opposite of a cloud-based resource? 
A: On-Premise: in contrast to the “cloud”–based resources are remotely provisioned, and are scalable and measured, the conventional non-cloud based resources are called on-premise (in that they are located on the “premises” of the IT enterprise).


## Slide 21

### Cloud Service Models


NIST lists out three service (delivery) models based on the abstraction level:
Infrastructure as a Service (IaaS) 
Platform-as-a-Service (PaaS)
Software-as-a-Service (SaaS)


## Slide 22

### Main Service Models


## Slide 25

### Main Service Models: IaaS


The NIST definition of Infrastructure as a Service (IaaS):
“The capability provided to the consumer is to provision processing, storage, networks, and other fundamental computing resources where the consumer is able to deploy and run arbitrary software, which can include operating systems and applications. The consumer does not manage or control the underlying cloud infrastructure but has control over operating systems, storage, and deployed applications; and possibly limited control of select networking components (e.g., host firewalls).”


2Mell, Peter, and Tim Grance. ”The NIST definition of cloud computing.” (2011): http://nvlpubs.nist.gov/nistpubs/Legacy/ SP/nistspecialpublication800-145.pdf


## Slide 26

### Main Service Models: IaaS


So the main resources being provided:
Servers (with CPU, Cache, RAM, etc)
Disk Storage 
Networking.
In particular, the consumer has to install the OS, the application server (database server, the web server), job scheduling, resource usage governance, scaling, auto-scaling, caching,
middle-ware (messaging between applications), distributed computing, compilers/builders, deployment and integration tools, all the necessary runtime libraries, add-ons and plug-inns, and of course, their applications (including their application security/API/admin/etc).

What they get is a virtually “infinite” pool of computing resources that they can use-per-need and pay-per-use.


## Slide 27

### Main Service Models: IaaS


Instagram: a success story of IaaS:
Three programmers on a bootstrap budget launch a photo-sharing application, 25,000 sign up the first day, 40 million in less than 2 year, when facebook acquired it for ∼ 1$ Billion.3
Another good-use example:
When a consumer wants to build and deploy different server configurations, “test” and decide among them (even as an on-premise solution). So instead of hundreds of thousands of pounds this can be done in couple of tens.


3As of Sep 2017 it was about 600 million active users monthly:
https://www.statista.com/statistics/253577/ number-of-monthly-active-instagram-users/


## Slide 28

### Main Service Models: IaaS


Some currently well-known IaaS vendors:
Amazon Elastic Compute (EC2), Microsoft Azure, IBM Softlayer, Google Compute Engine, RackSpace Cloud Servers, vmware, CloudSigma, GoGrid, Joyent


But consumers can also build their own IaaS capabilities in-house (i.e., a private cloud) using OpenStack (say, to avoid vendor lock-in)


## Slide 33

### Main Service Models: PaaS


NIST’s definition of PaaS:4
“The capability provided to the consumer is to deploy onto the cloud infrastructure consumer-created or acquired applications created using programming languages,
libraries, services, and tools supported by the provider. The consumer does not manage or control the underlying cloud infrastructure including network, servers, operating systems, or storage, but has control over the deployed applications and possibly configuration settings for the application-hosting environment.”


4Mell, Peter, and Tim Grance. ”The NIST definition of cloud computing.” (2011): http://nvlpubs.nist.gov/nistpubs/Legacy/ SP/nistspecialpublication800-145.pdf


## Slide 34

### Main Service Models: PaaS


PaaS provides a platform to build, run, deploy and manage applications on the cloud.

Unlike in IaaS, the PaaS vendors take care of tasks like: 
automatic scaling based on traffic/usage load (scaling up/down and/or scaling out/in), which involves mechanisms like caching, asynchronous messaging, database scaling, etc., as well as providing the OS, 
the compilers/builders, the web/database servers, runtime libraries, testers, backups, deployment tools, etc., 
so that the consumers (developers) focus on the business logic and not re-inventing the wheel in implementing these IT-“plumbing” requirements.


## Slide 35

### Main Service Models: PaaS


Some famous PaaS providers (vendors):
Amazon Web Services (AWS), Redhat OpenShift, Microsoft Azure, , Google App Engine, Heroku Enterprise, Forece.com and AppExchange by salesforce, SAP Hana, CloudBees, Engine Yard, Centurylink AppFog, IBM Bluemix. . .


Besides these, tools like Dokku and OpenShift allow enterprises to build their own private PaaS.


## Slide 36

### Main Service Models: PaaS


In return for agility and speed to market, consumers are constrained by the tools and software stacks that the vendor offers.
They also give up a degree of control: e.g., they cannot manage memory allocation, stack configuration, number of threads, amount of cache, patch levels, etc.
This means, e.g., that scalability is not as flexible as an IaaS solution (e.g. in the case of Instagram, they went with AWS, although the famous platform for Python was Google Apps Engine).
Mature PaaS vendors also offer third-party add-ons (plug-ins, or extensions). These frees the developers from re-invent the wheel or obtaining and renewing licenses, install, patch, etc. and only pay-per-use:
Logging, Monitoring, Security, Caching, Search, E-mail, Analytics, Payments, etc.


## Slide 37

### Main Service Models: SaaS


NIST’s definition of SaaS:5
“The capability provided to the consumer is to use the provider’s applications running on a cloud infrastructure. 
The applications are accessible from various client devices through either a thin client interface, such as a web browser (e.g., web-based email), or a program interface (API). 
The consumer does not manage or control the underlying cloud infrastructure including network, servers, operating systems, storage, or even individual application capabilities, with the possible exception of limited user-specific application configuration settings.”


5Mell, Peter, and Tim Grance. ”The NIST definition of cloud computing.” (2011): http://nvlpubs.nist.gov/nistpubs/Legacy/ SP/nistspecialpublication800-145.pdf


## Slide 38

### Main Service Models: SaaS - popular


Low initial cost: If you want to develop your custom software, it is definitely going to cost you a fortune and will burn your pockets deep if you were to add some complex functionalities. 
Reduced Time:- SaaS applications can save a tremendous amount of your time because all the updates and improvisations are offloaded to the provider. So, you can focus on your core business without being worried about the recent technological advancements. 
Scalability:- This is another imperative feature of the SaaS model for small businesses that face frequent transitions in their process. As all SaaS applications are subscription-based, you have the flexibility to change your plan according to your needs. 
Try and use:- Most of the premium brands and providers always offer a free trial to use their SaaS app first. Hence, pay only if it solves your problem and satisfies your needs, else just move away and find the alternative without wasting a single penny, as simple as that.


## Slide 39

### Main Service Models: SaaS


Some very common SaaS applications are: Customer Relationship Management (CRM), Enterprise Resource Planning (ERP), payroll, accounting, video-conferencing, emails, etc.


There are thousands of SaaS providers, but the big money-makers serve enterprises.


## Slide 40

### Other Cloud Service Models


These main service models need to be updated, as new models are invented:


Container-as-a-Service (CaaS)
Business Process as a Service (BPaaS)
Information-as-a-Service (InaaS)


## Slide 41

### Deployment Models


NIST lists the following 4 deployment models:6
Private: The cloud infrastructure is provisioned for exclusive use by a single organization comprising multiple consumers (e.g., business units). It may be owned, managed, and operated by the organization, a third party, or some combination of them, and it may exist on or off premises.
Q: Which enterprises are likely to use a private cloud? And why?


6Mell, Peter, and Tim Grance. ”The NIST definition of cloud computing.” (2011)


## Slide 42

### Deployment Models


NIST lists the following 4 deployment models:6
Private: The cloud infrastructure is provisioned for exclusive use by a single organization comprising multiple consumers (e.g., business units). It may be owned, managed, and operated by the organization, a third party, or some combination of them, and it may exist on or off premises.
Q: Which enterprises are likely to use a private cloud? And why?
  A: Those with deep pockets and concern for security/privacy or strict regulations (government, finance, healthcare), e.g. Data Protection Act (DPA), Service Organization Control (SOC), Payment Card
	Industry Data Security Standard (PCI DSS), etc.
6Mell, Peter, and Tim Grance. ”The NIST definition of cloud computing.” (2011)


## Slide 43

### Deployment Models


Community: The cloud infrastructure is provisioned for exclusive use by a specific community of consumers from organizations that have shared concerns (e.g., mission, security requirements, policy, and compliance considerations). It may be owned, managed, and operated by one or more of the organizations in the community, a third party, or some combination of them, and it may exist on or off premises.


Examples of such communities: all banks of a financial zone, all hospitals of a city, all universities of a country, etc.


## Slide 44

### Deployment Models


Public: The cloud infrastructure is provisioned for open use by the general public. It may be owned, managed, and operated by a business, academic, or government organization, or some combination of them. It exists on the premises of the cloud provider.


## Slide 45

### Deployment Models


Hybrid: The cloud infrastructure is a composition of two or more distinct cloud infrastructures (private, community, or public) that remain unique entities, but are bound together by standardized or proprietary technology that enables data and application portability (e.g., cloud bursting for load balancing between clouds).
