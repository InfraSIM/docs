vNode Deployment and Control
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Deploy virtual Compute Node to ESXi manually
    The virtual compute node OVA image is a **standard** image that can be deployed to ESXi directly.

    **IMPORTANT NOTE:** Login to the ESXi server through SSH and echo by issuing the **"vhv.enable = "TRUE""** command to the /etc/vmware/config file. This command enables nested ESXi and other hypervisors in vSphere 5.1 or higher version. This step only needs to be done once by using the command: echo 'vhv.enable = "TRUE"' >> /etc/vmware/config.

#. Deploy virtual compute node by vRackSystem
    Please access `vRackSystem User Manual <userguide.html#vracksystem>`_ for more information.