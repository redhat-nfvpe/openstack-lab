Chapter 1: Hardware Requirements and Setup
==========================================

Our scripts will be running on what we'll call our "Orchestrator" computer. We'll also be fetching
logs from our lab to it, under the `workspace/` directory here. We will be making no changes to this
machine so it should be fine to use an existing workstation.

The lab itself comprises a minimum of 2 computers and two LANs:


/diagram/


Manager Computer
----------------

This is the computer that manages and provisions the OpenStack infrastructure.

It must be capable of running virtual machines and have two network interfaces, but otherwise can
be simple and cheap consumer-grade equipment.


Compute/Storage Computer (one or more)
--------------------------------------

This is the computer that will host our OpenStack workloads, both virtual machines ("servers") and
storage ("volumes"). We can add more such computers to increase our capacity, and indeed separate
the "compute" and "storage" roles. (We have chosen to combine them, in what's known as
["hyper-converged infrastructure" (HCI)](https://en.wikipedia.org/wiki/Hyper-converged_infrastructure),
in order to minimize our hardware requirements.)

Like the Manager, it must be capable of running virtual machines and have two network interfaces.
However, because this computer is managed and provisioned by the Manager, it is highly recommended
that it have a motherboard that supports
[IPMI](https://en.wikipedia.org/wiki/Intelligent_Platform_Management_Interface) or any of the other
remote power control technologies supported by
[Ironic](https://docs.openstack.org/ironic/latest/admin/drivers.html). This would mean more
expensive server- or workstation-class equipment.

Indeed, depending on the kinds of workloads you will want to run on your cloud, you might also need
it to have many CPU cores, lots of RAM, and lots of storage.


Office LAN
----------

All machines should be connected to this LAN and this LAN should also allow egress to the Internet.
This will allow for your Orchestrator to connect to the Manager and your cloud workloads, and for
software to be downloaded from the Internet and installed on the lab computers.

Another important use is for the Manager to access the Compute/Storage computer's IPMI. So, if the
Compute/Storage computer supports IPMI on only one of its network interfaces, then that is the one
you *must* connect to the office LAN.

We need static IP addresses for the Manager and the Compute/Storage IPMI. You should set these by
by configuring your DHCP server according to their MAC addresses.

Important: The Compute/Storage "normal" networking MAC address is *different* from its IPMI MAC
address. Indeed that network interface will have *two* IP addresses when running. We specifically
only need a static IP address for the IPMI MAC address, *not* the "normal" networking MAC address,
which can get a dynamic IP address from DHCP.

Otherwise we have no special requirements for the office LAN, so it can be implemented using a
simple and cheap consumer-grade router/switch or indeed be your existing office LAN setup.


Datacenter LAN
--------------

This LAN connects the Manager and Compute/Storage computers. It must be isolated and dedicated
because we will be running our own DHCP servers on the Manager as well as TFTP servers to provision
the Compute/Storage computer(s).

An additional requirement is that the LAN support
[virtual LAN (VLAN)](https://en.wikipedia.org/wiki/Virtual_LAN) technology. (In a production
environment, a "real" datacenter, these may very well be implemented as several isolated and
dedicated physical LANs.)

This means that your equipment *must* support VLAN configuration. Not all consumer-grade switches 
do, but most "managed" switches will have this feature. In the switch you will need to configure the
following VLANs, and optionally name them:

* Tag 10: External
* Tag 20: InternalApi
* Tag 30: Storage
* Tag 40: StorageMgmt
* Tag 50: Tenant

Again, make sure that this LAN is isolated. Specifically we want to avoid having additional DHCP
servers other than our own.


Next
----

[Continue to Chapter 2: Prepare the Orchestrator](orchestrator/README.md)
