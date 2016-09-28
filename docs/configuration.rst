Configuration
=========================



Virtual Server Configuration file
------------------------------------------------

**Forrest - Under construction**

Networking
------------------------------------------------

#. Virtual server NAT or host-only mode, this is default mode implemented in infrasim-compute
    * vCompute is accessible ONLY inside Ubuntu host 
    * Software running in vCompute can access outside network if connecting Ubuntu host NIC with virtual bridge
    * Configuration YAML file can specify which NIC IPMI over LAN traffic flows through

    .. image:: _static/networking_nat.PNG
        :align: center

#. Bridge mode - single
    * Work as virtual switch
    * Connect BMC NIC and NICs in virtual compute together
    * Configuration YAML file controls how many NICs that virtual compute has and specify bridge they connect to

    .. image:: _static/networking_bridge_single.PNG
        :align: center

    .. note:: It requires setting up bridge and connect to NIC of underlying host in advance. 
    
    Here's steps for this example::

            # brctl addr br0
            # brctl addif br0 eth1
            # brctl setfd br0 0
            # brctl sethello < bridge name > 1
            # brctl stp br0 no
            # ifup br0

#. Bridge mode - multiple

    .. image:: _static/networking_bridge_multiple.PNG
        :align: center

Virtual Power Distribution Unit
------------------------------------------------

 Current Virtual PDU implementation only supports running entire virutal infrastructure on VMWare ESXi because it only supports functionality of simulating power control chassis through VMWare SDK.

 .. image:: _static/networkwithoutrackhd.png
    :align: center

**Robert - Under construction** 