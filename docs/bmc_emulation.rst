Access vBMC Data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This section explains how to access the vBMC information and how to change the boot option of the virtual node. You can access information from the virtual BMC, change sensor values,
configure the sensor mode, add a SEL entry for a particular sensor, and monitor the virtual node BIOS/iPXE/OS/ runtime output instead of VNC.


#. Accessing BMC Static Data
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

#. Accessing Dynamic Sensor Data
    BMC emulation has two parts: the IPMI simulator and the IPMI simulator wrapper. Both parts start running after the vBMC boots up.

    To communicate with the IPMI simulator, use the ipmitool command. To communicate with the IPMI simulator wrapper, use SSH. You can use manual or automated methods to the communicate with the IPMI simulator wrapper.

    If you are on the top of the application server, use::

            ssh vbmc_ip -p 9300

    If you are on the top of vbmc server, use::

            ssh localhost -p 9300

    You can also write your SSH client code to talk with it in automation way.
