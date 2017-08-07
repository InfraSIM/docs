Features
=========================


Bare-metal server simulation
-----------------------------------

Here's a list of physical servers that InfraSIM has simulation support:

* Dell R730XD, R630 and C6320
* Quanta T41, D51
* Intel S2600KP, S2600TP and S2600WTT

Below list all the functionalities, regarding to how InfraSIM simulates behaviors, properties of those physical server.

Virtual BMC
~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Defining channel number and support internal and remote IPMI command

    * Virtual BMC is interally accessible (similar with a virtual KCS path) through software applications - typically ipmitool - running inside virtual server
    * Virtual BMC supports IPMI over LAN
    * Supports ipmitool "-t" option and specify access channel

#. FRU, Sensors, SDR, LAN, User

    * FRU, Sensor and SDR data - simulating what corresponding physical server is presenting
    * Define Chassis/Node relation by customizing, Chassis s/n and Node slot information

#. Chassis Control

    * Power control and power status monitoring
    * Connecting to virtual host to really simulate power control behavior of a physical server

#. IPMI master read/write to simulate I2C device 

    * Define and inject data for IPMI master read/write of particular I2C device

#. SEL

    * System power up event generating in SEL
    * SEL event generating on clear operation
    * SEL event generating on sensor reading beyond threshold
    * Inject SEL entry based on sensor event
    * Inject SEL entry based on OEM-defined format

#. Sensor data manipulating and injecting

    * Sensor readings dynamically change
    * Manually specify sensor (analog type) reading at run time 
    * Manually specify sensor (discrete type) reading at run time 


#. Supports changing boot order, activate/de-activate server Serial-Over-Lan

#. Specify NIC to transfer data for IPMI over LAN


Virtual network interface controller
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Add, remove NIC for virtual server
#. Randomly generating MAC address for each NIC to prevent duplication
#. Supports NAT, bridge and MACVTAP modes


Virtual host
~~~~~~~~~~~~~~~~~~~~~~~~

#. Support booting from PXE, ISO, HDD
#. SMBIOS data capturing and injecting
#. Define processor, memory properties


Virtual direct-attached storage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Specify drive properties:
    * SSD or spinning drive
    * Serial number
    * Physical information
    * Enable RAM disk to boost virtual disk drive performance 
    * Page data injection


Intelligent power distribute unit simulation
------------------------------------------------

InfraSIM has simulation for 2 types of PDU: Panduit PDU and Server Tech PDU. So far it only supports powering control virtual servers running on top of VMWare ESXi. All supported features include:

#. SNMP interface for management and control
#. Telnet/SSH Service to configure virtual PDU
#. Authentication
#. Retrieve telemetry data
#. Virtual server and outlet binding 
#. Power control virtual node hosted by ESXi
#. Notification / Trap
