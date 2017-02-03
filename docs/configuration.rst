Configuration
=========================



Virtual Server Configuration file
------------------------------------------------

There's one central virtual server configuration file which is **~/.infrasim/.node_map/default.yml** (`source code <https://github.com/InfraSIM/infrasim-compute/blob/master/template/infrasim.yml>`_). All adjustable parameters are defined in this file. This is the only file to modify if you want to customize or make adjustment on the virtual server node. While not all supported options are explicitly listed in this file for purpose of simplicity. However there's one example configuration file - **/etc/infrasim.full.yml.example** (`source code <https://github.com/InfraSIM/infrasim-compute/blob/master/etc/infrasim.full.yml.example>`_) - listed all supported parameters and definitions. By referring content in example file, you can `update node configuration <https://github.com/InfraSIM/infrasim-compute/wiki/Manage-node-config>`_ and then restart ``infrasim node`` service and then new properties will take effect.

If you want to manage your own cofiguration and start infrasim instance accordingly, refer to this article: `Manage Node Config <https://github.com/InfraSIM/infrasim-compute/wiki/Manage-node-config>`_

Here's full list of the example configuration file; every single key-value pair is supported to be add/modify in your real-in-use infrasim.yml::

    # This example virtual server configuration file intends to throughout
    # list parameters and properties that infrasim-compute virtual server
    # supports to adjust. In most cases it is fine to use default value
    # for particuar configuration by skipping putting it into infrasim.yml
    # configuration file. For anything item you're interested, it is recommended
    # to look up infomation here first. For example, if you'd like to customize
    # properties of your drive - either serial number or vender - in below there're
    # corresponding item to show how to achieve that.

    # Unique identifier
    name: node-0

    # Node type is mandatory
    # Node type of your infrasim compute, this will determine the
    # bmc emulation data and bios binary to use.
    # Supported compute node names:
    #  quanta_d51
    #  quanta_t41
    #  dell_c6320
    #  dell_r630
    #  dell_r730
    #  dell_r730xd
    #  s2600kp - Rinjin KP
    #  s2600tp - Rinjin TP
    #  s2600wtt - Node of Hydra, Python
    type: quanta_d51

    compute:
        # n - Network (PXE); c - CD-ROM;
        # d - Drive (bootindex in drive sections controls order of booting HDD)
        boot_order: ncd
        kvm_enabled: true
        numa_control: true
        cmdline: API_CB=192.168.1.1:8000 BASEFS=base.trusty.3.16.0-25-generic.squashfs.img OVERLAYFS=discovery.overlay.cpio.gz BOOTIF=52-54-BF-11-22-33
        cpu:
            model: host
            features: +vmx
            quantities: 8
        memory:
            size: 4096
        # Currently the PCI bridge is only designed for megasas storage controller
        # When you create multiple megasas controller, the controllers will be assigned
        # a different pci bus number
        pci_bridge_topology:
            -
                device: i82801b11-bridge
                addr: 0x1e.0x0
                multifunction: on
                    -
                        device: pci-bridge
                        chassis_nr: 0x1
                        msi: false
                        addr: 0x1
        storage_backend:
            -
                controller:
                    type: ahci
                    max_drive_per_controller: 6
                    drives:
                    -
                        model: SATADOM
                        serial: HUSMM142
                        bootindex: 1
                        # To boot esxi, please set ignore_msrs to Y
                        # sudo -i
                        # echo 1 > /sys/module/kvm/parameters/ignore_msrs
                        # cat /sys/module/kvm/parameters/ignore_msrs
                        file: chassis/node1/esxi6u2-1.qcow2
                    -
                        vendor: Hitachi
                        model: HUSMM0SSD
                        serial: 0SV3XMUA
                        # To set rotation to 1 (SSD), need some customization
                        # on qemu
                        # rotation: 1
                        # Use RAM-disk to accelerate IO
                        file: /dev/ram0
                    -
                        vendor: Samsung
                        model: SM162521
                        serial: S0351X2B
                        # Create your disk image first
                        # e.g. qemu-img create -f qcow2 sda.img 2G
                        file: chassis/node1/sda.img
                    -
                        vendor: Samsung
                        model: SM162521
                        serial: S0351X3B
                        file: chassis/node1/sdb.img
                    -
                        vendor: Samsung
                        model: SM162521
                        serial: S0451X2B
                        file: chassis/node1/sdc.img
            -
                controller:
                    type: megasas-gen2
                    use_jbod: true
                    use_msi: true
                    max_cmds: 1024
                    max-sge: 128
                    max_drive_per_controller: 1
                    drives:
                        -
                            vendor: HITACHI
                            product: HUSMM168XXXXX
                            serial: SN0500010351XXX
                            rotation: 1
                            slot_number: 0
                            wwn: 0x50000ccaxxxxxxxx
                            file: <path/to/your disk file>

        networks:
            -
                network_mode: bridge
                # Bridge need to be prepared beforehand with brctl
                network_name: br0
                device: vmxnet3
                mac: 00:60:16:9e:a8:e9
            -
                network_mode: nat
                device: e1000
        ipmi:
            interface: bt
            chardev:
                backend: socket
                host: 127.0.0.1
                reconnect: 10
            ioport: 0xca8
            irq: 10
        smbios: chassis/node1/quanta_d51_smbios.bin
        monitor:
            mode: readline
            chardev:
                backend: socket
                server: true
                wait: false
                host: 127.0.0.1
                port: 2345
        # set vnc display <X>
        vnc_display: 1
    bmc:
        interface: br0
        username: admin
        password: admin
        address: <ip address>
        channel: 1
        lancontrol: <path/to/lan control script>
        chassiscontrol: <path/to/chassis control script>
        startcmd: <cmd to be excuted>
        startnow: true
        poweroff_wait: 5
        kill_wait: 5
        historyfru: 20
        config_file: <path/to/your config file>
        emu_file: chassis/node1/quanta_d51.emu
        ipmi_over_lan_port: 623

    # SSH to this port to visit ipmi-console
    ipmi_console_ssh: 9300

    # Renamed from telnet_listen_port to ipmi_console_port, extracted from bmc
    # ipmi-console talk with vBMC via this port
    ipmi_console_port: 9000

    # Used by ipmi_sim and qemu
    bmc_connection_port: 9100

    # Socket file to bridge socat and qemu
    serial_socket: /tmp/serial

