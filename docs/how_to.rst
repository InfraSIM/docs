How To
=========================

How to install VMWare ESXi on Physical Server
------------------------------------

#. Requirement of physical server
    The physical server must support ESXi 6.0 and it should be allocated at least 3 NIC ports. The first NIC port is used for the admin network connection. The second and third NIC ports are used for control network connection(The second NIC is required. The third NIC is optional). The fourth NIC port is used for data network connection (optional).

    Virtual InfraSIM servers runs in the best performance if hardware-assisting technology has been enabled on underlying physical machines. These technology includes VT-d feature and AMD-V for processors from Intel and AMD.

    .. note:: **Physical machine** - enable VT-d in BIOS

        .. image:: _static/configBIOSpng.png
            :height: 400
            :align: center

#. Setting Up Network Connections
    You must have IP addresses for the physical servers in the test environment to be used to configure the VMKernal port of ESXi and called as ESXi_Admin_IP.

    * Allocate or reserve a static IP address from the Lab admin.
    * Connect the server’s admin NIC ports into the Lab network.
    * To set up a multiple server environment, connect Port C1 on each server by using an Ethernet switch.

#. Install ESXi 6.0
    From the VMWare web site, a 60-day free trial version is available after user registration.

    * Go to https://my.vmware.com/web/vmware/details?downloadGroup=ESXI600&productId=490&rPId=7539
    * Download the VMWare vSphere Hypervisor 6.0 (ESXi6.0) ISO image.
    * Install ESXi 6.0 on each physical server.
    * Configure the static IP address ESXi_Admin_IP on first NIC port.
    * Set the Administrator user name by using the format <User Name>.
    * Set the Administrator Password by using the format <Password>.

#. Installing VMWare vSphere Client (Remote System)
    * Go to the VMWare web site.
    * Download the VMWare vSphere Client.
    * Install the client on a remote system that can connect to the physical servers.

#. Configuring the Virtual Network
    * Launch the vSphere client and connect to ESXi on the physical server by using ESXi_Admin_IP.

    * On the Configuration tab, click Add Networking, to create the Control vSwitch. In the example, the network label is "VM Network 2".
    
        .. image:: _static/virtualnetwork1.png
            :height: 400
            :align: center

    * Select Virtual Machine
    
        .. image:: _static/virtualnetwork2.png
            :height: 400
            :align: center

    * Select Create a vSphere standard switch > vmnic2.
    
        .. image:: _static/virtualnetwork3.png
            :height: 400
            :align: center

    * In the Network Label field, type port group name on target switch.
    
        .. image:: _static/virtualnetwork4.png
            :height: 300
            :align: center

    * Enable the SSH service on ESXi. To do this, open the Configuration tab and select Security Profile. Then select SSH and click Properties to set the SSH (TSM-SSH) to start and stop manually.

        .. note:: Login to the ESXi server through SSH and echo by issuing the **"vhv.enable = "TRUE""** command to the /etc/vmware/config file. This command enables nested ESXi and other hypervisors in vSphere 5.1 or higher version. This step only needs to be done once by using the command: echo 'vhv.enable = "TRUE"' >> /etc/vmware/config.
    
            .. image:: _static/ssh_ESXi.png
                :height: 300
                :align: center

        .. note:: Set **Promiscuous Mode** to Accept and tick Override. To do this, open the Configuration tab and select Networking. Then click Properties of the vSwitch, choose port group, edit, security, tick the checkbox to override setting and select Accept.    
    
            .. image:: _static/virtualnetwork5.png
                :height: 300
                :align: center


How to deploy InfraSIM virtual server on different type of platforms
--------------------------------------------------------------------------------------

This chapter will introduce how to build, deploy and run virtual compute node on top of KVM, Docker and VirtualBox.


