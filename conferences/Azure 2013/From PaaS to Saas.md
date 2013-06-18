From PaaS to SaaS
===

The topic of nk.pl comes up in terms of scaling the infrastructure out. They did not cut it.
PlaceChallenge (the presenter's company) uses Azure for infrastructural backbone.

Patterns for supporting which cloud computing would be appropriate:
 - going on and off
 - growing fast
 - unpredictable bursts
 - predictable (sequential) bursting

Abstracting away infrastructure maintenance and administration.
It is possible to define server usage/requirement scenarios - to match usage patterns.

Iaas -> host

PaaS -> build

SaaS -> consume

What we need to do ourselves is to provide the HDD image containing the OS and required software.
There is a possibility to choose from a range of predefined clean Windows/Linux machines that are made available by M$.
Preconfiguration is a non-issue in this case - Software as a Service is also available.

Azure has a **global footprint** - so that you can host your service in Asia, Europe, USA etc.
It is a matter of choice, making it a valuable option for global services.

**Traffic management** - traffic analysis is possible, allowing server location optimisation to achieve best performance for the largest group of users.

###Achieving high SLA
Clustering virtual machines.
Possibility to send VMs to another cloud hosting providers or retrieving them.
VHD format support is required.

The data is being replicated across Azure infrastructure matrices to provide data redundancy, useful in case of crash n burn situations.
Geo-replication in case somebody decides to bomb one data center.

###Azure websites
Basically, IIS in the cloud.
M$ provides 10 websites for free.

Azure Tools Service Package - can be sent to the cloud where it is unpacked and the contents are interpreted, which means preparing the infrastructure for the website being published.

Azure is reachable through VS by creating service packages from VS and uploading them into the cloud.

###Network Load Balancer
Load balancer polls the used servers to see whether they are responsive.
If not, it plugs the server off and creates an order for an equivalent server from the pool.

It checks for error HTTP status codes on port 80 by default. Creating custom tasks is possible (different ports, status conditions etc.)

Multiple database types are supported.
Blob storage, video, audio etc- all of it is going to be correctly replicated. The content is available from the geographic location nearest to the requesting user (additionaly paid feature)

Low-latency, in-memory distributed cache. 

User identification
 - integrate with enterprise identity
 - enable single sign-on within your apps
 - enterprise graph REST API

Service bus.

VIP Swap
 - a service package is sent to a staging server
 - a swap with the production server is performed.
 - there are multiple update policies available to choose from.


