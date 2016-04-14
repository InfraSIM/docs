How Tos
===============================

VMWare Workstation deployment
---------------------------------------------
The simulated hardware elements(vNode, vPDU, vSwitch) by InfraSIM can be deployed on VMWare workstation. Please follow below steps to setup InfraSIM in VMWare workstation.

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


.. include:: how_to_build.rst


Write test case
--------------------------------------

This section introduces how to write test case in puffer.

#. Create a test script file

   * **Test Case Name**

     The name of test case should follow the same format::

        T\d+_[a-z0-9A-Z]+_[a-z0-9A-Z]+

     In puffer, test case name should:
      - Start with capital letter **T** and case id
      - Followed by the **field type** and **short description** about this case with underscores in the interval. Field types defined in class CBaseCase.

        **Note:** The field type for InfraSIM is **idic**.

     For example, a test case named **T123456_idic_CheckPowerStatus**:
      - **T** is short for test
      - **123456** for case id
      - **idic** for field type
      - **check the power status** for the short description


   * **Test Suite**

     You should put your test case scripts into **<puffer_directory>/case/<test_suite>**. Each folder under **<puffer_directory>/case** is a test suite. When you give the suite folder to puffer.py as a parameter, puffer will executes all test case scripts which in the folder, including subfolders.


#. Create case runtime data file

   Case Runtime Data is used to maintain some specific data for different test objects. These data generally require the user to add and update manually.

   The format of case runtime data defined in the json file which have same name and folder with case script. Please see the chapter `Case Runtime Data </userguide.html#case-runtime-data>`_ .

#. Write test case

   A. Import CBaseCase

      Class CBaseCase defined in **<puffer_directory>/case/CBaseCase.py**, contains some member functions to help test case running::

          from case.CBaseCase import *

   B. Class Declaration

      We declaration each case as subclass of class CBaseCase and the class name is case name. For example, if case name is T123456_idic_CheckPowerStatus, the class name should be same to it.

      A test case maybe looks like::

          from case.CBaseCase import *

          class T000000_firmware_shortdescription(CBaseCase):

              def __init__(self):
                  CBaseCase.__init__(self, self.__class__.__name__)

              def config(self):
                  CBaseCase.config(self)

              def test(self):
                  pass

              def deconfig(self):
                  CBaseCase.deconfig(self)

      And then, we need to override methods of class CBaseCase, such as config(), test() and deconfig().

   C. Override config()

      This method configuration system to expected status, configuration runtime HWIMO environment and stack environment.

      The HWIMO configuration will set logger to save session log into log file and configuration SSH agent and stack configuration will build stack object, configuration stack ABS according to dict, build all nodes and power on.

      However, in some case we want to enable some components we need to enable manually in configuration(). For example, if we want to use the ssh inside vbmc, we need enable the bmc_ssh in configuration()::

          def config(self):
              CBaseCase.config(self)
              self.enable_bmc_ssh()

   D. Override test()

      This method is the main part of the test.

      You can:

      - Use self.stack to get the stack which build in config().

      - Use self.data[] to get case runtime data.

      - Use self.monorail to use Monorail API.

      - Use self.log() to log the information.

      - Use self.result() to save the case result.

      For example::

          def test(self):
              #get racks from stack and get nodes from rack
              for obj_rack in self.stack.get_rack_list():
                  for obj_node in obj_rack.get_node_list():

                      #log the information
                      self.log('INFO', 'Check node {} of rack {} ...'
                          .format(obj_node.get_name(), obj_rack.get_name()))

                      #get and match outlet power
                      for power_unit in obj_node.power:
                          pdu_pwd = power_unit[0].get_outlet_password(power_unit[1])
                          power_unit[0].match_outlet_password(power_unit[1], pdu_pwd)

                      #virtual node power control
                      obj_node.power_on()

                      #use case runtime data
                      node_name = obj_node.get_name()
                      node_lan_channel = self.data[node_name]

                      #send command to virtual bmc through ssh
                      obj_bmc = obj_node.get_bmc()
                      bmc_ssh = obj_bmc.ssh
                      ssh_rsp = bmc_ssh.send_command_wait_string(
                          str_command = 'ipmitool -I lanplus -H localhost -U {} -P {} lan print {} {}'.format(obj_bmc.get_username(), obj_bmc.get_password(), node_lan_channel, chr(13)),
                          wait = '$',
                          int_time_out = 3,
                          b_with_buff = False)

                      #send command to virtual bmc through ipmitool
                      ret, ipmi_rsp = obj_node.get_bmc().ipmi.ipmitool_standard_cmd('lan print')

                      #if case failed
                      if ret != 0:
                          self.result(FAIL, 'FAIL_INFORMATION')
                      else:
                      #if no issue in this run, case pass.
                          self.log('INFO', 'PASSED.')

   E. Override deconfig()

      This method deconfig system to expected status, reset REST and SSH sessions, deconfig stack and log handler::

          def deconfig(self):
              self.log('INFO', 'Deconfig')
              CBaseCase.deconfig(self)
