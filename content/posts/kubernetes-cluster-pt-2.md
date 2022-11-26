---
title: "secretProject part 2 - fun with Ansible"
date: 2022-10-20T10:38:10-05:00
draft: false
toc: false
images:
tags: 
  - dev
  - secretProject
---

>Mr. Wizard!  Get me the hell out of here!.
>
>-Neo
---

# Previously on codedinsugar.com

Continuing where we [left off](https://codedinsugar.com/posts/kubernetes-cluster-pt-1/), we've used `libvirt/virt-install` to successfully create three vm's for our Kubernetes cluster for secretProject.  So what happens next?

# Let's play with Ansible

I've used Ansible often in the past but it was never a core part of my daily work.  There is a lot to learn about Ansible and I highly suggest you check out their [site](https://www.ansible.com/resources/get-started), especially if you're interested in the SRE/DevOps space.

This post is NOT an Ansible tutorial and if you're following along then these assumptions are made:

1. An `ansible` user has already been created where needed
2. SSH keys have been copied over with `ssh-copy-id`
3. `/etc/hosts` aligns with ansible inventory as needed
4. An initial test has been performed with `ansible $hosts -m ping`

I'm running ansible-core 2.13.3 and I need to install the following collections:

```
 ansible-galaxy collection install ansible.posix
 ansible-galaxy collection install community.general
 ```

To build the cluster 95% of the way, I used this playbook with multiple tasks:

```
- name: Kubernetes cluster setup
  hosts: lab
  become: yes
  
  tasks:
    - name: Disable selinux
      ansible.posix.selinux:
        state: disabled
    - name: Remove swapfile from /etc/fstab
      mount:
        name: "{{ item }}"
        fstype: swap
        state: absent
      with_items:
        - swap
        - none
    - name: Disable swap
      command: swapoff -a
      when: ansible_swaptotal_mb > 0
    - name: Configure firewalld rules
      firewalld:
        permanent: yes
        immediate: yes
        state: enabled
        port: "{{ item.port }}/tcp"
      with_items:
        - {port: "6443"}
        - {port: "2379-2380"}
        - {port: "10250-10252"}
        - {port: "10255"}
    - name: Add system modules
      modprobe:
        name: br_netfilter
        state: present
    - name: Enable bridge-nf-call tables
      sysctl:
        name: net.bridge.bridge-nf-call-iptables
        value: 1
        state: present
        reload: yes
    - name: Reload sysctl
      ansible.builtin.shell: |
        sysctl --system
    - name: Add Docker repo to yum
      ansible.builtin.get_url:
        url: https://download.docker.com/linux/centos/docker-ce.repo
        dest: /etc/yum.repos.d/docker-ce.repo
    - name: Install docker-ce
      ansible.builtin.yum:
        name: "{{ packages }}"
        state: present
      vars:
        packages:
          - docker-ce
          - docker-ce-cli
          - containerd.io
    - name: Enable docker service
      ansible.builtin.service:
        name: docker
        state: started
        enabled: yes
    - name: Enable containerd service
      ansible.builtin.service:
        name: containerd
        state: started
        enabled: yes
    - name: Generate containerd config.toml and change cgroup
      ansible.builtin.shell: |
        containerd config default | sudo tee /etc/containerd/config.toml
        sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
    - name: Add Kubernetes repo to yum
      ansible.builtin.shell: |
        cat <<EOF > /etc/yum.repos.d/kubernetes.repo
        [kubernetes]
        name=Kubernetes
        baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
        enabled=1
        gpgcheck=1
        repo_gpgcheck=1
        gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
        EOF
    - name: Install kubernetes and bootstrap cluster
      ansible.builtin.yum:
        name: "{{ packages }}"
        state: present
      vars:
        packages:
          - kubeadm
          - kubectl
          - kubelet
    - name: Enable kubelet service
      ansible.builtin.service:
        name: kubelet
        state: started
        enabled: yes
    - name: Restart containerd service
      ansible.builtin.service:
        name: containerd
        state: restarted
        enabled: yes
```

I say 95% of the way because ansible kept not playing nice with `kubeadm init`.  I'm  sure if I tune the playbook I'll get it to work but aint nobody go time for tha shit (right now at least).

So after the playbook runs successfully to completion, final steps would be:

```
#master node
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

```
#worker nodes
sudo kubeadm join 192.168.150.25:6443 --token en2660.qe9xusas64icvt7t --discovery-token-ca-cert-hash sha256:foobarblahblahblah
```

Always trust but verify:
```
NAMESPACE              NAME                                         READY   STATUS    RESTARTS   AGE
default                nginx-ff6774dc6-jp77l                        1/1     Running   0          4h50m
kube-flannel           kube-flannel-ds-8gbbr                        1/1     Running   0          4h55m
kube-flannel           kube-flannel-ds-9n9jv                        1/1     Running   0          4h56m
kube-flannel           kube-flannel-ds-tmzql                        1/1     Running   0          4h54m
kube-system            coredns-565d847f94-rm6c8                     1/1     Running   0          4h59m
kube-system            coredns-565d847f94-tzxkn                     1/1     Running   0          4h59m
kube-system            etcd-control1                                1/1     Running   0          4h59m
kube-system            kube-apiserver-control1                      1/1     Running   0          4h59m
kube-system            kube-controller-manager-control1             1/1     Running   0          4h59m
kube-system            kube-proxy-4rcsq                             1/1     Running   0          4h59m
kube-system            kube-proxy-jndl5                             1/1     Running   0          4h54m
kube-system            kube-proxy-kfxvq                             1/1     Running   0          4h55m
kube-system            kube-scheduler-control1                      1/1     Running   0          4h59m
```

Beautiful!

I even tested an nginx deployment:
```
#kubectl apply -f the contents below
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

```
kubectl get pods | grep nginx
nginx-ff6774dc6-jp77l   1/1     Running   0          4h53m
```

The fat lady has sung!