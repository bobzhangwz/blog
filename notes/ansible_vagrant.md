# Ansible and Vagrant

## What vagrant

1. Infrustructure as code
2. Manager vm by code

Example to start ubuntu16 in virtualbox, `Vagrantfile`
```
# coding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  if Vagrant.has_plugin?("vagrant-cachier")
    # Configure cached packages to be shared between instances of the same base box.
    # More info on http://fgrehm.viewdocs.io/vagrant-cachier/usage
    config.cache.scope = :box
  end

  config.ssh.insert_key = false

  cluster = {
    "master" =>  { :ip => "192.168.33.11", :mem => 1024,  :cpu => 1 }
  }

  cluster.each_with_index do | (hostname, info), index |
    config.vm.synced_folder ".", "/vagrant", disabled: false
    config.vm.define hostname do | host |
      host.vm.box = "bento/ubuntu-16.04"
      host.vm.provider "virtualbox" do |v|
        v.memory = info[:mem]
        v.cpus   = info[:cpu]
      end
      host.vm.hostname = hostname
      host.vm.network "private_network", ip: info[:ip]

      config.vm.provision "shell", inline: <<-SHELL
        apt-get update -yyd
        apt-get install python -y
      SHELL
    end
  end

  config.vm.box_check_update = false
end
```

### Command

```bash
vagrant init
vagrant up
vagrant ssh
vagrant halt
vagrant box list
```

## What ansible

Ansible is an IT automation tool. It can configure systems, deploy software, and orchestrate more advanced IT tasks such as continuous deployments or zero downtime rolling updates.

Similar tool: puppet, saltstack, chef

Example, https://github.com/bobzhangwz/dotfiles/blob/master/init.yml:
```
---

- hosts: localhost
  connection: local
  tasks:
    - name: install base packages
      homebrew:
        package:
        - ack
        - ag
        - coreutils
        - dos2unix
        - jq
        - tmux
        - tree

    - name: Download spf13
      get_url:
        url: https://j.mp/spf13-vim3
        dest: /tmp/spf13
        mode: +x
```

### How to use

https://docs.ansible.com/ansible/latest/user_guide/

```
ansible-playbook xxx.yml
ansible-playbook xxx.yml -v
ansible-playbook -i INVENTOR
ansible-playbook --help
```

## Workshop

### Part1 起手式

* 新建目录复制文件
  * https://github.com/bobzhangwz/toolbox/blob/master/Vagrantfile
  * https://github.com/bobzhangwz/toolbox/blob/master/ansible.cfg
  * https://github.com/bobzhangwz/toolbox/blob/master/ansible_inventory
* 创建一个 `task.yml`
```
- name: init docker env
  hosts: Master
  become: true
  gather_facts: yes
  tasks:
    - name: ls dir
      command: ls
```

* 执行命令, `vagrant up`
* `vagrant ssh master`
* `ansible-playbook -i ansible_inventory task.yml`

### Part2 深入

```
vagrant ssh master
apt-get update
apt-get install -y python-mysqldb python-pip
pip install docker-py -y
```

#### As ansible

https://docs.ansible.com/ansible/latest/modules/apt_module.html
```
- name: update apt
  apt:
    update_cache: yes
    cache_valid_time: 3600

- name: install base packages
  apt:
    name: "{{item}}"
  with_items:
    - python-mysqldb
    - python-pip

- command: 'pip install --upgrade pip'

- name: install python module for ansible
  pip:
    name: "{{item}}"
  with_items:
    - docker-py
```

#### Then, extract it to the base role:

```
ansible-galaxy init roles/base
```
Copy tasks to roles/base/tasks/main.yml
```
- name: init docker env
  hosts: Master
  become: true
  gather_facts: yes
  roles:
    - base
```

### Part3. Install docker

Change these command to roles/docker
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04

```
ansible-galaxy init roles/docker
```

`./roles/docker/defaults/main.yml`:
```
---
docker_version: 17.06.1~ce-0~ubuntu
user_of_docker_group: root
```

`./roles/docker/tasks/main.yml`: https://github.com/bobzhangwz/toolbox/blob/master/roles/docker/tasks/main.yml

```
- name: init docker env
  hosts: Master
  become: true
  gather_facts: yes
  vars:
    docker_version: 18.03.1~ce-0~ubuntu
    user_of_docker_group: vagrant
    docker_registry_mirror: https://registry.docker-cn.com
  roles:
    - base
    - docker
```

#### Quick way

https://galaxy.ansible.com/geerlingguy/docker

### Part4. install Jenkins

https://github.com/bobzhangwz/toolbox/tree/master/roles/jenkins

Focus on `templates/main.yml` and `register` syntax

## Pipeline as Code in Jenkins

https://github.com/bobzhangwz/hello-spring/blob/master/Jenkinsfile
