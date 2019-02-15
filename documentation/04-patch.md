# Fuse Patch Role

This role patches several Fuse Standalone instances in a set of hosts.

The main tasks done are:

* Backup Fuse Standalone instances
* Install Patch Fuse binaries
* Simulate and patch Fuse instances with *patch* commands

This process is documented in [Patching a Standalone Container](https://access.redhat.com/documentation/en-us/red_hat_jboss_fuse/6.3/html-single/configuring_and_running_jboss_fuse/#ESBRuntimePatchStandalone)
section of the Red Hat JBoss Fuse Documentation

#### Configuration parameters

Role's execution could be configured with the following variables.

Global Variables are defined in **group_vars/all.yaml** file.:

* **fuse**: Define the Red Hat JBoss Fuse version and patch to install. This values will
	form the path the the binaries: */tmp/jboss-fuse-karaf-{{ fuse['version'] }}.redhat-{{ fuse['patch'] }}.zip*

		# Fuse Version and Patch
		fuse:
		  version: '7.2.0'
		  patch: '035'
		  full_version: '720035'
		  maven_version: '00001'

* **fuse_patch**: Define the Red Hat JBoss Fuse version and patch to patch into the
	current Fuse Instance runnning. This values will form the path the
	the binaries: */tmp/jboss-fuse-karaf-{{ fuse_patch['version'] }}.redhat-{{ fuse_patch['patch'] }}.zip*

		# Patch Fuse Version and Patch
		fuse_patch:
			version: '7.2.0'
			patch: '035'
			full_version: '720035'
			maven_version: '00001'

There aren't host variables to define in this role.

#### Example playbook

To execute this playbook:

    ansible-playbook -i hosts fuse-patch.yaml

Inventory (*host* file):

		[fuse-lab-environment]
		rhel7server01
		rhel7server02
		rhel7server03

Playbook (*fuse-patch.yaml* file):

		---
		- name: Patch Playbook of a Fuse Standalone Environment
			hosts: fuse-lab-environment
			serial: 2
			remote_user: cloud-user
			gather_facts: true
			become: yes
			become_user: root
			become_method: sudo
			roles:
				- { role: fuse-patch, esb_name: 'esb01', port_offset: '0' }
				- { role: fuse-patch, esb_name: 'esb02', port_offset: '100' }
