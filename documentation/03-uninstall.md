# Fuse Uninstall Role

This role uninstall several Fuse Standalone instances from inventory host.

The main tasks done are:

* Stop Fuse services
* Remove OS services
* Remove Fuse binaries

#### Configuration parameters

Role's execution could be configured with the following variables.

This role uses the global variables and host variables defined for
**fuse-install** role.

#### Example playbook

To execute this playbook:

		ansible-playbook -i hosts fuse-uninstall.yaml

Inventory (*host* file):

		[fuse-lab-environment]
		rhel7server01
		rhel7server02
		rhel7server03

Playbook (*fuse-undeploy-bundle.yaml* file):

		---
		- name: Uninstall Playbook of a Fuse Standalone Environment
		  hosts: fuse-lab-environment
		  remote_user: cloud-user
		  gather_facts: true
		  become: yes
		  become_user: root
		  become_method: sudo
		  roles:
		    - { role: fuse-uninstall, esb_name: 'esb01' }
		    - { role: fuse-uninstall, esb_name: 'esb02' }
