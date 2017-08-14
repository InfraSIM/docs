User Guide
===============================================

This chapter will deep to the InfraSIM usage of virtual servers, virtual PDU.

Customizing virtual Server
--------------------------------------------

All supported virtual server configurations and properties of sub-component in that `central configuration file <configuration.html#virtual-server-configuration-file>`_. This sections describes key blocks and fileds in this YAML configuration file:

* Name of server node::

    name: node-1

* Node type which specify BMC configurations and behavior (server of specific model from specific vendor) and properties defeined in SMBIOS data. Implementation behind is specifying emulation data for vitual BMC and SMBIOS to load. Then ultimate, those IPMI command and dmidecode running on virtual server will get response exactly the same as what you can get from one physical server. By default it loads emulation data of Quanta D51 type::

    type: quanta_d51

* compute - This is one big block which contains several sub-block: storage, network, ipmi

    * Storage block is also arranged in an hierarchy way by storage_backend/controller/drives; for every single drive added, InfraSIM allows defining model/serial number/vendor/media/image file/page file::

        vendor: Hitachi
        model: HUSMM0SSD
        serial: 0SV3XMUA
        # To set rotation to 1 (SSD), need some customization
        # on qemu
        # rotation: 1
        # Use RAM-disk to accelerate IO
        file: /dev/ram0
        page-file: /directory/to/page_file_name.bin

* networks - defining network sub-system of virtual server. As below, 2 vmxnet3 type NICs are populated and connected to virtual switch br0::

        -
            network_mode: bridge
            network_name: br0
            device: vmxnet3
        -
            network_mode: bridge
            network_name: br0
            device: vmxnet3


 .. note:: Virtual bridge need to be created manually beforehand by using brctl utility


* ipmi - support specifying NIC (from host) attached and BMC credential and emulation data file::

        bmc:
            interface: br0
            username: admin
            password: admin
            emu_file: chassis/node1/quanta_d51.emu


Storage backend operation
--------------------------------------------

This sections describes storage backend operation supported by InfraSIM.

* Drive erasure.

  Drive erasure feature is implemented in Qemu code. After erasing, all data residing on a disk drive will be overwritten with all zero. Below are examples for SAS and SATA drive erase we experimented in Ubuntu 16.04.
    
  * SAS drive erasure.
    
     * First, install sg3-utils::
     
         apt-get install sg3-utils
     
     * Then, erase drive using sg3-utils::
     
         sg_format --format /dev/sd*
         
       *Note: Currently we support '-e', '-w' options.*
       
  * SATA drive erasure.
    
     * First, install hdparm::
     
         apt-get install hdparm
     
     * Then, erase drive with user password.
       
       Set security user password::
       
         hdparm --security-set-pass <PASSWD> /dev/sd*
      
       Perform drive erasure::
       
         hdparm --security-erase <PASSWD> /dev/sd*
       
       *Note: To disable security user password, please run below command*::
     
         hdparm --security-disable <PASSWD> /dev/sd*
     
     * Or, erase drive with master password.
       
       Set security master password::
       
         hdparm --user-master m --security-set-pass <PASSWD> /dev/sd*
       
       Perform drive erasure::
       
         hdparm --user-master m --security-erase <PASSWD> /dev/sd*
           
     
BMC run-time manipulating
--------------------------------------------------------

InfraSIM implemented one IPMI console which allows manipulating BMC behavior at run time; it can be treated as backdoor of virtual BMC which is particular useful when simulating chassis abnormal conditions and failures. It includes functionalities:

  * Update sensor reading with specified value, or cross-threshold value
  * Generate dynamicly-changing reading for specific sensor
  * Inject SEL entries for the particular sensors
  * Inject SEL entries for arbitry defined format

