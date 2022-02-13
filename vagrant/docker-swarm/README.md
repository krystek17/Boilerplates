# Docker-swarm with Vagrant and Ansible

## Files and Directory Structure

```
├── provisioning
│  ├── bootstrap.yml
│  └── roles
│     ├── base
│     │  └── tasks
│     │     └── main.yml
│     ├── deploy
│     │  └── tasks
│     │     └── main.yml
│     ├── managers
│     │  └── tasks
│     │     └── main.yml
│     ├── swarm-init
│     │  └── tasks
│     │     └── main.yml
│     └── workers
│        └── tasks
│           └── main.yml
└── Vagrantfile

```
## Code Overview
With this Vagrantfile we are creating 4 machines on the same subnet:
```ruby
N = 4

(1..N).each do |id|
Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  config.vm.define "node-#{id}" do |node|
    node.vm.hostname = "node-#{id}"
    node.vm.network "private_network", ip: "192.168.77.#{9+id}"
    node.vm.box = "generic/ubuntu2004"
    node.vm.provider "libvirt" do |v|
      v.memory = "512"
      v.cpus = "1"
      end

    if id == N
      node.vm.provision :ansible do |ansible|
        ansible.limit = "all"
        ansible.playbook = "provisioning/bootstrap.yml"
        ansible.verbose = false
        ansible.compatibility_mode = "2.0"
        ansible.groups = {
            "managers" => ["node-[1:2]"],
            "workers" => ["node-[3:4]"],
          }
        end
      end
    end
  end
end
```
And we provision each of them with the following playbook:
```yaml
---
- name: install docker & fish
  hosts: all
  become: true
  gather_facts: false
  roles:
    - base

- name: initialise swarm
  hosts: node-1
  become: true
  roles:
    - swarm-init

- name: add manager to swarm
  hosts: node-2
  become: true
  gather_facts: false
  roles:
    - managers

- name: add workers to swarm
  hosts: workers
  become: true
  gather_facts: false
  roles:
    - workers

- name: deploy apps
  hosts: node-1
  become: true
  roles:
    - deploy
```


**[`^        back to top        ^`](#)**
