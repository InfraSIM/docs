Configuration
=========================



Virtual Server Configuration file
------------------------------------------------

There's one central virtual server configuration file which is /etc/infrasim/infrasim.yml (`source code <https://github.com/InfraSIM/infrasim-compute/blob/master/template/infrasim.yml>`_). All adjustable parameters are defined in this file. This is only one place if you want to customize or make adjustment on the virtual server node. While not all the options are explicitly listed in this file for purpose of simplicity. However there's one example configuration file - /etc/infrasim.full.yml.example (`source code <https://github.com/InfraSIM/infrasim-compute/blob/master/etc/infrasim.full.yml.example>`_) - listed all supported parameters and definitions. By referring content in example file, you can modify real file infrasim.yml and then restart infrasim-main service, new properties will take effect.

Here's full list of the example configuration file; every single key-value pair is supported to be add/modify in your real-in-use infrasim.yml::

    # Unique identifier
    name: node-1

    # Node type is mandatory
    type: quanta_d51

    compute:
        kvm_enabled: true
        numa_control: true
        cpu:
            model: host
            features: +vmx
            quantities: 8
        memory:
            size: 4096
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
        networks:
            -
                network_mode: bridge
                network_name: br0
                device: vmxnet3
            -
                network_mode: bridge
                network_name: br0
                device: vmxnet3
        ipmi:
            interface: bt
            host: 127.0.0.1
        smbios: chassis/node1/quanta_d51_smbios.bin
    bmc:
        interface: br0
        username: admin
        password: admin
        emu_file: chassis/node1/quanta_d51.emu

    # Renamed from telnet_listen_port to ipmi_console_port, extracted from bmc
    ipmi_console_port: 9000

    # Used by ipmi_sim and qemu
    bmc_connection_port: 9100

    # Used by socat and qemu
    serial_port: 9003



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

Virtual Power Distribution Unit - Robert - Under construction
------------------------------------------------

 Current Virtual PDU implementation only supports running entire virutal infrastructure on VMWare ESXi because it only supports functionality of simulating power control chassis through VMWare SDK.

 .. image:: _static/networkwithoutrackhd.png
    :align: center