Up to infrasim-compute commit `a02417c3 <https://github.com/InfraSIM/infrasim-compute/commit/a02417c37f6b6fb266244e77e992f66938c73f8d>`_

.. _yamlName:

- **name**

    This attribute defines nodes name, which is a unique identifier for infrasim-compute instances on the same platform.
    More specifically, it is used as `workspace <https://github.com/InfraSIM/infrasim-compute/wiki/Compute-Node-Workspace>`_ folder name.

    **NOT Mandatory**

    **Default**: "node-0"

    **Legal Value**: String

.. _yamlType:

- **type**

    This attribute defines supported nodes type in InfraSIM. With this attribute, infrasim-compute will set BMC emulation data for ``ipmi_sim`` and BIOS binary for ``qemu`` accordingly, you can get corresponding .emu and .bin in ``/usr/local/etc/infrasim/`` by default.

    **Mandatory**

    **Legal Values**:

        - "quanta_d51"
        - "quanta_t41"
        - "dell_c6320"
        - "dell_r630"
        - "dell_r730"
        - "dell_r730xd"
        - "s2600kp", for Rinjin KP
        - "s2600tp", for Rinjin TP
        - "s2600wtt", for Hydra, Python

.. _yamlCompute:

- **compute**

    This block defines all attributes used by `QEMU <http://wiki.qemu.org/Main_Page>`_.
    They will finally be translated to one or more ``qemu`` command options.
    The module ``infrasim.model.CCompute`` is handling this translation.
    This is much like a definition for `libvert <https://libvirt.org/>`_, but we may want it to be lite, and compatible with some customized qemu feature in InfraSIM.

.. _yamlComputeBootorder:

- **compute:boot_order**

    This attribute defines boot order for ``qemu``. Will be translated to ``-boot {boot_order}``.

    **Not Mandatory**

    **Default**: "ncd", means in a order of pxe > cdrom > default.

    **Legal Value**: See ``-boot`` in `qemu-doc <http://wiki.qemu.org/download/qemu-doc.html>`_.

