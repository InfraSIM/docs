Build, Package and Deployment
=================================================

This chapter will introduce how to build, deploy and run virtual compute node on top of KVM, Docker and VirtualBox.

In the following `User Guide <userguide.html>`_ chapter, it will describe how to deploy and run both virtual compute node, virtual PDU and virtual switch on top of VMWare ESXi. There’s also one `vRackSystem <userguide.html#vracksystem>`_ which supports deploying large scale infrastructure crossing multiple virtual racks.

Build Generic Virtual Compute Node
---------------------------------------------

#. Install Prerequisite packages::

    # sudo apt-get install mkisofs autoconf pkg-config libtool nsis bison flex libncurses5 libncurses5-dev zlib1g-dev libglib2.0-dev libpopt-dev libssl-dev python-dev git libdumbnet1 libdumbnet-dev tclsh


#. Git clone `InfraSIM <https://github.com/InfraSIM/InfraSIM.git>`_ repository::

    # git clone <idic repo url>

#. Configure the Virtual Compute Node::

    # cd idic
    # fakeroot make menuconfig NODE=vnode

   You can make some changes for default configuration. Refer to `How to build vNode and vPDU <how_tos.html#build-vnode-and-vpdu>`_. After you finish the changes, you can save and exit.

#. Build the Virtual Compute Node Root Filesystem::

    # cd idic
    # fakeroot make vnode

   If success, you will see **config, System.map, vmlinuz and ramfs.lzma** under the directory "idic/pdk/linux/vnode"


Deploy and Run Virtual VMs
----------------------------

Currently, the hypervisors that InfraSIM supported are:
    -  `Virtualbox <https://www.virtualbox.org/>`_
    -  `KVM <http://www.linux-kvm.org>`_
    -  `Docker <https://www.docker.com>`_
    -  `VMWare product <https://www.vmware.com>`_

Here's introduction on how to deploy InfraSIM build on different hypervisors.

**Prerequisite**:

#. Clone `tool <https://github.com/InfraSIM/InfraSIM.git>`_ repository
#. Build the vnode image following the steps in `Build Generic Virtual Compute Node <builddeploy.html#build-generic-virtual-compute-node>`_ Section.

Run Docker-based VM
~~~~~~~~~~~~~~~~~~~

#. Install necessary packages.::

    #sudo apt-get install docker docker-engine

#. Build Docker-based Image.::

    # cd tools/docker_builder
    # sudo ./docker_image_builder.sh -d <your idic directory>/pdk/linux/vnode -t vnode

#. Start Docker-based VM.::

    # ./run_docker.sh -n vnode -t vnode


Run KVM-Based Virtual Machine
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Install necessary packages.::

     # sudo apt-get install virt-manager

#. Build KVM-based Image.::

	# cd tools/kvm_builder
	# sudo ./kvm_builder.sh -d <your idic directory>/pdk/linux/vnode -t vnode


   If success, you will see "vnode.qcow2" disk image under the directory.

#. Start KVM-based VM using qemu.::

     # ./start_kvm_vm.sh -n vnode

#. Use virt-manager to see the GUI interface
   You can see the running virtual machine in virt-manager

   .. image:: _static/virt-manager.png

Run Virtualbox-based VM
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We use `vagrant <https://www.vagrantup.com>`_ to create the Virtualbox-based VM.
Using vagrant, you can quickly create the Virtualbox-based VM on your laptop, work PC whatever your system is MacOS, Linux or even Windows.

**Notice:** We will assume that you have basic knowledge about how vagrant works and vagrant version should be > 1.8

#. Install necessary packages::

    #sudo apt-get install virtualbox

#. Build Virtualbox-based Image::

    # cd tools/virtualbox_builder
    # sudo ./virtualbox_builder.sh -d <your idic directory>/pdk/linux/vnode -n vnode

   You will see that the "vnode.box" is under this directory

#. Start Virtualbox-based VM::

    # vagrant box add --name vnode vnode.box
    # mkdir vnode && cd vnode
    # vagrant init vnode
    # vagrant up

   **Notice:** Because VirtualBox itself doesn't and has no plan to support `Nested Virtualization: VT-in-VT <https://www.virtualbox.org/ticket/4032>`_ feature, running Qemu inside VirtualBox can't enable KVM acceleration, which suffers a big performance penalty.

Run VMWare product based VM
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#. Refer to `How to build vNode and vPDU <how_tos.html#build-vnode-and-vpdu>`_ for building OVA image working for both ESXi and VMWare workstation.
#. Refer to `VMWare Workstation deployment <how_tos.html#vmware-workstation-deployment>`_ for deploying virtual node on VMWare workstation.
#. Refer to :ref:`setup-infrasim-on-esxi-label` for deploying virtual node on VMWare ESXi.
