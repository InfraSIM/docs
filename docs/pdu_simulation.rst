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
      Please access `vRackSystem User Manual <userguide.html#vracksystem-user-manual>`_ for more information.

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


#. Verify the vPDU Service

   You can use SNMP commands to verify the vPDU service.
   * Get the PDU device name::

        snmpwalk -On -v2c -c ipia <vPDU IP Address> SNMPv2-MIB::sysName

   * Retrieve information about the PDU device::

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