.. _yamlComputeKvmenabled:

- **compute:kvm_enabled**

    This attribute enable `kvm <http://wiki.qemu.org/Features/KVM>`_ when you announce it as True and your system supports kvm. It will be translated to ``--enable-kvm``. You can check if your system supports kvm by check if ``/dev/kvm`` exists.

    **Not Mandatory**

    **Default**: Depends on if ``/dev/kvm`` exists.

    **Boolean Table**

    +------------+-------------+--------------+
    |kvm_enabled |/dev/kvm     |--enable-kvm  |
    +============+=============+==============+
    |true        |yes          |yes           |
    +------------+-------------+--------------+
    |true        |no           |no            |
    +------------+-------------+--------------+
    |false       |yes          |no            |
    +------------+-------------+--------------+
    |false       |no           |no            |
    +------------+-------------+--------------+
    |not define  |yes          |yes           |
    +------------+-------------+--------------+
    |not define  |no           |no            |
    +------------+-------------+--------------+

.. _yamlComputeNumacontrol:

- **compute:numa_control**

    This attribute enable `NUMA <https://en.wikipedia.org/wiki/Non-uniform_memory_access>`_ to improve InfraSIM performance by binding to certain physical cpu.
    If you have installed ``numactl`` and set this attribute to True, you will run qemu in a way like ``numactl --physcpubind={cpu_list} --localalloc``.

    **Not Mandatory**

    **Default**: Disabled

.. _yamlComputeCmdline:

- **compute:cmdline**

    This attribute will be appended to qemu in string as part of the option ``--append {cmdline}``.
    See ``--append`` in `qemu-doc <http://wiki.qemu.org/download/qemu-doc.html>`_.
    It will be then used by qemu as kernel parameters.
    You can view your O/S's kernel parameters by ``cat /proc/cmdline``.

    **Not Mandatory**

    **Default**: None, there will be no ``--append`` option.

.. _yamlComputeCpu:

- **compute:cpu**

    This group of attributes set qemu cpu characteristics. The module ``infrasim.model.CCPU`` is handling the information.

.. _yamlComputeCpuModel:

- **compute:cpu:model**

    This attribute sets qemu cpu model.

    **Not Mandatory**

    **Default**: "host"

    **Legal Values**: See ``-cpu model`` in `qemu-doc <http://wiki.qemu.org/download/qemu-doc.html>`_.

.. _yamlComputeCpuFeatures:

- **compute:cpu:features**

    This attribute adds or removes cpu flags according to your customization. It will be translated to ``-cpu Haswell,+vmx`` for example.

    **Not Mandatory**

    **Default**: "+vmx"

    **Legal Values**: See ``-cpu model`` in `qemu-doc <http://wiki.qemu.org/download/qemu-doc.html>`_.

.. _yamlComputeCpuQuantities:

- **compute:cpu:quantities**

    This attribute sets virtual cpu numbers in all. With default socket 2, CCPU calculates core per socket. Default set to 1 thread per cores.
    It will be translated to ``-smp {cpus},sockets={sockets},cores={cores},threads=1`` for example.

    **Not Mandatory**

    **Default**: 2

    **Legal Values**: See ``-smp`` in `qemu-doc <http://wiki.qemu.org/download/qemu-doc.html>`_.

.. _yamlComputeMemory:

- **compute:memory**

    This attribute refers to RAM, which the virtual computer devices use to store information for immediate use.
    The module ``infrasim.model.CMemory`` is handling the information.

.. _yamlComputeMemorySize:

- **compute:memory:size**

    This attribute sets the startup RAM size. The default is 1024MB.

    **Default**: 1024

    **Legal Values**: See ``-m`` in `qemu-doc <http://wiki.qemu.org/download/qemu-doc.html>`_.

.. _yamlComputeStoragebackend:

- **compute:storage_backend**

.. _yamlComputeStoragebackendController:

- **compute:storage_backend:-:controller**

.. _yamlComputeStoragebackendControllerType:

- **compute:storage_backend:-:controller:type**

.. _yamlComputeStoragebackendControllerMaxdrivepercontroller:

- **compute:storage_backend:-:controller:max_drive_per_controller**

