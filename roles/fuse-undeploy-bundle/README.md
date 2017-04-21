# Fuse Installation Role

## About

The role has to be executed with root permission, using the root user or via sudo because it will install packages and configure system

This role will install Fuse Standalone Karaf on the servers, for example:

```yml
[fuse-servers]
	10.0.0.10
	10.0.0.11
	10.0.0.12
```

## Ansible Requirements

rhel-7-server-rpms repo have to be available and enabled to install packages

Role has to be executed in a playbook with the gather_facts activated, because hostname, lvm configuration and other parameters will be obtained from the host facts

## Configuration parameters

### Required variables

Variables included in the defaults/main.yaml can be overwritted in the executing playbook.

### Disk Configuration
If no ```disk``` variable is detected, no LVM configuration is created.

### RHEL Subscription Required
This role is prepared to be executed in a RHEL7 Server. Subscription is required to execute the **Grant RHEL and EAP repos enabled** task, which enables the required repositories to install RHSSO dependencies

## Example playbook

```yml
---
- name: Fuse Installation
  hosts: <<host list>>
  remote_user: <<user>>
  gather_facts: true
  become: yes
  become_user: root
  become_method: sudo
  vars:
  roles:
        - ansible-fuseInstaller
```
