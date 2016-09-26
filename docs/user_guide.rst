User Guide
===============================================

This chapter will deep to the InfraSIM usage of deploying large scale virtual infrastructure. If simply virtual compute nodes or small scale infrastructure already works, you can refer to `Quick Start <gettingstart.html>`_ to get that setup.

Many functionalities described in this chapter, such as vRackSystem for InfraSIM deployment and virtual PDU are supported only on top of `VMWare vSphere Client(ESXi) <https://www.vmware.com/products/vsphere>`_ as of now. In `Build, Package and Deployment <builddeploy.html>`_ chapter, we describe how to build and deploy virtual compute node on top of KVM, Docker, Virtual Box, VMWare workstation and ESXi.

**Notice:** Before you start this chapter, please follow the instructions in `Build vNode and vPDU <how_tos.html#build-vnode-and-vpdu>`_ to build the vNode and vPDU OVA images, which can be deployed on ESXi.


**Index of User Guide:**

#. :ref:`setup-infrasim-on-esxi-label`

   * How to install ESXi on physical server
   * Deploy and control vNode
   * Deploy and control vPDU
   * Access vBMC data
   * vSwitch setup

#. `vRackSystem User Manual <userguide.html#vracksystem>`_
#. `Test InfraSIM <userguide.html#puffer-infrasim-test>`_


Access vBMC Data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This section explains how to access the vBMC information and how to change the boot option of the virtual node. You can access information from the virtual BMC, change sensor values,
configure the sensor mode, add a SEL entry for a particular sensor, and monitor the virtual node BIOS/iPXE/OS/ runtime output instead of VNC.


#. Accessing BMC Static Data via ipmitool command
    Communicate with the BMC by using the ipmitool command. ipmitool commands for accessing BMC data have the following format::

      ipmitool –I <interface> –H <address> –U <username> –P <password> <expression>

    The following table describes the information to include.

    .. list-table::
       :widths: 20 80
       :header-rows: 1

       * - Option
         - Description
       * - -I <interface>
         - The IPMI interface to use. Use *-I lanplus** for commands that are described in this section.
       * - -H <address>
         - The IP address of the BMC.
       * - –U <username>
         - Admin user name.
       * - –P <password>
         - Admin password.
       * - <expression>
         - The operation to perform. For example.
             sensor: operate the available sensors

             fru : operate the available FRUs

             sdr: check the sdr info

             raw: send raw command to virtual BMC

             chassis: power off/power on/set bootdev of the system

             sel: check/clear the sel entries in the vBMC

             lan: check lan information of the vBMC

             User: check user information

#. Accessing Dynamic Sensor Data by IPMI_SIM
    InfraSIM support to access the dynamic sensor data by IPMI_SIM, it includes below functionalities:
       * Dynamic sensor reading
       * Ability to change sensor value
       * Generate sensor values across sensor thresholds
       * Inject SELs for the particular sensors.

    **Please follow below steps to play with InfraSIM IPMI_SIM**

    * Enter IPMI_SIM by below command.
       If you are on the top of the application server, use::

              ssh vbmc_ip -p 9300


       If you are on the top of vbmc server, use::

              ssh localhost -p 9300


    * Enter help to check all the commands supported.::

           # help

   *  Below tables shows the detail information about each command.

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

   * You can also get the BMC data by IPMI command. For example, have a check on fan speed and check the sel list by: ::

       # ipmitool -I lanplus -U admin -P admin -H <vm ip address> sdr type fan
       # ipmitool -I lanplus -U admin -P admin -H <vm ip address> sel list




vPDU deployment and control
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can implement the vSwitch component of InfraSIM by deploying the Cisco Nexus 1000v switch on the ESXi host.

For more information on downloading and using Cisco Nexus 1000v switch, refer to http://www.cisco.com/c/en/us/products/switches/nexus-1000v-switch-vmware-vsphere/index.html.
