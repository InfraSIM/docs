.. _Installation:

Installation
=========================

For virtual node simulating server or PDU, either bare-metal machine (server, laptop, desktop) or virtual machine can host one. It requires configuring network (either corresponding physical one or virtual one, even mixing physical and virtual network together for hybrid configuration) in order to compose one infrastructure containing virtual servers, PDUs and specified network topology.

Here's requirement on hardware environment and virtualization environment running InfraSIM:

Requirement
------------------------------------------------

Pre-requisite
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Several mandatory configuration has to be made as below which is required to accommodate InfraSIM virtualization-nesting design. `How to install VMWare ESXi <how_to.html#how-to-install-vmware-esxi-on-physical-server>`_ describes a example of how to achieve them when installing and configuring VMWare ESXi.

#. Virtual InfraSIM servers runs in the best performance if hardware-assisting technology has been enabled on underlying physical machines. These technology includes VT-d feature and AMD-V for processors from Intel and AMD.

    .. note:: **Physical machine** - enable VT-d in BIOS

#. When virtual server is running inside VM, it also requires underlying hypervisor passing down the hardware-virtualization-assisting to virtual machine it spawn.

    .. note:: **VMWare ESXi hypervisor** - Set **"vhv.enable = "TRUE"**

    .. caution:: **InfraSIM running on VirtualBox** will have performance penalty when running specific work load (deploying operating system, running compute-intensive application inside virtual server). This is because VirtualBox doesn't support simulating a platform which is capable of supporting hardware-virtualization-assisting feature.

#. Ensure Promiscuous Mode of virtual switch, virtual network controller has been enabled for underlying hypervisors hosting virtual machines running InfraSIM inside. Here's example on how to achieve it on top VMWare ESXi

    .. note:: **Promiscuous Mode** - `How to install VMWare ESXi <how_to.html#how-to-install-vmware-esxi-on-physical-server>`_


Resource Requirement
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* 1 physical CPU or 1 virtual CPU
* 4GB memory
* 16GB disk space
* 1 virtual or physical NIC


Software environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ubuntu Linux 64-bit - 16.04 is recommended


Virtual Server
------------------------------------------------

As Python Application
~~~~~~~~~~~~~~~~~~~~~~~

#. Ensure sources.list integrity then install dependency::

    sudo apt-get update
    sudo apt-get install python-pip libpython-dev libssl-dev

#. Upgrade pip and install setuptools::

    sudo pip install --upgrade pip
    sudo pip install setuptools

#. Select either one of below ways to install infrasim:

    * install infrasim from source code::

        git clone https://github.com/InfraSIM/infrasim-compute.git
        cd infrasim-compute
        sudo pip install -r requirements.txt

        sudo python setup.py install

    * install infrasim from python library::

        sudo pip install infrasim-compute

As Docker Image
~~~~~~~~~~~~~~~~~~~

We also provide docker support in `InfraSIM tools<https://github.com/InfraSIM/tools/tree/master/docker>`_.

You can get:

* ``Dockerfile`` to build your InfraSIM docker image
* ``docker.py`` to setup a self-defined InfraSIM cluster in your environment, it has several docker runtimes hosting
InfraSIM virtual node respectively, with openvswitch connection powered by pipework

.. hide_content::


        Virtual Power Distribution Unit - **Robert - Under construction**
        ------------------------------------------------

        Current Virtual PDU implementation only supports running entire virtual infrastructure on VMWare ESXi because it only supports functionality of simulating power control chassis through VMWare SDK.
