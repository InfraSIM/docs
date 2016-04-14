InfraSIM Overview
===================


InfraSIM provides the technology to simulate the interface and behavior of hardware devices including compute, storage, networking, and smart PDUs(Power Distribute Units).
It leverages the technology of virtualization which enables to simulate a big amount of hardware devices with limited physical resources. And these simulated hardware devices can be configured to construct an @scale infrastructure.

InfraSIM Overview
------------------------------------

InfraSIM offers the interface to set up, configure an @scale infrastructure which provides a total solution for CI application development and test. The diagram below illustrates the development concept of the @Scale deployment.

.. image:: _static/atscaledeploy.png
     :align: center

.. image:: _static/theme/img/space.png
    :height: 40
    :align: center


InfraSIM Components
------------------------------------


Below tables demonstrate the simulated hardware elements by InfraSIM.

.. list-table::
   :widths: 20 100
   :header-rows: 1

   * - Terminology
     - Description
   * - InfraSIM
     - Use the combination of hardware virtualization and emulation technologies to simulate the interfaces and behaviors of hardware elements in the test domain.
          The simulated hardware elements are called by the 'vXXX' term, with the prefix "v" for virtual.
   * - vCompute
     - Virtual Compute Node
          The simulation of a physical compute node which includes the core compute subsystem and the standby BMC that control and monitor hardware resources of the compute node.
   * - vHost
     - Virtual Host (CPU subsystem)
          The simulation of the core compute subsystem of a compute node.
          vHost is the core hardware resources of the compute node that host OS and product applications.
   * - vBMC
     - virtual BMC. It contains two concepts depending on the reference context:
          1. The simulated BMC controller of a compute node. 
          2. A wrapping VM image containing virtual BMC and the whole compute node implementation.
   * - vSwitch
     - Virtual Switch
          The virtualized control, data, or admin switch.
   * - vPDU
     - Virtual Smart PDU
          The simulation of the smart PDU.

InfraSIM Use Cases
------------------------------------

Currently, InfraSIM has successfully proved that it is capable of not only saving lots of cost of purchasing hardware material for setting up a pure bare-metal environment, but also providing many flexibilities in software developing and testing areas. Here's one example of how infraSIM is leveraged as part of test solution for RackHD™:

**Notes:**
RackHD™ is an open source project that provides hardware orchestration and management through RESTful APIs. For more information about RackHD, go to http://rackhd.readthedocs.org/en/latest/.

#. At scale test
    We can validate RackHD functionalities by having it manage and orchestrate a virtual infrastructure with adjustable or big scalability. Then we can evaluate RackHD performance benchmark and ensure its functionatlies in an environment with:
       * Big number of nodes
       * Diversity of node type - different type, model, vendors, etc
       * Increased complexity of network topology

#. Telemetry data testing
    InfraSIM allows generating and modifying server sensor readings that we can better test feature of telemetry data of RackHD.

#. Node provision
    InfraSIM allows customizing node device tree and manipulating FE behavior, we can better test node provision feature, for example, bootstrapping servers and deploying operating systems, hypervisors and applications.

#. Error injection
    Because infraSIM is adopting software approach to simulate hardware, both elements and entire infrastructure, it provided more feasibility and easiness to simulate hardware failures to test our software error handing logic.

#. More...


