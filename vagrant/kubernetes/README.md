# Kubernetes with Vagrant and Ansible

## Files and Directories Structure
```
.
├── playbook.yml
├── README.md
├── roles
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
