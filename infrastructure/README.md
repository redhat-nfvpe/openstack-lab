Chapter 5: Prepare the Infrastructure
=====================================

As mentioned in chapter 4, OpenStack is modular such that each participating machine may have a
different role. For the purpose of this lab our "compute" role will have its own dedicated hardware,
while the "controller" role will live on the Manager as a virtual machine.

> Note that it is technically possible for the "compute" role to also be a virtual machine, such
that our entire OpenStack deployment would run on a single physical machine. Though it would work,
it would introduce an unwanted side effect: our OpenStack workloads would be running in *nested
virtualization*. This advanced feature has various limitations and is often accompanied by a
significant hit to performance. For this reason we are installing the "compute" role on dedicated
hardware.


Step 1: Prepare the Inventory
-----------------------------

Before we introspect our infrastructure (in the next step) we need to create the extra virtual
machine that will be part of it and also prepare the operating system images that will used by all
infrastructure machines during the introspection process, as well as different images that will
be used to install the OpenStack services on them (in step 3).

    infrastructure/prepare

What [this script](../infrastructure/prepare) does:

* Creates operating system images that TripleO will use to introspect and provision the
  infrastructure nodes using PXE
* Creates a virtual machine named `openstack-controller` configured by the files in 
  [`configuration/libvirt/domains/openstack-controller/`](../configuration/libvirt/domains/openstack-controller/)

The bulk of the work in this step will be handled by `openstack overcloud image build`, which
can take a while to complete, ~ 5 minutes.

> It's also possible to download ready-made images from the RDO project. From our experience the
repository is rather slow and does not offer an advantage over building them locally.

After it's done we can see that the images have been uploaded into TripleO:

	manager/undercloud/openstack image list

In TripleO's `tripleo` user's home directory:

* `ironic-python-agent.kernel` and `ironic-python-agent.initramfs` are
  small images used by TripleO's Ironic for introspection
* `overcloud-full.qcow2` and `overcloud-full.initrd` are the OpenStack
  installation images

Logs and configuration files have been fetched to the `workspace/results/` directory. Some useful
ones are from:

* Manager: `/var/log/libvirt/qemu/openstack-controller.log`
* Manager: `/var/log/virtualbmc/virtualbmc.log`
* TripleO: `ironic-python-agent.log`
* TripleO: `overcloud-full.log`

Note that when creating the `openstack-controller` virtual machine we also enable
[VirtualBMC](https://docs.openstack.org/virtualbmc/latest/) to access it. This is necessary because
TripleO, our OpenStack infrastructure manager, does not inherently support virtual machines as part
of the infrastructure due to its reliance on Ironic. VirtualBMC will thus provide an 
[IPMI](https://en.wikipedia.org/wiki/Intelligent_Platform_Management_Interface) to the virtual
machine so that it could be remote controlled like a physical machine.

We can test this using [ipmitool](https://github.com/ipmitool/ipmitool), which we installed for this
purpose on the Manager in the previous chapter. Use [this shortcut](../manager/ipmitool):

    manager/ipmitool 6230 power status

The first argument is the IPMI port: each virtual machine supported by VirtualBMC gets its own
dedicated port. 6230 is the port we configured for `openstack-controller`. We also have a
[shortcut](../manager/vbmc) to VirtualBMC's client CLI:

    manager/vbmc list


Step 2: Introspect the Inventory
--------------------------------

Our infrastructure is not yet ready for deploying OpenStack on it. We must first "introspect" our
infrastructure nodes, which in our case are the physical "compute" node and the virtual
"controller" node:

    infrastructure/introspect

The bulk of the work of this [this script](../infrastructure/introspect) is handled by the
`openstack overcloud node introspect` command. It is configured by
[`configuration/infrastructure-inventory.yaml`](../configuration/infrastructure-inventory.yaml).

During introspection each node is provisioned the `ironic-python-agent` minimal operating system
image via PXE (which we prepared in the previous step), which boots up and reports back to TripleO
with a detailed profile of its hardware. TripleO will store this profile in its database.

This can take a while, 15 minutes and more. How long exactly depends on our hardware.

While it runs it could be useful to see the changing state of the infrastructure nodes. We can
set up a `watch` like so:

    watch manager/undercloud/openstack baremetal node list

We should see the nodes powering on, then changing their provisioning state from "enroll" to
"verifying" to "manageable". (If we set `clean_nodes` to true in `undercloud.conf` then we will
also see "cleaning" and "clean wait".) When introspection finishes successfully for all nodes they
will be set at once to "available".

manager/undercloud/openstack overcloud profiles list

> TODO: machines without IPMI. "pm_type: manual-management". need monitor-keyboard-mouse.

Logs and configuration files have been fetched to the `workspace/results/` directory. Some useful
ones are from:

* TripleO: `/home/tripleo/introspection/` has the introspection results in JSON
* TripleO: `/var/log/containers/ironic/ironic-conductor.log`
* TripleO: `/var/log/containers/ironic-inspector/ironic-inspector.log`
* TripleO: `/var/log/containers/ironic-inspector/ramdisk/` contains `tar.gz` files for each
  infrastructure node workflow, which internally contain the `journal` of the introspection process
* TripleO: `/var/log/containers/mistral/engine.log`
* TripleO: `/var/log/containers/heat/heat-engine.log`

In TripleO's `tripleo` user's home directory:

* `/home/tripleo/infrastructure-nodes.yaml` copied from
  `configuration/tripleo/infrastructure-nodes.yaml`


Next
----

[Continue to Chapter 6: Deploy OpenStack](../overcloud/README.md)
