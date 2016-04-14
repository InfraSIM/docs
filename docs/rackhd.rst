Integrating RackHD with InfraSIM
--------------------------------------------------

RackHD is an open source project that provides hardware orchestration and management through APIs. For more information about RackHD, go to http://rackhd.readthedocs.org/en/latest/.

The virtual hardware elements(virtual compute node, virtual PDU, virtual Switch) simulated by InfraSIM can be managed by RackHD.

The following picture shows the deployment model for the integration of InfraSIM and RackHD:

.. image:: _static/connections_RackHD.png
           :height: 600
           :align: center

Please follow up below steps to step the entire environment. After that, RackHD can discover and manage the virtual server and virtual PDU just as the real physical server and PDU. 

#. Please refer RackHD document(http://rackhd.readthedocs.org/en/latest/getting_started.html) to setup the RackHD Server. RackHD server should be configured with as least two networks, "Admin network" and "Control Network".
    * "Admin Network" is used to communicate with external servers
    * "Control Network" is used to control the virtual servers.

#. Deploy virtual compute node and virtual PDU on ESXi with the network connection as in the pictures show. The ESXi show have network connection with the RackHD server.

After you setup the environment successfully, you can get the server information and control the servers by RackHD APIs. More information about how RackHD APIs communicate with the compute server and PDU, Please refer http://rackhd.readthedocs.org/en/latest/rackhd/index.html#rackhd-api
