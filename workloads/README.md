Chapter 7: Example Workloads
============================


Step 1: Common Resources
------------------------

Now we have an OpenStack instance, but it's still very bare. Let's add the basics:

    ./prepare-cloud

This will create:

* A `public` network in the `admin` project (subnet 10.0.2.0/24) 
* Conventional public flavors, such `m1.tiny` and `m1.large`
* Common images, such as `common.centos7` in the `common` project


Step 2: Hello World
-------------------

./ssh hello-world "python -m SimpleHTTPServer 8080"

We have a script to create a "hello world" example:

    ./create-simple-project test hello-world

This creates a `test` project with its own `test.private` network, subnet, and router that used the
`public` network as its external gateway. We make sure the project's `default` security group rules
allow for ping (ICMP) and ssh. We then create a server named `hello-world` with a new `test`
keypair, create and attach a 2 GB volume to it, and finally assign it a floating IP (again on the
`public` network).

You can ssh into the fully functioning virtual machine using the private key, e.g.:

    ./ssh-virtual -i ~/openstack-keypairs/test centos@10.0.0.210

You must manually format and mount the attached volume:

    sudo mkfs.ext4 /dev/vdb
    sudo mount /dev/vdb /mnt

When done testing, you can delete the project and all its resources using our `delete-project`
script:

    ./delete-project test

Note that deleting a project is tricky in OpenStack, because the simple `openstack project delete`
command will *not* delete associated resources and instead leave them orphaned and dangling. It's
likewise tricky to delete networks, because the operation will fail if ports on the network are in
use. Yet another issue is that ports on the network can be created in *other* projects. This script
does the heavy lifting to ensure that all resources associated with the project are safely deleted.