Here's instructions on how to use InfraSIM IPMI console:

  * Start ipmi console service by running command on host console::

      sudo ipmi-console start &
  
  * Enter IPMI_SIM by below command. <vbmc_ip> is localhost if you're run command in host, otherwise it is IP address of NIC specified in configuration file for ipmi to use. Prompt means successfull connection to ipmi console::

      ssh <vbmc_ip> -p 9300
      IPMI_SIM>

  * Enter help to check all the commands supported.::

      IPMI_SIM>help

  *  Below tables show the detail information about each command.

     .. list-table::
        :widths: 120 100
        :header-rows: 1

        * - Commands
          - Description
        * - sensor info
          - Get all the sensor information.
        * - sensor mode set <sensorID> <user>
          - Set the sensor mode to the user mode.
            Leaves the sensor reading as it currently is until instructed otherwise
        * - sensor mode set <sensorID> <auto>
          - Set the sensor mode to the auto mode.
            Changes the sensor reading to a random value between the lnc and unc thresholds every 5 seconds.
        * - sensor mode set <sensorID> <fault> <lnr | lc | lnc | unc | uc | unr >
          - Set the sensor mode to the fault mode.
            Changes the sensor reading to a random value to cause a particular type of fault as instructed (lnr, lc, lnc, unc, uc, unr)
                   lower non-recoverable threshold

                   lower critical threshold

                   lower non-critical threshold

                   upper non-critical threshold

                   upper critical threshold

                   upper non-recoverable threshold
        * - sensor mode get <sensorID>
          - Get the current sensor mode.
        * - sensor value set <sensorID> <value>
          - Set the value for a particular sensor..
        * - sensor value get <sensorID>
          - Get the value of a particular sensor.
        * - sel set <sensorID> <event_id> <'assert'/'deassert'>
          - Inject(Assert/Deassert) a sel error.
            You can use the sel set command to add a SEL entry for a particular sensor.
        * - sel get <sensorID>
          - Get the sel error for a sensor.
            You can use the sel get command to get the available events for a particular sensor.

  * Here's a example on how this console should be used and how it is chaning sensor readings. Let's prepare 2 terminal consoles: 1 for ipmi console and the other one is just normal console to use ipmitool to check how the manipulation works. 
  
    #. First lets check processor temperature of virtual server::

        sudo ipmitool -I lanplus -U admin -P admin -H localhost sensor get Temp_CPU0
            Locating sensor record...
            Sensor ID              : Temp_CPU0 (0xaa)
            Entity ID             : 65.1
            Sensor Type (Threshold)  : Temperature
            Sensor Reading        : 40 (+/- 0) degrees C
            Status                : ok
            Lower Non-Recoverable : na
            Lower Critical        : na
            Lower Non-Critical    : na
            Upper Non-Critical    : 89.000
            Upper Critical        : 90.000
            Upper Non-Recoverable : na
            Positive Hysteresis   : Unspecified
            Negative Hysteresis   : Unspecified
            Assertions Enabled    : unc+ ucr+ 
            Deassertions Enabled  : unc+ ucr+

    #. Then let's peek and poke this sensor reading from 40 degree C to 85 degree C in ipmi console::

            IPMI_SIM> sensor value get 0xaa
            Temp_CPU0 : 40.000 degrees C
            IPMI_SIM> 
            IPMI_SIM> sensor value set 0xaa 85
            Temp_CPU0 : 85.000 degrees C                 

    #. Last we can verify processor temerature sensor reading by issuing IPMI command again to check that sensor reading is really changed to 85 degree C::

        sudo ipmitool -I lanplus -U admin -P admin -H localhost sensor get Temp_CPU0
            Locating sensor record...
            Sensor ID              : Temp_CPU0 (0xaa)
            Entity ID             : 65.1
            Sensor Type (Threshold)  : Temperature
            Sensor Reading        : 85 (+/- 0) degrees C
            Status                : ok
            Lower Non-Recoverable : na
            Lower Critical        : na
            Lower Non-Critical    : na
            Upper Non-Critical    : 89.000
            Upper Critical        : 90.000
            Upper Non-Recoverable : na
            Positive Hysteresis   : Unspecified
            Negative Hysteresis   : Unspecified
            Assertions Enabled    : unc+ ucr+ 
            Deassertions Enabled  : unc+ ucr+ 


