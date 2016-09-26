Installation
=========================


Requirement
------------------------------------------------

.. note:: Configure the BIOS to enable virtualization.

    .. image:: _static/configBIOSpng.png
        :height: 400
        :align: center

.. note:: Login to the ESXi server through SSH and echo by issuing the **"vhv.enable = "TRUE""** command to the /etc/vmware/config file. This command enables nested ESXi and other hypervisors in vSphere 5.1 or higher version. This step only needs to be done once by using the command: echo 'vhv.enable = "TRUE"' >> /etc/vmware/config.


.. note:: Set **Promiscuous Mode** to Accept and tick Override. To do this, open the Configuration tab and select Networking. Then click Properties of the vSwitch, choose port group, edit, security, tick the checkbox to override setting and select Accept.    

**Bryan - Under construction**


Virtual Server
------------------------------------------------

**Mark - Under construction**


Virtual Power Distribution Unit
------------------------------------------------

 Current Virtual PDU implementation only supports running entire virutal infrastructure on VMWare ESXi because it only supports functionality of simulating power control chassis through VMWare SDK.
**Robert - Under construction**




