---
title: "The Beginning of secretProject"
date: 2022-09-10T10:48:23-04:00
draft: false
toc: false
images:
tags: 
  - dev
  - secretProject
---

>What's in a name?  That which we call a rose by any other name would smell just as sweet.
>
>-William Shakespeare
---

# Where to start?
Ask a thousand software enthusiasts what the two most difficult things about software development is and you'll likely aggregate to:

1.  Deciding what to develop
2.  Naming what you want to develop

---

# What to develop?
About 5 years ago I came up with the idea for a personal project (hereinafter called secretProject), an application that would mainly serve two purposes:

1.  Help me (and my family) with our daily lives
2.  Expand my knowledge of the software development lifecycle

---

# How to get there?
In order to achieve these goals, I have mentally been fine tuning secretProject on how I want to build it to (hopefully) meet these features:

*  Platform agnostic
   *  Ideal - The application should be able to run on bare metal, virtual machines, containers, cloud platform, etc.
   *  Realistic - To save on costs, secretProject will be containerized and deployed to a Kubernetes cluster running on virtual machines hosted on a local physical server I have in the basement.
*  Automated provisioning
   *  Ideal - In the event of a disaster, a ci/cd pipeline should be able to provision the application based on it's last healthy configuration.
   *  Realistic - KVM (virsh), Ansible, and Terraform will be triggered manually at first.
*  Mobile ready
   *  Ideal - I'd love to build a mobile Android app so I can submit content to secretProject for processing.
   *  Realistic - All RESTful queries will have to be performed locally on the same subnet as secretProject.

---

# Where will this stuff go?

With the idea of secretProject in mind and infrastructure on hand, what happens next?  I know secretProject will be containerized and deployed in a Kubernetes cluster which is running on KVM virtual machines so let's start there.

I built up a homelab/gaming server about 10 years ago running with hardware spec:

*  Asus M5A99FX PRO R2.0 motherboard
*  AMD FX-8320 8 core 3500MHz cpu
*  32GB Corsair Vengeance DDR3 memory
*  Nvidia GeForce GTX 650 Ti Boost video card
*  Samsung SSD 850 EVO 1TB hard drive

I've had AlmaLinux 8.5 running on it for about a year now (RIP CentOS) which is stable enough to run as a KVM hypervisor with 3-5 vm's depending on compute metrics needed.

---

# What happens next?

I know what I want to build, where to build it, and how to build it, but what's our plan?  Let's keep it simple and start with just:

1.  Provision 2-3 vm's on my lab server (hereinafter called soulbox) using libvirt/virsh

See you in the next post...

---