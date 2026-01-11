# Lecture_1.1_Clouds Intro


## Slide 1

### Cloud Computing Intro


## Slide 2

### Intro to Cloud Computing: Agenda


Computing at scale
The need for scalability; scale of current services
Scaling up: From PCs to data centers
Problems with 'classical' scaling techniques 
Utility computing and cloud computing
What are utility computing and cloud computing?
What kinds of clouds exist today?
What kinds of applications run on the cloud?
Virtualization: How clouds work 'under the hood'
Some cloud computing challenges


2


Intro to Cloud Computing: Agenda


## Slide 3

### How many users and objects?


Flickr has 10 billion photos
Facebook has 1.4 billion daily active users
Google is serving 3.5 billion queries/day on over 130 trillion pages
5 billion videos/day watched on YouTube


3


How many users and objects?


## Slide 4

### How much data?


Modern applications use massive data:
Rendering 'Avatar' movie required >1 petabyte of storage
eBay has >6.5 petabytes of user data
CERN's LHC will produce about 15 petabytes of data per year
In 2008, Google processed 20 petabytes per day
German Climate computing center dimensioned for 60 petabytes of climate data
Google now designing for 1 exabyte of storage
NSA Utah Data Center is said to have 5 zettabyte 
How much is a zettabyte?
1,000,000,000,000,000,000,000 bytes
A stack of 1TB hard disks that is 25,400 km high


4


How much data?


## Slide 5

### No single computer can process that much data
Need many computers!
How many computers do modern services need?
Facebook is thought to have more than 60,000 servers
1&1 Internet has over 70,000 servers
Akamai has 95,000 servers in 71 countries
Intel has ~100,000 servers in 97 data centers
Microsoft reportedly had at least 200,000 servers in 2008 
Google is thought to have more than 1 million servers, is planning for 10 million


5


How much computation?


## Slide 6

### Why should I care?


Suppose you want to build the next Google
How do you...
... download and store billions of web pages and images?
... quickly find the pages that contain a given set of terms?
... find the pages that are most relevant to a given search?
... answer 1.2 billion queries of this type every day?

Suppose you want to build the next Facebook
How do you...
... store the profiles of over 500 million users?
... avoid losing any of them?
... find out which users might want to be friends?


6


Why should I care?


## Slide 7

### Why use a cloud?


Reliability
It’s someone else’s responsibility to fix broken machines

Cheap and On-Demand Scalability
Pricing is per hour or second instead of sunk hardware cost
Can create and destroy nodes on a per second basis
Many clouds (GCP and AWS) recently switched to per-second billing
Hardware Abstraction
Don’t have to care about underlying hardware, just the specs of your VM

“Special Sauce”
Proprietary features (i.e. AWS DynamoDB or Google BigQuery)


## Slide 9

### Intro to Cloud Computing: Agenda


Computing at scale
The need for scalability; scale of current services
Scaling up: From PCs to data centers
Problems with 'classical' scaling techniques 
Utility computing and cloud computing
What are utility computing and cloud computing?
What kinds of clouds exist today?
What kinds of applications run on the cloud?
Virtualization: How clouds work 'under the hood'
Some cloud computing challenges


9


Intro to Cloud Computing


## Slide 10

### Scaling up


What if one computer is not enough?
Buy a bigger (server-class) computer
What if the biggest computer is not enough?
Buy many computers


10


PC


Server


Cluster


Computing Size


## Slide 11

### Many similar machines, close interconnection (same room?)
Often special, standardized hardware (racks, blades)
Usually owned and used by a single organization


11


Many nodes/blades(often identical)


Network switch(connects nodes witheach other and with other racks)


Storage device(s)


Rack


Characteristics of a cluster:


## Slide 12

### Power and cooling


Example: 140 Watts per server
Rack with 32 servers: 4.5kW (needs special power supply!)
Most of this power is converted into heat
Large clusters need massive cooling
4.5kW is about 3 space heaters
And that's just one rack!


12


Clusters need lots of power


## Slide 13

### Scaling up


Build a separate building for the cluster
Building can have lots of cooling and power
Result: Data center


13


PC


Server


Cluster


Data center


What if your cluster is too big (hot, power hungry) to fit into your office building?


## Slide 14

### What does a data center look like?


A single data center can easily contain 10,000 racks with 100 cores in each rack (1,000,000 cores total)


14


Google data center in The Dalles, Oregon


Data centers(size of a football field)


Coolingplant


A warehouse-sized computer


## Slide 15

### What's in a data center?


15


Source: 1&1


Hundreds or thousands of racks


## Slide 16

### What's in a data center?


16


Source: 1&1


Massive networking


## Slide 17

### What's in a data center?


17


Source: 1&1


Emergency power supplies


## Slide 18

### What's in a data center?


18


Source: 1&1


Massive cooling


## Slide 19

### Energy matters!


Makes sense to build them near sources of cheap electricity
Example: Price per KWh is 3.6ct in Idaho (near hydroelectric power), 10ct in California (long distance transmission), 18ct in Hawaii (must ship fuel)
Most of this is converted into heat ® Cooling is a big issue!


19


Company | Servers | Electricity | Cost
eBay | 16K | ~0.6*105 MWh | ~$3.7M/yr
Akamai | 40K | ~1.7*105 MWh | ~$10M/yr
Rackspace | 50K | ~2*105 MWh | ~$12M/yr
Microsoft | >200K | >6*105 MWh | >$36M/yr
Google | >500K | >6.3*105 MWh | >$38M/yr
USA (2006) |  |  | 


Source: Qureshi et al., SIGCOMM 2009


10.9M


610*105 MWh


$4.5B/yr


Data centers consume a lot of energy


## Slide 20

### There are even bigger energy consumers:
http://motherboard.vice.com/read/bitcoin-could-consume-as-much-electricity-as-denmark-by-2020
2020 14 Gigawatts (Total power generation capacity of Denmark)
1 Bitcoin in 2020 would consume half the annual household usage emitting 4,000 kg of CO2
This is a big issue because of the enthusiasm for the Blockchain driving new functionality


20


Intro to Cloud Computing: Agenda


## Slide 21

### Scaling up


What if even a data center is not big enough?
Build additional data centers
Where? How many?


21


PC


Server


Cluster


Data center


Network of data centers


Intro to Cloud Computing: Agenda


## Slide 22

### Global distribution


Data centers are often globally distributed
Example above: Google data center locations (inferred)
Why?
Need to be close to users (physics!)
Cheaper resources
Protection against failures


22


Intro to Cloud Computing: Agenda


## Slide 23

### Trend: Modular data center


Need more capacity? Just deploy another container!


23


Intro to Cloud Computing: Agenda


## Slide 24

### Clouds Service models


“Private” Clouds
Used for a company’s internal services only
Example: Internal datacenters of companies like Facebook, Google, etc.

“Public” Clouds
Anyone can purchase resources
You can build your own company on top of another company’s cloud
Example: AWS, GCP, Azure


## Slide 25

### Cloud Providers


## Slide 26

### The Giants
