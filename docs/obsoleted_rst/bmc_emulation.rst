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
