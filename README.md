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

### Issue: Error: Error initializing source` in Ansible Automation Platform 2.1

```
$ ansible-navigator
--------------------------------------------------------------------------------------------------------------
Execution environment image and pull policy overview
--------------------------------------------------------------------------------------------------------------
Execution environment image name:  registry.redhat.io/ansible-automation-platform-21/ee-supported-rhel8:latest
Execution environment image tag:   latest
Execution environment pull policy: tag
Execution environment pull needed: True
--------------------------------------------------------------------------------------------------------------
Updating the execution environment
--------------------------------------------------------------------------------------------------------------
Trying to pull registry.redhat.io/ansible-automation-platform-21/ee-supported-rhel8:latest...
Error: initializing source docker://registry.redhat.io/ansible-automation-platform-21/ee-supported-rhel8:latest: unable to retrieve auth token: invalid username/password: unauthorized: Please login to the Red Hat Registry using your Customer Portal credentials. Further instructions can be found here: https://access.redhat.com/RegistryAuthentication
[ERROR]: Execution environment pull failed
 [HINT]: Check the execution environment image name, connectivity to and permissions for the registry, and try again
```

```
$ podman login registry.redhat.io
Username: <access.redhat.com username>
Password: 
Login Succeeded!
```

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
$ cat aansible-playbook -i lab_inventory/hosts ansible-lab-examples/rhel/roles/subscription/subscription_manager.yml
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

- 確認 RHEL Host 尚未註冊授權
```
[root@atest2 ~]# subscription-manager status
+-------------------------------------------+
   System Status Details
+-------------------------------------------+
Overall Status: Unknown

System Purpose Status: Unknown

[root@atest2 ~]# subscription-manager version
server type: This system is currently not registered.
subscription management server: 3.2.21-1
subscription management rules: 5.41
subscription-manager: 1.28.21-3.el8
```

- Execution

```sh
$ aansible-playbook -i lab_inventory/hosts ansible-lab-examples/rhel/roles/subscription/subscription_manager.yml

PLAY [subscription-manager module demo] ******************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************
ok: [atest2]

TASK [register with subscription-manager] ****************************************************************************************************************************************

changed: [atest2]

PLAY RECAP ***********************************************************************************************************************************************************************
atest2                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

- Check Subscription

```
[root@atest2 ~]# subscription-manager version
server type: Red Hat Subscription Management
subscription management server: 3.2.21-1
subscription management rules: 5.41
subscription-manager: 1.28.21-3.el8
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

- main.yml
```
$ cat ansible-lab-examples/rhel/roles/apache_vhost/tasks/main.yml

---
# tasks file for roles/apache_vhost
- name: Apache server installed
  hosts: atest2
  become: yes
  tasks:
  - name: latest Apache version installed
    yum:
      name: httpd
      state: latest
  - name: Apache enabled and running
    service:
      name: httpd
      enabled: true
      state: started
  - name: copy web.html
    copy:
      src: web.html
      dest: /var/www/html/index.html
```

- 執行

```
$ ansible-navigator run ansible-lab-examples/rhel/roles/apache_vhost/tasks/main.yml -i ~/lab_inventory/hosts -m stdout

PLAY [Apache server installed] *************************************************

TASK [Gathering Facts] *********************************************************
ok: [atest2]

TASK [latest Apache version installed] *****************************************
changed: [atest2]

TASK [Apache enabled and running] **********************************************
changed: [atest2]

TASK [copy web.html] ***********************************************************
changed: [atest2]

PLAY RECAP *********************************************************************
atest2                     : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

```
$ ansible-navigator run check_httpd_status.yml -i ~/lab_inventory/hosts-m stdout
$ ansible-navigator run check_httpd_status.yml -i ~/lab_inventory/hosts -m stdout
$ ansible-navigator run check_package.yml -i ~/lab_inventory/hosts -m stdout
```

#### Reference
* [Workshop Exercise - Writing Your First Playbook - Build  Apache web server](https://aap2.demoredhat.com/exercises/ansible_rhel/1.3-playbook/)
