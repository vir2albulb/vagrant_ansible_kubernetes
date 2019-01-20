# Kubernetes deployment using VirtualBox, Vagrant and Ansible

Playbook is created to deploy Kubernetes master and nodes. It allows to easily
create environment using json-like file.
VirtualBox was used to build VMs as VBox allows usage of concurrent builds
in [Vagrantfile](../master/Vagrantfile).

### Config

##### config.json

All parts of playbook use single config file:

[config.json](../master/files/config.json)

```json
{
  "box": "centos/7",
  "box_version": "1811.02",
  "vm_provider": "virtualbox",
  "instances": [
    {
      "hostname": "master",
      "ip_address": "192.168.33.10",
      "cpus": "2",
      "memory": "2048",
      "groups": [ "k8s-master" ]
    },
    {
      "hostname": "node01",
      "ip_address": "192.168.33.11",
      "cpus": "1",
      "memory": "1024",
      "groups": [ "k8s-node" ]
    }
  ]
}
```

### Requirements

#### Install VirtualBox, Vagrant and Ansible

[VirtualBox](https://www.virtualbox.org/manual/ch02.html)

[Vagrant](https://www.vagrantup.com/intro/getting-started/install.html)

[Ansible](http://docs.ansible.com/ansible/latest/intro_installation.html)

#### Download/clone repository

Download Bitbucket repository or clone it using:

```shell
# using HTTPS
$ git clone https://bitbucket.org/vir2albulb/vagrant_ansible_kubernetes.git
```

### Usage

#### Configfile [config.json](../master/files/config.json)

If you want to use customize config file and supply more Kubernetes masters/nodes.
By default configuration allows to create Kubernetes cluster with one master and
one node.

#### Kubernetes setup [all.yml](../master/provisioning/group_vars/all.yml)

Supported network plugins are:

- calico,
- flannel.

By default playbook deploys the newest Kubernetes cluster. Edit configuration
to provide your values.

```yaml
---
repos:
  docker:
    baseurl: https://download.docker.com/linux/centos/7/$basearch/stable
    gpgkey: https://download.docker.com/linux/centos/gpg
  kubernetes:
    baseurl: https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
    gpgkey: "https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg"

packages:
  docker:
    name: docker-ce
    version: 18.06.1.ce-3.el7
  kubernetes:
    version: ''

network:
  plugin: 'calico'   # calico or flannel
  cidr: 172.31.0.0/16
```

#### Create deployment using vagrant

```shell
$ vagrant up
```

Kubernetes API server is available under 192.168.33.10 by default.

#### Changes in playbooks

If you want to apply some changes in playbooks:

```shell
# Destroy and create new infrastructure
$ vagrant destroy -f
$ vagrant up

## OR ##
# Provision changes using vagrant
$ vagrant provision
```

The reason for usage `vagrant provision` rather then `ansible-playbook` is
because [Vagrantfile](../master/Vagrantfile) passes IP addresses of Kubernetes
master and nodes to playbook using extra arguments.

## License
```
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
