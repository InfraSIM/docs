Getting started
=========================

This chapter describes how to access virtual server, virtual PDU and virtual infrastructure provided by InfraSIM.

Quick start of infrasim-compute application
------------------------------------------------

Command interfaces
~~~~~~~~~~~~~~~~~~~~~

#. Initialization (you need do it once) ::

    sudo infrasim init

#. Start Infrasim Service::

    sudo infrasim node start

   Verify your service by `VNC and IPMI <startInterface_>`_

#. Status and version number check::

    sudo infrasim node status
    sudo infrasim version

#. Stop Infrasim Service::

    sudo infrasim node stop

#. Start IPMI Console::

    sudo ipmi-console start

#. Stop IPMI Console::
   
    sudo ipmi-console stop

.. _startInterface:

Interface to access virtual server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Server graphic UI
    VNC service is available through port **5901**. You can see the virtual monitor is already running and listing boot devices of virtual node. Through this booting devices, you can deploy hypervisor or operating system into virtual compute node just like operating on one physical server

	  .. image:: _static/vnc.png

#. Virtual BMC

   * Install ipmitool on host machine.::

		sudo apt-get install ipmitool

    IPMI over LAN::

		ipmitool -I lanplus -U admin -P admin -H <IP address> sdr list

    .. note:: <IP address> is address of NIC assigned to BMC access in YAML configuration file

    IPMI over internal path (vKCS) which requires OS and ipmitool application deployed inside virtual server::

        ipmitool sdr list

    You can get the command result like the following ::

		Pwr Unit Status  | Not Readable      | ns
		IPMI Watchdog    | Not Readable      | ns
		FP NMI Diag Int  | Not Readable      | ns
		SMI TimeOut      | Not Readable      | ns
		System Event Log | Not Readable      | ns
		System Event     | Not Readable      | ns
        ...

  #. Serial over LAN

    It requires activate SoL through IPMI command and console running IPMI console will becomes serial console of virtual server. After InfraSIM services started, this command is to activate SoL::

      sudo ipmitool -I lanplus -U admin -P admin -H localhost sol activate
      [SOL Session operational.  Use ~? for help]


.. hide_content::

            Virtual Power Distribution Unit - Robert - Under construction
            ------------------------------------------------

            Current Virtual PDU implementation only supports running entire virutal infrastructure on VMWare ESXi because it only supports functionality of simulating power control chassis through VMWare SDK.


Setup an InfraSIM Virtual Server on ESXi
---------------------------------------------------------

To setup an InfraSIM Server on ESXi, you should have an OVA with necessary environment prepared. You can consult the InfraSIM team to get the image or build one with the `packer build image <https://github.com/InfraSIM/tools/blob/master/packer/README.md>`_. Below are the steps to deploy and run InfraSIM on ESXi: 

    #. Get ESXi environment prepared by following `instruction <how_to.html#how-to-install-vmware-esxi-on-physical-server>`_
    #. Spin up a virtual machine by choosing "Deploy OVF Template". Specify the URL of the OVA image.
    #. Map the networks used in the OVA. The networking configured inside OVA is multi-bridge mode:

            * .. image:: _static/networking_bridge_multiple.PNG
                :align: center

    #. Modify YAML configuration file as you need. The default configuration for OVA is `infrasim.yml <https://github.com/InfraSIM/tools/blob/master/packer/scripts/infrasim.yml>`_. The path is::

           ~/.infrasim/.node_map/.default.yml

    #. Kick off all InfraSIM `services <get_start.html#command-interfaces>`_.

    #. Done, enjoy this virtual server!

.. note:: No need to run **infrasim-init** because it's already done during image build.

Configuration for OVA can be refered on `Packer OVA Configuration <https://github.com/InfraSIM/tools/blob/master/packer/infrasim-vmware.json>`_. Below are the major parameters::

    Disk Size: 40G
    Memory: 8G
    Number of CPUs: 2
    Number of NICs: 4
    Type of NICs: VMXNET 3
    NIC0:
        Name: ens160
        networkName: ADMIN
    NIC1:
        Name: ens192
        networkName: BMC
    NIC2:
        Name: ens224
        networkName: CONTROL
        Promiscuous Mode: on
    NIC3:
        Name: ens256
        networkName: DATA
        Promiscuous Mode: on