.. _yamlComputeStoragebackendControllerUsejbod:

- **compute:storage_backend:-:controller:use_jbod**

.. _yamlComputeStoragebackendControllerUsemsi:

- **compute:storage_backend:-:controller:use_msi**

.. _yamlComputeStoragebackendControllerMaxcmds:

- **compute:storage_backend:-:controller:max_cmds**

.. _yamlComputeStoragebackendControllerMaxsge:

- **compute:storage_backend:-:controller:max-sge**

.. _yamlComputeStoragebackendControllerDrives:

- **compute:storage_backend:-:controller:drives**

.. _yamlComputeStoragebackendControllerDrivesModel:

- **compute:storage_backend:-:controller:drives:-:model**

.. _yamlComputeStoragebackendControllerDrivesSerial:

- **compute:storage_backend:-:controller:drives:-:serial**

.. _yamlComputeStoragebackendControllerDrivesBootindex:

- **compute:storage_backend:-:controller:drives:-:bootindex**

.. _yamlComputeStoragebackendControllerDrivesFile:

- **compute:storage_backend:-:controller:drives:-:file**

.. _yamlComputeStoragebackendControllerDrivesVendor:

- **compute:storage_backend:-:controller:drives:-:vendor**

.. _yamlComputeStoragebackendControllerDrivesRotation:

- **compute:storage_backend:-:controller:drives:-:rotation**

.. _yamlComputeNetworks:

- **compute:networks**

.. _yamlComputeNetworksNetworkmode:

- **compute:networks:-:network_mode**

.. _yamlComputeNetworksNetworkname:

- **compute:networks:-:network_name**

.. _yamlComputeNetworksDevice:

- **compute:networks:-:device**

.. _yamlComputeNetworksMac:

- **compute:networks:-:mac**

.. _yamlComputeIpmi:

- **compute:ipmi**

.. _yamlComputeIpmiInterface:

- **compute:ipmi:interface**

.. _yamlComputeIpmiChardev:

- **compute:ipmi:chardev**

.. _yamlComputeIpmiChardevBackend:

- **compute:ipmi:chardev:backend**

.. _yamlComputeIpmiChardevHost:

- **compute:ipmi:chardev:host**

.. _yamlComputeIpmiChardevReconnect:

- **compute:ipmi:chardev:reconnect**

.. _yamlComputeIpmiIoport:

- **compute:ipmi:ioport**

.. _yamlComputeIpmiIrq:

- **compute:ipmi:Irq**

.. _yamlComputeSmbios:

- **compute:smbios**

.. _yamlComputeMonitor:

- **compute:monitor**

.. _yamlComputeMonitorMode:

- **compute:monitor:mode**

.. _yamlComputeMonitorChardev:

- **compute:monitor:chardev**

.. _yamlComputeMonitorChardevBackend:

- **compute:monitor:chardev:backend**

.. _yamlComputeMonitorChardevServer:

- **compute:monitor:chardev:server**

.. _yamlComputeMonitorChardevWait:

- **compute:monitor:chardev:wait**

.. _yamlComputeMonitorChardevPath:

- **compute:monitor:chardev:path**

.. _yamlComputeVncdisplay:

- **compute:vnc_display**

.. _yamlBmc:

- **bmc**

    This block defines attributes used by `OpenIPMI <http://openipmi.sourceforge.net/>`_.
    They will finally be translated to one or more ``ipmi_sim`` command options, or be defined in the configuration file for it.
    The module ``infrasim.model.CBMC`` is handling this translation.

.. _yamlBmcInterface:

