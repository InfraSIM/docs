Puffer - infraSIM test
-------------------------------------
Puffer is test framework developed for InfraSIM testing. Source code is in `InfraSIM/test <https://github.com/InfraSIM/test>`_. It is a framework which can be easily extended to test products of different type, for example, standalone or web-based software and firmware. Here's its block diagram.

          .. image:: _static/puffer_architecture.png
             :align: center
             
For any test target specified, those target behavior encapsulation need to be developed and a set of tests cases need to be added on top of encapsulation layer. `Write test case </how_tos.html#write-test-case>`_ described how to work out one test cases against infraSIM. Below sections introduced all details about setting up buffer and execute infraSIM testing with it.

Setup environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Refer to the section 7.1 Physical Servers and ESXi Environment Setup.

Code::

    git clone https://github.com/InfraSIM/test.git

Install necessary package::

    sudo python test/install/PackageInstall.py


Define environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
You can see a configuration file example in test/configure/stack_example.json.
To test  your environment, you must define your environment in a file, and it must be in a valid JSON format.


#. Define the overall test environment.

   *  `(Optional)` `vRackSystem </userguide.html#vracksystem-user-manual>`_  - The test may leverage vRackSystem and have REST talk.
   * available_Hypervisor - A list of hypervisors information. If your test has to handle hypervisors, this attribute is a required.
   * vRacks - A list of virtual racks you have built.

   ::

    {
        "vRackSystem": {},
        "available_HyperVisor": [],
        "vRacks": [],
    }

#. `(Optional)` Define `vRackSystem </userguide.html#vracksystem-user-manual>`_  key information for REST interaction, this definition can be an empty dictionary::

    {
        "protocol": "http",
        "ip": "192.168.1.1",
        "port": 8888,
        "username": "admin",
        "password": "admin",
        "root": "/api/v1"
    }

#. Specify hypervisor information using available_HyperVisor.

   For a single definition, here is an example::

    {
        "name": "hyper1",
        "type": "ESXi",
        "ip": "192.168.1.2",
        "username": "username",
        "password": "password"
    }

#. Specify a list of vRacks. Each definition includes:

   * name - any name you like.

   * hypervisor - The hypervisor you used in above definition. All virtual node, PDU, and switch are deployed on this hypervisor.

   * vPDU - A list of virtual PDU definition. The list can be empty.

   * vSwitch - A list of virtual switch definition. The list can be empty.

   * vNode - A list of virtual node definition. The list can be empty.


   They are organized in the following list::

    {
        "name": "vRack1",
        "hypervisor": "hyper1",
        "vPDU": [],
        "vSwitch": [],
        "vNode": []
    }

#. Specify a list of virtual PDUs. For each definition, you need to maintain:

   * name - virtual PDU's name in hypervisor

   * datatstore - on which datastore this PDU is deployed.

   * community - control community for SNMP access.

   * ip - PDU IP

   * outlet - A mapping of outlet to corresponding control password.

   Example::

    {
        "name": "vpdu_1",
        "datastore": "Datastore01",
        "community": "foo",
        "ip": "172.31.128.1",
        "outlet": {
            "1.1": "bar",
            "1.2": "bar",
            "1.3": "bar"
        }
    }

#. vSwitch is currently not enabled.

#. Specify a list of virtual nodes. For each definition, you need to maintain:

   * name - The virtual node's name in hypervisor.

   * datastore - The datastore this node is deployed on.

   * power - A list of power control connection, each connection defines a specific PDU and outlet, you may have two power control, if this list is empty, node will not be controlled by any PDU.

   * network - A definition for connection to virtual switch, currently not used.

   * bmc - A definition on how to access virtual BMC of this node, including IP, username and password for ipmi over LAN access.


   Example::

    {
        "name": "vnode_a_20160126114700",
        "datastore": "Datastore01",
        "power": [
            {"vPDU": "vpdu_1", "outlet": "1.1"},
        ],
        "network": [],
        "bmc": {
            "ip": "172.31.128.2",
            "username": "admin",
            "password": "admin"
        }
    }

   **Verify every IP is available from your test execution environment!**

   **Verify PDU can access substream hypervisor!** (see chapter 7.1.3 vPDU Configuration for detail)

Case Runtime Data
~~~~~~~~~~~~~~~~~~~~~~~~
Case Runtime Data used to maintain some specific data for different test objects. These data generally require the user to add and update manually. For example, if you want to test one type of sensor for multiple nodes, you need to add and update sensor ID corresponds to each node.

#. Configuration file:

   Case Runtime Data is defined in the json file which have same name with case script. If name of case script is T0000_test_HelloWorld.py, the name of runtime data shall be T0000_test_HelloWorld.json.

   Here's an example::

    [
        {
            "name_1": "value_1",
            "name_2": "value_2"
        }
    ]
    
   If your configuration json like above, you can get "value_1" by call self.data["name_1"] in test case.

   Here's another example::

    [
        {
            "node_1": "0x00",
            "node_2": "0x01"
        },
        {
            "node_1": "0x02",
            "node_2": "0x03"
        }
    ]

   If your configuration json has two objects in an array like above, same case shall be run twice for each runtime data.

   You will get "0x00" by call self.data["node_1"] in test case for the first time, and "0x02" for the second time.

#. Test Result:

   You shall get two separate result and a summary. Case's final result is the worst result for all execution.

   For example, if the case "failed" in first time and "passed" in second time, the final result is still "failed", the summary will list all run results.


Run test
~~~~~~~~~~~~~~~~~~~~~~~~~
Trigger test::

    cd test
    python puffer.py -s infrasim --stack=<your_configuration>

<your_configuration> can be an absolute or related path of your configuration file.
About how to run test, please check readme for detail::

    cat README.md

You log file is kept in a folder of log/InfraSIM, each test task is packaged in a folder
with time stamp as it's folder name.


**Notice:** Please follow `How to write test case </how_tos.html#write-test-case>`_ to write a new test case.
