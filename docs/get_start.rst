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

Methodology for booting virtual nodes
------------------------------------------------

There are three types of device for booting virtual nodes, which are network (pxe), disk and cdrom. We can modify the ``boot_order`` in YAML configuration file (The default configuration for OVA is `default.yml <https://github.com/InfraSIM/tools/blob/master/packer/scripts/infrasim.yml>`_, and the default path is ``~/.infrasim/.node_map/default.yml``) or send ipmitool command to choose the device for booting.

Booting from network
~~~~~~~~~~~~~~~~~~~~~

You can set the ``boot_order`` as ``n`` then start the node::

    set the boot_order = n in the YAML configuration file
    sudo infrasim node start

or send the ipmitool command after the node start like the following::

    sudo infrasim node start
    ipmitool -H 127.0.0.1 -U admin -P admin chassis bootdev pxe
    ipmitool -H 127.0.0.1 -U admin -P admin chassis power off
    ipmitool -H 127.0.0.1 -U admin -P admin chassis power on

Booting from disk
~~~~~~~~~~~~~~~~~~

Here you need a disk image file for booting first. Then add this disk image file path as a parameter ``file`` in YAML configuration file like the following::

    48     storage_backend:
    49         #Set drive list and define drive attributes
    50         -
    51             controller:
    52                 type: ahci
    53                 max_drive_per_controller: 8
    54                 drives:
    55
    56                 -
    57                     #Set node disk size, the unit is GB.
    58                     #The default value is 8GB
    59                     #
    60                     size: 8
    61                     # Add the disk image file path here
    62                     file: [disk image file path]

Then set the ``boot_order`` as ``c`` then start the node::

    set the boot_order = c in the YAML configuration file
    sudo infrasim node start

or send the ipmitool command after the node start like the following::

    sudo infrasim node start
    ipmitool -H 127.0.0.1 -U admin -P admin chassis bootdev disk
    ipmitool -H 127.0.0.1 -U admin -P admin chassis power off
    ipmitool -H 127.0.0.1 -U admin -P admin chassis power on

Booting from cdrom
~~~~~~~~~~~~~~~~~~~~~~

There are two ways to boot from cdrom. Both need to add the iso file path in the YAML configuration file to give the iso file to qemu. The default configuration for OVA is `default.yml <https://github.com/InfraSIM/tools/blob/master/packer/scripts/infrasim.yml>`_ and the default path is ``~/.infrasim/.node_map/default.yml``. The first one is giving the iso file to qemu directly, that is, an iso file is needed. The second one is editing settings in VMware to give the iso file to qemu.

#. Steps for the first way

    Here you need an iso file for booting first and add this iso file path in YAML configuration file. You can add the parameter ``cdrom`` in the YAML configuration file like the following::

        73             network_mode: bridge
        74             network_name: br1
        75             device: e1000
        76      # Add the iso file path here
        77      cdrom: [iso file path]
        78 bmc:
        79     interface: ens192

#. Steps for the second way

    This way is only for VMware environment. You need to get an iso file by editing settings in VMware and then add the iso file path in the YAML configuration file.

    * Edit settings in VMware::


        a. Choose “edit settings” to enter the “Virtual Machine Properties” page;
        b. Click on “CD/DVD drive1”;
        c. Browse and choose an ISO file in “Datastore ISO File”;
        d. As for the “Device Status”, check “Connected” and “Connect at power on”;
        e. Click on “OK” to save the change.

    * Modify the YAML configuration file::

        73             network_mode: bridge
        74             network_name: br1
        75             device: e1000
        76      # Add the iso file path here
        77      cdrom: /dev/sr0
        78 bmc:
        79     interface: ens192


After either way, set the ``boot_order`` as ``d`` then start the node::

    set the boot_order = d in the YAML configuration file
    sudo infrasim node start

or send the ipmitool command after the node start like the following::

    sudo infrasim node start
    ipmitool -H 127.0.0.1 -U admin -P admin chassis bootdev cdrom
    ipmitool -H 127.0.0.1 -U admin -P admin chassis power off
    ipmitool -H 127.0.0.1 -U admin -P admin chassis power on