Currently, hypervisors or platforms that InfraSIM could be deployed on top of are:
    -  `VirtualBox <https://www.virtualbox.org/>`_
    -  `KVM <http://www.linux-kvm.org>`_
    -  `Docker <https://www.docker.com>`_
    -  `VMWare product <https://www.vmware.com>`_, both VMWare vSphere or VMWare workstation

However, there should be corresponding images ready beforehand - deploying InfraSIM application into operating system running in virtual machines or containers on top of specified hypervisor or platform. These images are: OVA file for VMWare workstation or vSphere; QCOW2 file for KVM/QEMU; BOX or vagrant/VirtualBox, etc. Below listed some steps on how to deploy these template into different systems:  


Deploy virtual server VMWare Workstation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Please follow below steps to setup InfraSIM in VMWare workstation.

#. Configure the BIOS to enable virtualization.
    .. image:: _static/configBIOSpng.png
        :height: 400
        :align: center

#. Download VMWare Workstation 11.0 from http://www.vmware.com/products/workstation/workstation-evaluation and install it. (The VMWare Workstation is not free)

#. Configure a virtual network in the VMWare Workstation.
    * From the Windows Start menu, open "Virtual Network Editor".
        .. image:: _static/vmworkstation1.png
            :height: 450
            :align: center

    * Click "Add Network…" to add a new network VMnet2.
        .. image:: _static/vmworkstation2.png
            :height: 100
            :align: center

    * Clear the "Connect a host virtual adapter to this network" check box and un-check the "Use local DHCP service to distribute IP address to VMs", then set the Subnet IP to "172.31.128.0", and the subnet mask to "255.255.255.0".
        .. image:: _static/vmworkstation3.png
            :height: 450
            :align: center

#. Import and configure InfraSIM OVA images.
    * Import the InfraSIM OVA image to the VMWare Workstation by "File -> Open…", and then select the InfraSIM OVA image.
        .. image:: _static/vmworkstation4.png
            :height: 350
            :align: center

    * After the InfraSIM OVA image imported successfully, open the virtual machine settings to enable the virtualization engine, and change the number of processor and number of cores per processor.
        .. image:: _static/vmworkstation5.png
            :height: 450
            :align: center

    * Change the memory size to 1 GB.
        .. image:: _static/vmworkstation6.png
            :height: 450
            :align: center

    * Click Network Adapter, and connect the network adapter to "VMnet2" which was created in the previous step.
        .. image:: _static/vmworkstation7.png
            :height: 450
            :align: center



Here's introduction on how to deploy InfraSIM build on different hypervisors.

**Prerequisite**:

#. Clone `tool <https://github.com/InfraSIM/InfraSIM.git>`_ repository
#. Build the vnode image following the steps in `Build Generic Virtual Compute Node <builddeploy.html#build-generic-virtual-compute-node>`_ Section.

Run Docker-based VM
~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Install necessary packages.::

    #sudo apt-get install docker docker-engine

#. Build Docker-based Image.::

    # cd tools/docker_builder
    # sudo ./docker_image_builder.sh -d <your idic directory>/pdk/linux/vnode -t vnode

#. Start Docker-based VM.::

    # ./run_docker.sh -n vnode -t vnode


Run KVM-Based Virtual Machine
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Install necessary packages.::

     # sudo apt-get install virt-manager

#. Build KVM-based Image.::

	# cd tools/kvm_builder
	# sudo ./kvm_builder.sh -d <your idic directory>/pdk/linux/vnode -t vnode


   If success, you will see "vnode.qcow2" disk image under the directory.

#. Start KVM-based VM using qemu.::

     # ./start_kvm_vm.sh -n vnode

#. Use virt-manager to see the GUI interface
   You can see the running virtual machine in virt-manager

   .. image:: _static/virt-manager.png

Run Virtualbox-based VM
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We use `vagrant <https://www.vagrantup.com>`_ to create the Virtualbox-based VM.
Using vagrant, you can quickly create the Virtualbox-based VM on your laptop, work PC whatever your system is MacOS, Linux or even Windows.

