**WORK IN PROGRESS**

OpenStack Lab
=============

Deploy a reusable, reproducible, opinionated cloud based on [OpenStack](https://www.openstack.org/)
open-source software. The goals are:

1) to make it easy to deploy OpenStack,
2) using the least amount of hardware,
3) in a configuration that is similar to that of production environments.

We will be using the [RDO](https://www.rdoproject.org/) OpenStack distribution, which provides an
upgrade path to the fully supported
[Red Hat OpenStack Platform (RHOSP)](https://www.redhat.com/en/technologies/linux-platforms/openstack-platform).


Instructions
------------

1. [Hardware Requirements and Setup](HARDWARE.md)
2. [Prepare the Orchestrator](orchestrator/README.md)
3. [Prepare the Manager](manager/README.md)
4. [Deploy TripleO](undercloud/README.md)
5. [Prepare the Infrastructure](infrastructure/README.md)
6. [Deploy OpenStack](overcloud/README.md)
7. [Example Workloads](workloads/README.md)


Special Thanks
--------------

The following people may not show up in the commit history but deserve credit for contributing:

* Aaron Smith
* Andrew Bays
* Leif Madsen
* Ruslan Usichenko
* Yolanda Robla Mota
