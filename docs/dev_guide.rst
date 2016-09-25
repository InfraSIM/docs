Development Guide
=========================



Repositories
------------------------------------------------

The InfraSIM repositories provide you with the code to set up, configure, and test a virtual environment consisting of simulated servers, storage devices, and smart PDUs. A thorough understanding of the individual repositories is essential for contributing to the project.

.. list-table::
   :widths: 25 35 40
   :header-rows: 1

   * - Application
     - Repository
     - Description
   * - IDIC
     - https://github.com/InfraSIM/idic
     - Idic repository includes vBMC, vCompute, and vPDU. vBMC is the base OS of virtual BMC. vCompute simulates the common functionalities of a compute node and the behaviors of a generic server and several servers from vendors like Dell, Quanta, etc.
   * - vpduserv
     - https://github.com/InfraSIM/vpduserv
     - Simulates the behaviors of the IPI PANDUIT PDU which conforms with vendor and open source specified licenses.
   * - QEMU
     - https://github.com/InfraSIM/qemu
     - QEMU is a generic and open source machine emulator and virtualizer, more information please access http://wiki.qemu-project.org/.
   * - OpenIPMI
     - https://github.com/InfraSIM/openipmi
     - OpenIPMI library, a library that makes it simple to build complex IPMI management software.
   * - Test
     - https://github.com/InfraSIM/test
     - Scripts for InfraSIM automation and integration tests. It includes the test framework(puffer) and many test cases against the features InfraSIM provided.
   * - Tools
     - https://github.com/InfraSIM/tools
     - Various tools and scripts to monitor and manage generic and common virtual nodes, virtual rack build.
   * - vRacksystem
     - https://github.com/InfraSIM/vracksystem
     - The vRacksystem provides both REST APIs and WebGUI for deploying and configuring vNode/vPDU to compose virtual racks.
   * - docs
     - https://github.com/InfraSIM/docs
     - The InfraSIM documentation available at http://InfraSIM.readthedocs.org/en/latest/.


Development conventions
------------------------------------------------



3rd-party binaries notes
------------------------------------------------



Component design notes
------------------------------------------------



Logging and debugging
------------------------------------------------



Unit test
------------------------------------------------



Functional test
------------------------------------------------



