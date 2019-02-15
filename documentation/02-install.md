# Fuse Install Role

This role deploys several Fuse Standalone instances in a set of hosts.

The main tasks done are:

* Install Fuse prerequisites and set up host
* Install Fuse binaries
* Create OS users and OS services
* Set up different Fuse features: Karaf, Security, A-MQ, Jetty, ...

Red Hat JBoss Fuse binaries should be downloaded from [Red Hat Customer Portal](https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?downloadType=distributions&product=jboss.fuse&version=6.3.0)
(subscription needed). The binaries should be copied into **/tmp** folder from
the host where the playbook will be executed.

#### Configuration parameters

Role's execution could be configured with the following variables.

Global Variables are defined in **group_vars/all.yaml** file.:

* **fuse_users**: Users map to be created in each Fuse instance. These users will
	be stored in **{{ fuse_home }}/etc/users.properties** file.

		# Fuse Administrative Users
		fuse_users:
			admin:
				username: admin
				password: karaf
				roles:
					- admin
					- group

Host Variables are defined in playbook:

* **esb_type**: Identify each Fuse type. This variable will define the final
	features needed in this instance. The current values permitted:
	* **full**: Full Fuse instance. Default value.

#### Example playbook

To execute this playbook:

    ansible-playbook -i hosts fuse-install.yaml

Inventory (*host* file):

		[fuse-lab-environment]
		rhel7server01
		rhel7server02
		rhel7server03

Playbook (*fuse-install.yaml* file):

		---
		- name: Fuse Standalone Playbook
			hosts: fuse-lab-environment
			serial: 2
			remote_user: cloud-user
			gather_facts: true
			become: yes
			become_user: root
			become_method: sudo
			roles:
				# Two Fuse Standalone with a Network of Brokers
				- { role: fuse-install, esb_name: 'esb01',  port_offset: '0' }
				- { role: fuse-install, esb_name: 'esb02',  port_offset: '100' }
