Kargo-Setup
===========

We found that there are a few fixes for a VM deployment of a Kargo kubernetes environment, and this ansible play tries to cover many of them.

There are a few elements that may need to be manually changed based on your environment, most significantly, the nameservers in the resolv.conf file, and the interface name in the ifcfg-eth0 file and the target location.

One key setup item.  The scripts currently expect that you have already built a
base VM, and named it VMtemplate, and that it has an id of 100.  In fact there
is also a model for a second PVE node in a cluster configuration (but without
  shared storage), and that may also need to be modified in the deploy.yml script.

Also in the deploy script, one can modify the start_stop and absent_present
parameters in order to create and start or stop and delete target vms.

The Fix script will resolve a number of issues in the deployed VMs that will
cause Kargo to crash.

NOTE: It appears that the proxmox community code requires ansible 2.1 or greater 
deployed via pip or easy_install.  Brew installed ansible does not seem to have
the appropraite modules installed.  Using a virtualenv for this task is a 
reasonable approach, and if you do, don't forget to also install the python
netaddr module:

```
pip install virtualenv
virtualenv .
. bin/activate
pip install ansible netaddr
```

First launch the deploy script (after you've created your template VM):

```
ansible-playbook -i inv deploy.yml
```

These scripts need to be launched separately, and one will likely need to adjust
the addresses in the inv file.  I have yet to discover a way to learn the IP
addresses assigned to the deployed VMs, so a manual discovery is needed,
unfortunately, this often means loging in to the VMs via the Console to learn
the eth0 ip address, perhaps with:

```
ip a s eth0
```
Once these addresses are learned, update the inv file with both the actual addresses and those that you'd like the systems to have statically defined (or
  use the same addresses as the DHCP assigned addresses, but do apply to both
  the ssh_host and ip parameters).

Now you can run the fix it script:

```
ansible-playbook -i inv fix.yml
```

Kargo
=====

The scripts here don't download kargo, but you can clone a copy either directly
into the local directory or into another directory:

```
git clone https://github.com/kubernetes-incubator/kargo
```

then copy the inventory file from the fix script into the kargo directory:

```
cp inventory/invenory kargo/inventory/
```

And finally launch the kargo cluster installer:

```
cd kargo
ansible-playbook -i inventory cluster.yml
```

It will still be necessary to create a kubernetes kubelet config file, and run
this on a master node, or follow the direction in the vharmony or sparta projects
to get kubectl configured.
