Windows Azure IaaS
===

IaaS = Virtualization + Servers + Storage + Networking
VM Depot Gallery - predefined virtual machines available for choosing and running.
###Affinity groups
Affinity groups allow clustering virtual machines together so that they are physically close to each other.

### Endpoints + Availability sets

**Endpoint** is defined by the port on which should the machine listen for traffic and to which port on other machines the traffic should be flowing to.

**Availability set** - machines within an availability set must be replicated and provisioned as a whole in case of crash n burn.

Azure storage is accessible is available through many means of communication, not only through a web interface.

 - VPN + Cloud <-> Datacenter
 - full control over VM naming.
 - DNS services can be provided by Azure

### Costs & licencing

**Costs** - basically, M$ strives to keep prices beneath Amazon.

**licencing** - the pricing for Windows-based VMs includes the price of the OS licence. SQL and BizTalk prices are also included. **Application Licence Mobility** for other M$ applications.

### Management
 - Azure website
 - PowerShell + CLI
    - VM management automatization through scripting
 - REST API
 - System Center
    - Orchestrator: management tool used for systems tintegration and automatization of the process.
A full set of management tools are available within Microsoft System Center 2012.

Azure + Orchestrator
 - automatic provisioning and deprovisioning.

Windows Azure Compliance Programs
Windows Azure Trust Center
