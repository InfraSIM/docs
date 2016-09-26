Features
=========================

**Bryan - Under construction**

Play with InfraSIM
--------------------------------------------

After virtual node, or a virtual rack is deployed, you can start to play with InfraSIM, either develop or validate your software on top of it.

#. Chassis management and hardware failure simulation. If the software application you're working on has logic designed to deal with server enclosures, for example, discovering, cataloging and monitoring every server node and related chassis, below commands are able to manipulate all chassis properties and generating hardware failures through virtual BMC module:    

   Please play with InfraSIM IPMI_SIM data by accessing `How to access vBMC data <userguide.html#access-vbmc-data>`_


#. Virtual PDU functionality are able to setup and simulate one power distribution network so that software developers don't have to pile up those physical PDUs, do cabling among server nodes, etc.    
	Please access `vPDU Node and Control <userguide.html#vpdu-deployment-and-control>`_ Section 3,4,5,6 for more information.

#. Operating system and hypervisor installation. All these software could be easily deployed on top of these simulated server nodes.  
    InfraSIM supports using different booting device, optical disk, hard disk drive, network device to boot into and install many operating systems and hypervisors. Then software developer could start developing and validating their application without noticing they're working with virtual hardware.    