- **bmc:interface**

   This attributes defines both:

   - from which network ``ipmi_sim`` will listen IPMI request

   - BMC's network properties printed by ``ipmitool lan print``

   The module ``infrasim.model.CBMC`` takes this attribute and comes out with two variable defined in ipmi_sim `configuration template <https://github.com/InfraSIM/infrasim-compute/blob/master/template/vbmc.conf>`_.

   - ``{{lan_interface}}``, network name for ``ipmitool lan print`` to print, e.g. "eth0", "ens190".

   - ``{{ipmi_listen_range}}``, IP address that ipmi_sim shall listen to and response IPMI command. If you set a valid interface here, an IP address in string will be assigned to this variable, e.g. "192.168.1.1".

   **Not Mandatory**


   **Default**

   - ``{{lan_interface}}``: first network device except ``lo``.

   - ``{{ipmi_listen_range}}``: "::", so that you shall see ``addr :: 623`` in vbmc.conf, it means ipmi_sim listen to IPMI request on all network on port 623


   **Valid Interface**: Use network devices from ``ifconfig``.

   - ``{{lan_interface}}``: the specified network interface.
   - ``{{ipmi_listen_range}}``: IP address of lan_interface("0.0.0.0" if interface has no IP).


   **Invalid Interface**: Network devices that don't exist.

   - ``{{lan_interface}}``: no binding device
   - ``{{ipmi_listen_range}}``: no range setting, which means user could only access ipmi_sim through kcs channel inside qemu OS.


.. _yamlBmcUsername:

- **bmc:username**

.. _yamlBmcPassword:

- **bmc:password**

.. _yamlBmcAddress:

- **bmc:address**

.. _yamlBmcChannel:

- **bmc:channel**

.. _yamlBmcLancontrol:

- **bmc:lancontrol**

.. _yamlBmcChassiscontrol:

- **bmc:chassiscontrol**

.. _yamlBmcStartcmd:

- **bmc:startcmd**

.. _yamlBmcStartnow:

- **bmc:startnow**

.. _yamlBmcPoweroffwait:

- **bmc:poweroff_wait**

.. _yamlBmcHistoryfru:

- **bmc:historyfru**

.. _yamlBmcConfigfile:

- **bmc:config_file**

.. _yamlBmcEmufile:

- **bmc:emu_file**

.. _yamlBmcIpmioverlanport:

- **bmc:ipmi_over_lan_port**

.. _yamlIpmiconsolessh:

- **ipmi_console_ssh**

.. _yamlIpmiconsoleport:

- **ipmi_console_port**

.. _yamlBmcconnectionport:

- **bmc_connection_port**

.. _yamlSerialsocket:

- **serial_socket**

    This attribute defines a `unix socket <https://en.wikipedia.org/wiki/Unix_domain_socket>`_ file to forward data.
    More specifically, it bridges ``socat`` and ``qemu`` for InfraSIM to forward input and output stream as a serial port.
    With this attribute designed, you will see ``socat`` starts with option ``unix-listen:<file>``,
    while ``qemu`` starts with a socket chardev ``-chardev socket,path=<file>,id=...``

    **Not Mandatory**

    **Default**: a file named ``.socket`` in `node workspace <https://github.com/InfraSIM/infrasim-compute/wiki/Compute-Node-Workspace>`_

    **Legal Values**: a valid file path, absolute or relative, to create such node

Networking
------------------------------------------------

#. Virtual server NAT or host-only mode, this is default mode implemented in infrasim-compute
    * vCompute is accessible ONLY inside Ubuntu host 
    * Software running in vCompute can access outside network if connecting Ubuntu host NIC with virtual bridge
    * Configuration YAML file can specify which NIC IPMI over LAN traffic flows through

    .. image:: _static/networking_nat.PNG
        :align: center

#. Bridge mode - single
    * Work as virtual switch
    * Connect BMC NIC and NICs in virtual compute together
    * Configuration YAML file controls how many NICs that virtual compute has and specify bridge they connect to

    .. image:: _static/networking_bridge_single.PNG
        :align: center

    .. note:: It requires setting up bridge and connect to NIC of underlying host in advance. 
    
    Here's steps for this example::

            # brctl addr br0
            # brctl addif br0 eth1
            # brctl setfd br0 0
            # brctl sethello < bridge name > 1
            # brctl stp br0 no
            # ifup br0

#. Bridge mode - multiple

    .. image:: _static/networking_bridge_multiple.PNG
        :align: center


.. hide_content::

            Virtual Power Distribution Unit - Robert - Under construction
            ------------------------------------------------

            Current Virtual PDU implementation only supports running entire virutal infrastructure on VMWare ESXi because it only supports functionality of simulating power control chassis through VMWare SDK.

            .. image:: _static/networkwithoutrackhd.png
                :align: center

