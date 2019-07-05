Chapter 3: Prepare the Manager
==============================

In production OpenStack environments there's no single controller, instead there would be many
"controllers" of various kinds running on dedicated hardware with redundancy. For our lab we will
converge all these roles into a single machine. However, it's important to understand that this
machine will be fulfilling several roles. In order to keep these roles isolated we will be running
each role in a virtual machine, which will also make it much easier to tear down and rebuild roles. 

Our Manager will need two NICs, with one NIC on our work LAN and one NIC on a dedicated OpenStack
control plane LAN.

/diagram/


Step 1: Install the Operating System
------------------------------------

Start with installing [CentOS](https://www.centos.org/). Tested with version 7.6. The minimal
install (with no desktop environment) is good enough. We will need:

* The root user password
* Its IP address on the work LAN 

Our scripts in the next steps will be making changes to this machine. We will do the best we can to
isolate our work: most of it will be under the user "manager", and most of what we will run will be
inside virtual machines.

However, some work will have to be done in root: installing some utility packages and setting up
custom network bridges as well as setting up the virtual machines themselves.


Step 2: Prepare the Manager
---------------------------

Let's edit [`configuration/environment`](../configuration/environment) and set
`MANAGER_OFFICE_IP_ADDRESS` to point to our Manager. We can use a host name. Then run:

    manager/prepare

We will be prompted only once for the root password for the Manager, which we will use to authorize
a keypair for ssh.

What [this script](../manager/prepare) does:

* Installs [libvirt](https://libvirt.org/) and related tools, such as
  [VirtualBMC](https://docs.openstack.org/virtualbmc/latest/), which provides an
  [IPMI](https://en.wikipedia.org/wiki/Intelligent_Platform_Management_Interface) for libvirt
  virtual machines
* Sets up network bridges to be used by our libvirt virtual machines, configured by the files in
  [`configuration/libvirt/networks/`](../configuration/libvirt/networks/).
* Creates and configures the "manager" user

After it's done we will find some files under our `workspace/` directory:

* `keys/root@manager` and `keys/root@manager.pub`
* `keys/manager@manager` and `keys/manager@manager.pub`
* `passwords/manager@manager`
* `ssh.config` is updated for `manager-root` and `manager`

Our scripts will later on add more keys and passwords and keep `ssh.config` updated. That file is
especially useful: it configures custom hosts that we can use it to ssh from our Orchestrator to our
lab machines, including virtual machines running inside the Manager, OpenStack, and Kubernetes. The
[`./ssh`](../ssh) and [`./rsync`](../rsync) shortcuts use this config. Examples of use:

    ./ssh manager
    ./ssh manager-root "ls -al"
    ./rsync manager:/etc/fstab .

Now that we have libvirt installed we can also use its CLI,
[virsh](https://libvirt.org/virshcmdref.html), via our [`manager/virsh`](../manager/virsh) shortcut: 

    manager/virsh net-list

You should see the three networks we created in this step. 

(Note that if we have libvirt installed on our Orchestrator it would also be possible to connect
[remotely](https://libvirt.org/remote.html) to the Manager's instance via a "qemu+ssh:" or
similar URI.)


How to Reset
------------

You can start from scratch if you experience any failures in the above or later steps. This script
will delete all the virtual resources from the Manager:

    manager/clean

You may need to run [this script](../manager/clean) several times until it completes successfully
because some libvirt resources might be in an indeterminate state when you run it.

You can also combine `clean` and `prepare`:

    manager/prepare -c


Next
----

[Continue to Chapter 4: Deploy TripleO](../undercloud/README.md)