.. hide_content::

            vPDU deployment and control
            -----------------------------------

            #. vPDU deployment

            **Deploy vPDU Manually**
                The vPDU is part of the vCompute node. The vPDU has two network adapters. One is connected to the management network and used to communicate with the ESXi host. The other is connected to the internal network and used to communicate with the application you are testing.

                * Get the vPDU OVA file that you built when you deployed the virtual compute nodes.
                * Deploy the vPDU image on the vSphere client by click File -> Deploy OVF Template
                * Configure the vPDU network adapters as shown in the following picture.
                    .. image:: _static/vpdu.png
                        :height: 500
                        :align: center

                * Start the vPDU VM.
                * Click Open console to see the vPDU IP address.

            **Deploy vPDU by vRackSystem**
                Please access `vRackSystem User Manual <userguide.html#vracksystem>`_ for more information.

            #. Configuring the vPDU

            **Configure vPDU Manually**

            * On a server that has a network connection to the vPDU, use the SSH client to log in to the vPDU.::

                    ssh <ip address> -p 20022

            * When the (vPDU) prompt displays, specify the ESXi host information.::

                    config esxi add <esxi host ip> <esxi host username> <esxi host password>
                    config esxi update host <esxi host ip>
                    config esxi update username <esxi host username>
                    config esxi update password <esxi host password>

                Note: Use *config esxi list* to verify the settings.

            * Configure the eth1 IP address that is used to communicate with ESXi host.::

                    ip set eth1 <ip address> <net mask>

                Note: Use *ip get eth1* and *ip link eth1 status* to verify the settings.

            * Configure mappings between the VM and the vPDU port.
                Add mapping between the VM and the vPDU port.::

                    map add <datastore name> <VM Name> <vPDU number> <vPDU port>

                List the current mappings on vPDU.::

                    map list

                Delete a VM from a datastore::

                    map delete <datastore name> <VM Name>

                Update an existing mapping between VM and vPDU port::

                    map update <datastore name> <VM Name> <vPDU number> <vPDU port>

                Delete all VMs in a datastore::

                    map delete <datastore name>

            * Restart the vPDU service::

                vpdu restart


            **Configure vPDU by vRackSystem**
                Please access `vRackSystem User Manual <userguide.html#vracksystem>`_ for more information.


            #. Retrieve vPDU Service

            You can use SNMP commands to retrieve information about the PDU device::

                    snmpwalk -v2c -c ipia <vPDU IP Address> HAWK-I2-MIB::invProdFormatVer
                    snmpwalk -v2c -c ipia <vPDU IP Address> HAWK-I2-MIB::invProdSignature
                    snmpwalk -v2c -c ipia <vPDU IP Address> HAWK-I2-MIB::invManufCode
                    snmpwalk -v2c -c ipia <vPDU IP Address> HAWK-I2-MIB::invUnitName
                    snmpwalk -v2c -c ipia <vPDU IP Address> HAWK-I2-MIB::invSerialNum
                    snmpwalk -v2c -c ipia <vPDU IP Address> HAWK-I2-MIB::invFwRevision
                    snmpwalk -v2c -c ipia <vPDU IP Address> HAWK-I2-MIB::invHwRevision

            #. Verify the password

            You must verify the password before you can control the vPDU because the password is used for communication::

                snmpset -v2c -c ipia <vPDU IP Address> HAWK-I2-MIB::pduOutPwd.1.[Port] s [Password]

            The following table describes the information to include.

            .. list-table::
                :widths: 20 80
                :header-rows: 1

                * - Option
                    - Description
                * - Port
                    - The vPDU port number (Range: 1-24)
                * - Password
                    - The password you set for a specific port

            #. Power Up and Booting the vPDU

            Power on, power off, or reboot the vPDU::

                    snmpset -v2c -c ipia <vPDU IP Address> HAWK-I2-MIB::pduOutOn.1.[Port] i [Action]


            The following table describes the information to include.

            .. list-table::
                :widths: 20 80
                :header-rows: 1

                * - Option
                    - Description
                * - Port
                    - The vPDU port number (Range: 1-24)
                * - Action
                    - On, off, or reboot

            #. Retrieving the vPDU Port State
            Get the state of the vPDU port::

                snmpget –v2c –c ipia 172.31.128.244 HAWK-I2-MIB::pduOutOn.1.[Port]


vSwitch Setup
-----------------------------------

You can implement the vSwitch component of InfraSIM by deploying the Cisco Nexus 1000v switch on the ESXi host.

For more information on downloading and using Cisco Nexus 1000v switch, refer to http://www.cisco.com/c/en/us/products/switches/nexus-1000v-switch-vmware-vsphere/index.html.
