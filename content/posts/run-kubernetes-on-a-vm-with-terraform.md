---
title: Run Kubernetes on a virtual machine
date: 2019-09-10 14:34:00
---
If you need a scratch Kubernetes cluster where you can test deployments and such you can follow this short guide to get a single node Kubernetes.

I'm using WSL with Ubuntu 18.04 so the information here is somehow related to my configuration.

Start by installing Ansible on your machine:

```shell
pip3 install ansible
```

Once that is completed, we'll be using these two roles from Ansible Galaxy: [Docker](https://galaxy.ansible.com/geerlingguy/docker) and [Kubernetes](https://galaxy.ansible.com/geerlingguy/kubernetes).

Install them with the following command:
```shell
ansible-galaxy install geerlingguy.docker geerlingguy.kubernetes
```
This will put those two roles inside `~/.ansible/roles` and you will be able to use them in all of your playbooks.

As for the target host where Kubernetes is going to be deployed, I'm using Ubuntu 18.04 in a Virtualbox VM running thourgh Vagrant. Execute `sudo apt install virtualbox vagrant` to install them both.

To run the VM you can use this `Vagrantfile` and run `vagrant up` in the directory containing the file:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"

  config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.network "public_network"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
  end
end
```

I also like to create my own user on the VM with `sudo adduser USERNAME`, put my SSH public key in `~/.ssh/authorized_keys` and add the user to `/etc/sudoers` with `sudo visudo`.

After you're done with the above, create a simple inventory file for Ansible and a small playbook like this one:

```yaml
---
- hosts: all

  vars:
    kubernetes_allow_pods_on_master: true

  roles:
    - geerlingguy.docker
    - geerlingguy.kubernetes
```

Run the playbook with:

```shell
ansible-playbook -i k8s.ini k8s.yml --become --ask-become-pass -u USERNAME
```

I needed `python` on my system before running the playbook, so it might be useful to run `sudo apt install python` first.

Once that is done you can login on the Kubernetes node and use `kubectl`.

You can also copy the file `/etc/kubernetes/admin.conf` to your dev machine if you want to manage remote clusters with `kubectl`. This [article](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/) can help you create a `kubectl` configuration file with multiple clusters, contexts and users.

***

References:

+ [https://github.com/geerlingguy/ansible-role-kubernetes](https://github.com/geerlingguy/ansible-role-kubernetes)
+ [https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
