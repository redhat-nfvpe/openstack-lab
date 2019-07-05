Chapter 2: Prepare the Orchestrator
===================================

The Orchestrator machine is where we will be running our scripts.

It just needs a minimal
Linux-like command line environment, which can run on in a virtual machine or on
[Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/about). 
Specifically we will need bash, ssh, sshpass, rsync, sed, grep, and awk. We do not require root
access.

Most of these environments will already have everything we need out of the box, except perhaps for
sshpass. To install it on Fedora:

    sudo dnf install sshpass

Alternatively, it *is* possible run orchestration on the Manager machine, in which case the Manager
IP address would be `localhost`. Just note that our scripts will create a dedicated user called
`manager` on this machine, so we will want to run orchestration on a different user.

To start, just clone this repository:

    git clone https://github.com/redhat-nfvpe/openstack-lab.git


Next
----

[Continue to Chapter 3: Prepare the Manager](../manager/README.md)
