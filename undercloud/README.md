Chapter 4: Deploy TripleO
=========================

We will now deploy [TripleO](https://docs.openstack.org/tripleo-docs/latest/) on our Manager, in
its own dedicated virtual machine. TripleO does two important jobs for us: 1) it deploys OpenStack
for us (chapter 6), and 2) it manages our infrastructure (chapter 5).

You might be wondering why our infrastructure manager is called "TripleO". It's actually named after
the pronunciation of "OoO" which is an acronym for "OpenStack on OpenStack".

Wait, what?

The idea is this: our infrastructure, which comprises various computers and storage devices and
network cards and switches, is itself a bit like a "cloud", though instead of being comprised of
virtual machines and virtual storage and virtual networks, which is what we mostly associate with
"cloud", it is (mostly) physical.

And OpenStack does support the provisioning of physical resources. This functionality is provided by
an OpenStack component called [Ironic](https://wiki.openstack.org/wiki/Ironic), which is used to
manage physical machines, or "bare metal". In a typical OpenStack cloud it handles the provisioning
of bare metal resources in addition to the more typical virtual resources.

You might already guess where this is going: in TripleO we will be using OpenStack itself, relying
heavily on Ironic, to manage our infrastructure as bare metal resources. The advantage is that we
can then use all the usual OpenStack tools to manage our infrastructure, including Heat and Mistral
orchestration.

The result is a deployment with *two* separate clouds: our infrastructure is called the
"undercloud", and our "actual" cloud, installed on the infrastructure, is called the "overcloud".
They could potentially be running on different versions of OpenStack!  

TripleO does not require a full-blown OpenStack deployment. For our lab we will be using a minimal
all-in-one installation within a virtual machine. But in production we can indeed divide the
undercloud OpenStack services among several machines, with full redundancy, high availability, and
scalability -- all the things that OpenStack already provides.

Confused? You really don't have to worry about it too much. The fact that our infrastructure
manager is itself implemented in OpenStack is just that: an implementation detail.

Note that in Red Hat OpenStack Platform (RHOSP) TripleO is called "Director".


Step 1: Prepare the Virtual Machine
-----------------------------------

Now that our Manager is set up as a hypervisor for hosting virtual machines we will prepare a
virtual machine for installation of our OpenStack infrastructure manager,
[TripleO](https://docs.openstack.org/tripleo-docs/latest/):

    undercloud/prepare

What [this script](prepare) does:

* Creates a virtual machine named `tripleo` based on a CentOS image using
  [cloud-init](https://cloudinit.readthedocs.io/en/latest/) to initialize it, configured by the
  files in [`configuration/libvirt/domains/tripleo/`](../configuration/libvirt/domains/tripleo/)
* Installs TripleO-client on it, which we will use in the next step to deploy TripleO (yes, it's
  complex enough that it deserves its own step)
* Installs Ceph Ansible playbooks on it, which TripleO will use later to deploy Ceph on our
  OpenStack infrastructure
* Creates and configures the "tripleo" user on it, which will be used by TripleO client to configure
  TripleO (though note that it does have sudo access, which will be necessary for *deploying*
  TripleO)    

Note that OpenStack packages, including those for TripleO and TripleO-client, are provided by the
[DLRN (pronounced "Delorean") project](https://dlrn.readthedocs.io/en/latest/). We will be
installing more than 500 packages in this step!
  
After it's done we will find some more files under our `workspace/` directory:

* `keys/tripleo@tripleo` and `keys/tripleo@tripleo.pub`
* `passwords/tripleo@tripleo`
* `ssh.config` is updated for `tripleo`

Logs and configuration files have been fetched to the `workspace/results/` directory. Some useful
ones are from:

* Manager: `/var/log/libvirt/qemu/tripleo.log`

We'll wait a few seconds for the `tripleo` virtual machine to start up and then we can connect to
it:

    ./ssh tripleo

Note that if you reboot the Manager this virtual machine will also be rebooted.


Step 2: Deploy TripleO Undercloud
---------------------------------

Now that we have the `tripleo` virtual machine ready with the TripleO client we can use it to
deploy TripleO:

    undercloud/deploy

The bulk of the work of this [this script](deploy) is handled by the
`openstack undercloud install` command. It is configured by files in
[`configuration/tripleo/`](../configuration/tripleo/). It itself comprises several steps and takes a
while to complete, ~ 15 minutes.

Internally it will be using [Puppet](https://puppet.com/) to orchestrate the installation and
configuration of the various OpenStack undercloud services as containers to be run by
[Podman](https://podman.io/). Containers allow for better isolation and portability. The undercloud
itself is installed using the OpenStack [Heat](https://docs.openstack.org/heat/latest/)
orchestration and [Mistral](https://docs.openstack.org/mistral/latest/) workflow services, which
internally use [Ansible](https://www.ansible.com/). The OpenStack container images are provided by
the [Kolla project](https://docs.openstack.org/kolla/latest/).

Logs and configuration files have been fetched to the `workspace/results/` directory. Some useful
ones are from:

* TripleO: `/home/tripleo/install-undercloud.log`
* TripleO: `/var/log/containers/mistral/engine.log`
* TripleO: `/var/log/containers/heat/heat-engine.log`
* TripleO: `/home/tripleo/undercloud/undercloud-passwords.conf`
* TripleO: `/home/tripleo/undercloud/container-images.yaml`

We can now access the undercloud's `openstack` command via a
[shortcut](../openstack) that uses the `stackrc` file created in the `tripleo`
user's home directory), e.g.:

    undercloud/openstack network list

The `openstack` command is mostly documented
[here](https://docs.openstack.org/python-openstackclient/train/cli/), though note that it is
extensible. Ironic adds
[these commands](https://docs.openstack.org/python-ironicclient/latest/cli/osc_plugin_cli.html)
and TripleO adds
[these commands](https://docs.openstack.org/python-tripleoclient/latest/commands.html).

We can directly access the individual service containers on any machine via the
[`./podman`](../podman), [`./podman-bash`](../podman-bash), and
[`./podman-restart`](../podman-restart) shortcuts. Run any of them without arguments to get a list
of available service containers. For example, to get a shell into the `mistral_engine` service
container:

    ./podman-bash tripleo mistral_engine
    cat /var/log/mistral/engine.log

(Note that `podman-bash` does not ssh into the service container, but rather explicitly executes
bash within the container. These are lightweight containers that are not running ssh servers.)

Podman's CLI intentionally mimics Docker's, so if you're familiar with commands like `docker ps`
just replace the command with `podman`, e.g.: `./podman tripleo ps`.

We don't actually have to access the containers to get to the logs, because the log directories
are shared from the host virtual machine at `/var/log/containers/`, for example the Mistral log we
saw above is also here:

    ./ssh tripleo
    cat /var/log/containers/mistral/engine.log

Unlike logs, which are shared from the host, configuration files for the containers are *imported*
from the host when they start up, from its `/var/lib/config-data/puppet-generated/` directory. So,
to manually edit `ironic.conf`:

    ./ssh tripleo
    vi /var/lib/config-data/puppet-generated/ironic/etc/ironic/ironic.conf

To use the file we will need to restart the service container:

    sudo systemctl restart tripleo_ironic_conductor.service

Editing files within the container is a bit trickier. We'll need to find the mount location for the
container's filesystem in order to access it from the host, e.g.:

    ./ssh tripleo
    sudo podman mount ironic_inspector
    ...


Next
----

[Continue to Chapter 5: Prepare the Infrastructure](../infrastructure/README.md)
