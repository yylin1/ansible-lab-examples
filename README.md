# Ansible Automation Platform - Practice 

This repository contains demo playbooks and roles used in our demo lab.

Please note that this content is meant for educational purposes and might contain errors or challenges.


----
![Red Hat Ansible Automation](images/logo-rh-ansible-automation.png)

## Ansible PlayBook Practice

Tasks list:
- [x] Register RHEL to Red Hat CDN
- [x] Execute via your own user with sudo
- [x] Install httpd
  - [x] Config index.html with variable
    - For example
      - User said: {{ words }}
- [ ] Log every content change date time when index.html had been modified
    - Log file: /tmp/achange.log
    - Owner: root:root
    - Permission: 0600
- [ ] Start httpd service
- [ ] Mail the result to your Red Hat mailbox

## Quick Start
In this section you can find the folloing step finish all practice tasks.

---

### Register RHEL to Red Hat CDN

Manage registration and subscriptions to RHSM using the `subscription-manager` command.

You might already have this collection installed if you are using the ansible package. It is not included in ansible-core. To check whether it is installed, run ansible-galaxy collection list.

- To install it, use: `ansible-galaxy collection install community.general`.

```sh
$ ansible-galaxy collection install community.general

Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/community-general-4.2.0.tar.gz to /home/student1/.ansible/tmp/ansible-local-133680hphzl_5c/tmpjxf25544/community-general-4.2.0-mhk5de53
Installing 'community.general:4.2.0' to '/home/student1/.ansible/collections/ansible_collections/community/general'
community.general:4.2.0 was installed successfully
```

- subscription-manager.yml

```sh
$ cat ansible-lab-examples/subscription/subscription-manager.yml
---
- name: subscription-manager module demo
  hosts: all
  become: true
  vars:
    subscription_username: "username"
    subscription_password: "password"
  tasks:
    - name: register with subscription-manager
      community.general.redhat_subscription:
        state: present
        username: "{{ subscription_username }}"
        password: "{{ subscription_password }}"
        auto_attach: true
```
- execution

```sh
$ ansible-navigator run -i subscription/subscription_manager.yml
```

#### Reference
* [Register a system with Red Hat Subscription-Manager — Ansible module redhat_subscription](https://ansiblepilot.medium.com/register-a-system-with-red-hat-subscription-manager-ansible-module-redhat-subscription-7134e77f8d26).
* [Ansible Documentation - Manage registration and subscriptions to RHSM using the subscription-manager command](https://docs.ansible.com/ansible/latest/collections/community/general/redhat_subscription_module.html).

---

### Execute via your own user with sudo

sudo is to execute a task as a root user, in other words, we call it as privileged execution. If you want to execute a certain task as the root user using ansible just become is sufficient


![image](https://user-images.githubusercontent.com/39441918/148001602-91c6d805-6976-44db-b884-6489fd6336b1.png)

#### Reference
* [Ansible sudo – ansible become example](https://www.middlewareinventory.com/blog/ansible-sudo-ansible-become-example/)

---

### Install httpd and Config index.html with variable


#### Reference
* [Workshop Exercise - Writing Your First Playbook - Build  Apache web server](https://aap2.demoredhat.com/exercises/ansible_rhel/1.3-playbook/)
