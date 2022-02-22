# Kubernetes with Vagrant and Ansible
## Table of contents

- [Files and Directories Structure](#files-and-directories-structure)
- [Code Overview](#code-overview)
- [Creates a Kubernetes cluster](#create-a-kubernetes-cluster)
  - [Containerd](#container-runtime)
  - [Kubernetes](#kube-packages)
  - [Configure the master](#master)
  - [Configure the nodes](#nodes)

## Files and Directories Structure
```
.
├── playbook.yml
├── roles
│  ├── cni
│  │  └── tasks
│  │     └── main.yml
│  ├── containerd
│  │  ├── defaults
│  │  │  └── main.yml
│  │  ├── handlers
│  │  │  └── main.yml
│  │  └── tasks
│  │     └── main.yml
│  ├── fish
│  │  └── tasks
│  │     └── main.yml
│  ├── kubernetes
│  │  ├── defaults
│  │  │  └── main.yml
│  │  ├── tasks
│  │  │  └── main.yml
│  │  └── templates
│  │     └── 20-extra-args.conf.j2
│  ├── masters
│  │  └── tasks
│  │     └── main.yml
│  └── nodes
│     └── tasks
│        └── main.yml
└── Vagrantfile
```
## Code Overview
Vagrantfile:
```ruby
#!/usr/bin/env ruby

N = 2
NETWORK = "192.168.77"
ANSIBLE_GROUPS = {
  "masters" => ["node-1"],
  "workers" => ["node-[2:#{N}]"],
  "cluster:children" => ["masters", "workers"]
}

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  config.vm.box = "generic/ubuntu2004"
  config.vm.provider "libvirt" do |v|
    v.memory = 2048
    v.cpus = 2
  end


  (1..N).each do |i|
    config.vm.define "node-#{i}" do |node|
      node.vm.network "private_network", ip: "#{NETWORK}.#{i + 9}"
      node.vm.hostname = "node-#{i}"
        
      if i == N
        node.vm.provision "ansible" do |ansible|
          ansible.playbook = "playbook.yml"
          ansible.compatibility_mode = "2.0"
          ansible.groups = ANSIBLE_GROUPS
          ansible.limit = "all"
        end
      end
    end
  end
end
```
The playbook:
```yaml
---
- hosts: cluster
  gather_facts: yes
  strategy: free
  become: yes
  roles:
    - fish
    - containerd
    - kubernetes

- hosts: masters
  gather_facts: yes
  become: yes
  roles:
    - masters
    - cni

- hosts: workers
  gather_facts: yes
  become: yes
  roles:
    - nodes
```
## Create a kubernetes cluster
We will be setting up a Kubernetes cluster that will consist of one master and one worker node. All the vms will run Ubuntu 20.04 and Ansible playbooks will be used for provisioning.


### Container-runtime
### Kube-packages
### Master
### Nodes