**Notice:** We will assume that you have basic knowledge about how vagrant works and vagrant version should be > 1.8

#. Install necessary packages::

    #sudo apt-get install virtualbox

#. Build Virtualbox-based Image::

    # cd tools/virtualbox_builder
    # sudo ./virtualbox_builder.sh -d <your idic directory>/pdk/linux/vnode -n vnode

   You will see that the "vnode.box" is under this directory

#. Start Virtualbox-based VM::

    # vagrant box add --name vnode vnode.box
    # mkdir vnode && cd vnode
    # vagrant init vnode
    # vagrant up

   **Notice:** Because VirtualBox itself doesn't and has no plan to support `Nested Virtualization: VT-in-VT <https://www.virtualbox.org/ticket/4032>`_ feature, running Qemu inside VirtualBox can't enable KVM acceleration, which suffers a big performance penalty.

Run VMWare product based VM
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#. Refer to `How to build vNode and vPDU <how_tos.html#build-vnode-and-vpdu>`_ for building OVA image working for both ESXi and VMWare workstation.
#. Refer to `VMWare Workstation deployment <how_tos.html#vmware-workstation-deployment>`_ for deploying virtual node on VMWare workstation.
#. Refer to :ref:`setup-infrasim-on-esxi-label` for deploying virtual node on VMWare ESXi.


How to simulate another server
---------------------------------------------
In infraSIM source code repository, there are one generic virtual node type (vnode) and several other server nodes (Dell, Quanta servers) simulation provided for end-user under idic/vcompute/vnode. InfraSIM also provided many utilities, interfaces for developers to build one simulation solution for a physical node that has not been supported by infraSIM This sections walk through all steps required to build one simulation for one specific server node.

#. Create a new directory for your node. If you want to create your own vNode, copy the full directory's content from idic/vcompute/vnode under idic/vcompute directory::

    $ git clone <idic-repo-url>
    $ cd idic/vcompute
    $ cp -rap vnode <your-vnode-name>

#. clone the tools repo for future use::

    $ git clone <idic-repo-url>

#. After you create the directory, set your node name in Makefile::

    $ cd <your-vnode-name>

#. Edit "Makefile" file, set "TARGETNAME = <your-vnode-name>" to your vnode name

#. Edit ".config" file, set "CONFIG_HOSTNAME" to your vnode name

#. To simulate a real hardware server, you have to get the server fru' data::

    $ cd data

   Under this directory, you can find "vnode.emu" file. In this file, we keep server fru' data here, like::

    $ mc_add_fru_data 0x20 0x0 0x100 data \
      0x01 0x00 0x01 0x04 0x0f 0x00 0x00 0xeb \
      0x01 0x03 0x17 0x00 0xcd 0x51 0x54 0x46 \
      0x43 0x4a 0x30 0x35 0x31 0x36 0x30 0x31 \
      ......

   You can use ipmitool to get BMC sensor's data::

    $ ipmitool -U <your-account> -P <your-password> -I lanplus -H <your-BMC-IP> fru read <fru ID> fru.bin

   call fru_gen.py script to dump fru.bin to hex format::

    $ cp ../../tools/data_generater/fru_gen.py ./
    $ python fru_gen.py fru.bin

   fru_result will be generated, replace original fru data with the expected one in this file.