Setup an InfraSIM Virtual Server in VirtualBox 
---------------------------------------------------------

Virtualbox is available on multiple platforms. To get an InfraSIM BOX image, refer to `packer build image <https://github.com/InfraSIM/tools/blob/master/packer/README.md>`_

   #. Install virtualbox on the host.
   #. Create a directory for the VM and move the BOX image along with `Vagrantfile <https://github.com/InfraSIM/tools/blob/master/packer/Vagrantfile>`_ under the directory. 
   #. CD to the directory and run commands::

         vagrant box add --name infrasim-compute <YOUR_BOX_IMAGE>
         vagrant up
         vagrant ssh

   #. Modify YML configuration if you need. 
   #. Start InfraSIM `services <get_start.html#command-interfaces>`_. No **"infrasim-init"** needed. 

BOX configuration can be refered on `Packer BOX Configuration <https://github.com/InfraSIM/tools/blob/master/packer/infrasim-box.json>`_ and `Vagrantfile <https://github.com/InfraSIM/tools/blob/master/packer/Vagrantfile>`_. The major parameters are::
    
    Disk Size: 40G
    Memory: 5G
    Number of CPUs: 2
    Number of NICs: 4
    NIC0:
        Name: enp0s3
        Network Adapter: NAT
    NIC1:
        Name: enp0s8
        Network Adapter: Internal Network
    NIC2:
        Name: enp0s9
        Network Adapter: Internal Network
        Promiscuous Mode: on
    NIC3:
        Name: enp0s10
        Network Adapter: Bridged Adapter
        Promiscuous Mode: on

Relationship table of Infrasim command and standard server command
--------------------------------------------------------------------

Here we list a table to reflect the operations on physical server and the corresponding InfraSIM command. Note that the InfraSIM command with (*) here is not the CLI command. Use "infrasim -h" can get the help message.

    +----------------------------------------------+-----------------------------------------------------------------------------------+
    |standard server command                       |InfraSIM command                                                                   |
    +==============================================+===================================================================================+
    |AC power on a-node                            |infrasim node start a-node                                                         |
    +----------------------------------------------+-----------------------------------------------------------------------------------+
    |AC power off a-node                           |infrasim node stop a-node                                                          |
    +----------------------------------------------+-----------------------------------------------------------------------------------+
    |dismiss server node a-node                    |infrasim node destroy a-node                                                       |
    +----------------------------------------------+-----------------------------------------------------------------------------------+
    |reset a-node                                  |infrasim node restart a-node                                                       |
    +----------------------------------------------+-----------------------------------------------------------------------------------+
    |Check server a-node specification             |infrasim node info a-node                                                          |
    +----------------------------------------------+-----------------------------------------------------------------------------------+
    |Check server a-node running status            |infrasim node status a-node                                                        |
    |                                              |(If you see "a-node-bmc is running", it indicates AC is on, bmc is alive.          |
    |                                              |If you see "a-node-node is running", it indicates the compute node is powered on)  |
    +----------------------------------------------+-----------------------------------------------------------------------------------+
    |KVM - virtual keyboard, visual monitor        |Connecting to InfraSIM with VNC client(*)                                          |
    +----------------------------------------------+-----------------------------------------------------------------------------------+
    |configuration update for a-node               |1. update a-node yaml file(*)                                                      |
    |(node type, nic, processor, drive, memory)    |2. infrasim config update a-node [a-node yaml file path]                           |
    |                                              |3. infrasim node stop a-node                                                       |
    |                                              |4. infrasim node destroy a-node                                                    |
    |                                              |5. infrasim node start a-node                                                      |
    +----------------------------------------------+-----------------------------------------------------------------------------------+
    |add new server node b-node                    |1. compose b-node yaml file(*)                                                     |
    |                                              |2. infrasim config add b-node [b-node yaml file path]                              |
    |                                              |3. infrasim node start b-node                                                      |
    |                                              |4. infrasim config list                                                            |
    +----------------------------------------------+-----------------------------------------------------------------------------------+
