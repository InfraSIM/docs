Software Architecture
=====================================

InfraSIM Components
-----------------------------------------

InfraSIM uses hypervisor - either VMWare ESXi or VMWare Workstation or KVM or VirtualBox or container(docker) - to host virtual elements of infrastructure. These virtual elements are implemented inside virtual machines and consists of the following components:

* **vCompute**
    The virtual node is used to simulate specific server node. The virtual node component is implemented within a virtual machine running on a hypervisor. 

    Each virtual node implemented a virtual BMC (vBMC) inside. All BMC functionalities such as sensor data, thresholds, power controls, and boot options are simulated with this module. Both local and remote IPMI command are fully supported by using popular IPMItool.

    There's one nested QEMU VM included, which is capable of simulating CPUs, DIMMs, and other hardware devices.


* **vPDU**
    The vPDU is simulating intelligent PDU which is used to control AC power of other virtual nodes.

* **vSwitch**
    The vSwitch is used to simulate network switches, including the connections to the virtual compute nodes within the virtual infrastructure.


Virtual Node
-----------------------------------------
The following diagram shows a high-level view of components in the virtual node architecture.



.. image:: _static/architecture1.png
     :align: center



vBMC is able to handle IPMI command from either external network or local virtual compute over vKCS interface, it is bridged to an external vSwitch, to be accessible to management network.


vCompute is nested QEMU VM. There are two virtual networks attached: one is connected to the same network as vBMC which allows traffic of DHCP, TFTP, PXE, etc; the other network is used as data network specifically for user work load.

The vNode could be running on most of popular hypervisors such as VMWare ESXi, VMWare Workstation, KVM, VirtualBox, as well as container (docker).


Virtual PDU
-----------------------------------------
The following diagram shows a high-level view of components in the virtual PDU architecture as well as shows how each component interacts with others.


.. image:: _static/vpdu_diagram.png
   :align: center

* SNMP Simulator

  The snmp simulator is to simulate SNMP protocol, to respond snmp requests from external sources. 
  It also supports parsing and responding according to definitions of a vendor-specific MIB data file, if you want to get more details of this simulator, please reference `snmpsim <http://snmpsim.sourceforge.net>`_

* vPDU Service
  
  The vPDU service will handle the messages from snmp simulator over pipe, and then call various control interface to power on, power off, reboot the virtual nodes.


* Control service

  The control service is an interface over SSH to configure vPDU such as ip address, simulated data settings, outlet settings etc.


Virtual Switch
-----------------------------------------
Regarding to vSwitch solution, infraSIM mainly leverages products from Hypervisor - for example VMWare vSwitch; or from vendor such as Cisco Nexsus 1000v, Arista vEOS.