#. Same as fru, in "vnode.emu" file, we keep server sensors' data here, like::

    $ sensor_add 0x20 0x0 0x01 0x02 0x01
      main_sdr_add 0x20 \
      0x00 0x00 0x51 0x02 0x2a \
      0x20 0x00 0x01 0x15 0x01 0x67 0x40 0x09 0x6f 0x71 0x00 0x71 0x00 0x71 0x00 0xc0 \
      0x00 0x00 0x01 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0xcf 0x50 0x77 0x72 0x20 0x55 \
      0x6e 0x69 0x74 0x20 0x53 0x74 0x61 0x74 0x75 0x73
    $ sensor_set_value 0x20 0x0 0x01 0x0 0x1

    You can use ipmitool to get BMC sensor's data::

    $ ipmitool -U <your-account> -P <your-password> -I lanplus -H <your-BMC-IP> sdr dump sensors

   The above command will dump your server BMC sensors' data to the file named: "sensors"
   Generally, the sensor file contains binary data, we have to convert it to strings::

      $ cp ../../tools/data_generater/sensors_gen.sh ./
      $ ./sensor_gen.sh

   After the command, you will get the file named: "all_sdr_sensors"::

    Use "all_sdr_sensors" file content to replace "vnode.emu" file of all "sensor_add" sections
        **Notice: This step is not necessary for your node unless you want to emulate the real BMC sensors' data.**

#. SMBIOS data is also needed, which can be got by using the command::

    $ dmidecode --dump-bin <your-vnode-name>_smbios.bin

#. Build your vnode with real hardware fru, sensors and smbios data.::

    $ make <your-vnode-name>

#. Enjoy your customized node.

How to simulate another vPDU
---------------------------------------------------------
InfraSIM provided ServerTech and Panduit PDU simulation initially. InfraSIM also provided many utilities, interfaces for developers to build simulation solution for other physical PDUs. This sections walk through all steps required to build one simulation for other PDU infraSIM doesn't support yet.

#. How to retrieve data from physical PDU

   If you want to retrieve PDU MIB data, you should have `snmpsim <http://snmpsim.sourceforge.net>`_ installed on your environment.Then run the following command to produce MIB snapshot for the PDU::

   # snmprec.py --agent-udpv4-endpoint=<PDU IP address>; --start-oid=1.3.6 --output-file=/path/<target snmprec file>; --variation-module=sql --variation-module-options=dbtype:sqlite3,database:/path/<target pdu database file>,dbtable:snmprec


   For more details of how to use snmprec.py, please go to section `Producing SNMP snapshots <http://snmpsim.sourceforge.net/snapshotting.html>`_ at snmpsim home page for more help.

#. How to simulate physical PDU in InfraSIM

   Once you retrieved data from physical PDU, the next step is to add a virtual PDU in InfraSIM for this physical server. The following steps will guide you how to do:

   A. Create a directory named **PDU name** at idic/vpdu
   B. Create a directory data at idic/vpdu/<PDU name>/data, and copy the data you get from physical server into data directory.

   C. Copy .config and Makefile into idic/vpdu/<PDU name>, and update target name in Makefile and .config

   D. Clone `vpduserv <https://github.com/InfraSIM/vpduserv.git>`_, and implement the new pdu logic based on vendor's PDU spec.


How to integrate RackHD with InfraSIM
--------------------------------------------------

RackHD is an open source project that provides hardware orchestration and management through APIs. For more information about RackHD, go to http://rackhd.readthedocs.org/en/latest/.

The virtual hardware elements(virtual compute node, virtual PDU, virtual Switch) simulated by InfraSIM can be managed by RackHD.

The following picture shows the deployment model for the integration of InfraSIM and RackHD:

.. image:: _static/connections_RackHD.png
           :height: 600
           :align: center

Please follow up below steps to step the entire environment. After that, RackHD can discover and manage the virtual server and virtual PDU just as the real physical server and PDU. 

#. Please refer RackHD document(http://rackhd.readthedocs.org/en/latest/getting_started.html) to setup the RackHD Server. RackHD server should be configured with as least two networks, "Admin network" and "Control Network".
    * "Admin Network" is used to communicate with external servers
    * "Control Network" is used to control the virtual servers.

#. Deploy virtual compute node and virtual PDU on ESXi with the network connection as in the pictures show. The ESXi show have network connection with the RackHD server.

After you setup the environment successfully, you can get the server information and control the servers by RackHD APIs. More information about how RackHD APIs communicate with the compute server and PDU, Please refer http://rackhd.readthedocs.org/en/latest/rackhd/index.html#rackhd-api