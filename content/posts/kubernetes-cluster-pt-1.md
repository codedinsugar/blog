---
title: "secretProject part 1 - fun with KVM"
date: 2022-10-01T07:31:12-04:00
draft: false
toc: false
images:
tags: 
  - dev
  - secretProject
---

>Unfortunately, no one can be told what the Matrix is. You have to see it for yourself.
>
>-Morpheus
---

# Previously on codedinsugar.com

Continuing where we [left off](https://codedinsugar.com/posts/the-beginning-of-secretproject/#what-happens-next), I wanted to build a kvm vm's to run a Kubernetes cluster for the secretProject platform.


Since the early 2000's I've used several virtualization hypervisors, including:

* Hyper-V
* KVM
* VMware
* VirtualBox
* XenServer
* Not including AWS EC2 and Docker

But for the purpose of secretProject and the need to self-host on a single bare metal machine, I needed the most lightweight solution and luckily without the need a ton of features.

---

# Enter KVM

KVM, which stands for Kernel-based Virtual Machine, is a virtualization solution which is typically native to the Linux kernel.  Of all the hypervisors that I'm familiar with, KVM will run as close to cpu as possible, making it the fastest and most lightweight.  Perfect.

This post is not a KVM tutorial.  If you're interested in learning more about it, [Josh Rosso's YT channel](https://www.youtube.com/watch?v=HfNKpT2jo7) is a really, really good resource.  He also wrote a book, [Production Kubernetes](https://www.oreilly.com/library/view/production-kubernetes/9781492092292/).  If you're on a RHEL distribution like I am, RedHat has extensive documention under their [Virtualization Deployment and Administration Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/index) which has everything you need to get started with KVM.

---

# Click vs Code

So fast forward to after all the kvm, qemu, libvirt, and all pre-requisites and dependencies are installed and we're ready to begin.  With my setup, I had two options of creating the vm's:

1. Use the [Cockpit UI](https://cockpit-project.org/)
2. Use [`virt-install`](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-guest_virtual_machine_installation_overview-creating_guests_with_virt_install) 

Using Cockpit UI is a little too easy, just browse to https://localhost:9090/machines, click Create VM, choose parameters and done.  I could easily do that anytime I need to create new vm's or re-create one with a specific image.  But remember from the [secretProject requirements](https://codedinsugar.com/posts/the-beginning-of-secretproject/#how-to-get-there), I want an automated solution that is highly configureable.  The problem with using Cockpit UI is that once the vm is created and started, you still have to manually click through the OS configuration screens.  Sure this task only takes 5-10 minutes but it's still [toil](https://sre.google/sre-book/eliminating-toil/).  So `virt-install` it is then :)

---

# `virt-install` Dry-run

Building a vm with the `libvirt` library is fairly simple and the RedHat docs [here](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-guest_virtual_machine_installation_overview-creating_guests_with_virt_install) provide a decent walkthough on the process.  Since I need to create three vm's (1x master and 2x worker), three shell scripts can get things started:

```
#!/usr/bin/env bash

echo "Creating vm1"
 
virt-install \
    --connect qemu:///system \
    --virt-type kvm \
    --name vm1 \
    --memory 8192 \
    --vcpus 2 \
    --location /var/lib/libvirt/images/AlmaLinux-8.5-x86_64-minimal.iso --unattended \
    --disk size=20 \
    --network bridge:virbr0 \
    --graphics=vnc \
    --console pty,target_type=serial \
    --wait 0 \
    --initrd-inject /home/codedinsugar/dev/cicd/gitlab/kvm/scripts/almalinux85-create-vm1-kickstart.cfg \
    --extra-args "ks=file:/almalinux85-create-vm1-kickstart.cfg console=tty0 console=ttyS0,115200n8"
```

The parameters should be self explanatory but notice the use of the kickstart file in `--initrd-inject` and `--extra-args` which looks like this:
```
#version=RHEL8
# Use graphical install
graphical

repo --name="Minimal" --baseurl=file:///run/install/sources/mount-0000-cdrom/Minimal

%packages
@^minimal-environment
@headless-management
@development
@standard
@system-tools
kexec-tools

%end

# Keyboard layouts
keyboard --xlayouts='us'
# System language
lang en_US.UTF-8

# Use CDROM installation media
cdrom

# Run the Setup Agent on first boot
firstboot --disable

ignoredisk --only-use=vda
autopart
# Partition clearing information
clearpart --none --initlabel

# System timezone
timezone America/New_York --isUtc

# Root password
rootpw --iscrypted $blahblahblah
user --groups=wheel --name=foo --password=$blahblahblah/ --iscrypted --gecos="foo"
user --name=ansible --password=ansible

# Network information
network  --bootproto=static --device=enp1s0 --gateway=192.168.122.1 --ip=192.168.122.25 --nameserver=192.168.122.1 --netmask=255.255.255.0 --ipv6=auto --activate
network  --hostname=vm1

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end

%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end

reboot
```

A few things of note here:
1. Prepping each vm for ansible with `user --name=ansible --password=ansible` :)
2. The `network` lines are defining layer 3 for each specific vm.

After defining any vm specific settings in the shell script and the kickstart file, we can see the vm's with:

```
sudo virsh list --all

 Id   Name           State
------------------------------
 1    vm1    running
 2    vm2    running
 3    vm3    running
 ```

Wunderbar!