# Fuse Standalone Playbook

## About
This [Ansible Playbook](http://docs.ansible.com/ansible/playbooks.html) includes
a set of different roles:

* **fuse-install**: Deploys a set of Red Hat JBoss Fuse Standalone instances on several hosts.
* **fuse-uninstall**: Uninstall a set of Red Hat JBoss Fuse Standalone instances from several hosts.
* **fuse-deploy-bundle**: Deploys a set of Application Bundles on several hosts.
* **fuse-undeploy-bundle**: Undeploys a set of Application Bundles on several hosts.
* **fuse-patch**: Patch a set of Red Hat JBoss Fuse Standalone instances from several hosts.

These roles with the right configuration will allow you to deploy a full complex
Red Hat JBoss Fuse environment and automate the most common tasks.

[Change Log](./Changelog.md)

## Ansible Requirements

This playbook has tested with the next Ansible versions:

		ansible 2.2.1.0
		ansible 2.3.0.0
		ansible 2.3.1.0

The playbook has to be executed with **root permission**, using the **root user** or
via **sudo** because it will install packages and configure the hosts.

## RHEL Subscription Required

This role is prepared to be executed in a RHEL7 Server. Subscription is required to
execute the **Grant RHEL and Fuse repos enabled** task, which enables the required
repositories to install Fuse dependencies

## Global and Host Variables

### Global Variables

Each playbook will use a set of Global Variables with values that it will be the same for all of them.

These variables will be defined in **groups_vars/all.yaml** file.

These variables are:

* **binary**: to define fuse binary location at your ansible control machine.
		
		# Fuse Binary Setting
		binary:
  			folder: 'Users/yohanesws/RedHat/Installer/'

* **fuse**: Define the Red Hat JBoss Fuse version and patch to install. This values will
	form the path the the binaries: */tmp/jboss-fuse-karaf-{{ fuse['version'] }}.redhat-{{ fuse['patch'] }}.zip*

		# Fuse Version and Patch
		fuse:
			version: '6.3.0'
			patch: '254'
			maven_version: '630262'

* **user**: OS user to execute the Fuse process.

		# OS User to install/execute Fuse
		user:
			name: 'fuse'
			shell: '/bin/bash'
			homedir: 'True'

* **java_home**: Path to Java Virtual Machine to execute the process.

		# Java Home
		java_home: /usr/lib/jvm/jre-1.8.0-openjdk

* **fuse_base**: Fuse base path where it will be installed everything.

		# Fuse Home
		fuse_base: '/opt/fuse'

* **fuse_home**: Fuse Home path where it will installed each instance. This
	variable will be defined for each host with a name. This allow installs more
	instances in the same hosts.

		# Fuse Home
		fuse_home: '{{ fuse_base }}/latest-{{ esb_name }}'

* **fuse_client**: Fuse Client command line to connect to a RHJF instance. It will
 	use administrative credentials.

		fuse_client: '{{ fuse_home }}/bin/client -r 3 -d 10 -u {{ fuse_users.admin.username }} -p {{ fuse_users.admin.password }}'
		
* **maven repo**: Maven repository configuration for Fuse to use it.

		#single maven repository
		maven_repository_manager: http://172.16.1.1:8381/artifactory/libs-snapshot

		#mutiple maven repository
		maven_repository:
		- 
			url: 'http://172.16.1.1:8381/artifactory/libs-release' 
			id: 'local.release'
			extra: ''
		- url: 'http://172.16.1.1:8381/artifactory/libs-snapshot'
			id: 'local.snapshot'
			extra: '@snapshots'

### Host Variables

Each role will use a set of Host Variables defined in the playbook for each
host defined in the inventory.

These variables are:

* **esb_name**: Logical name to identify each Fuse Instance. Mandatory.
* **port_offset**: Port offset to be added to default Fuse ports. Mandatory.

These variables should be defined in the playbook as:

		roles:
			# Two Fuse Standalone with a Network of Brokers
			- { role: fuse-install, esb_name: 'esb01',  port_offset: '0' }
			- { role: fuse-install, esb_name: 'esb02',  port_offset: '100' }

## Ansible Roles

### fuse-install role

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
					- SuperUser

* **nob_multicast_name**: Multicast cluster name to create the Network of Brokers.

		# Network of Brokers Multicast
		nob_multicast_name: default

* **kahadb_directory**: Absolute path for KahaDB store for each broker included
	in an A-MQ Master/Slave topology.

		# KahaDB persistence store
		kahadb_directory: /opt/fuse/kahadb

Host Variables are defined in playbook:

* **nob**: Sets that this instance should be a member of a Network of Brokers. Values: (true|false). Optional.

* **amq_master_slave**: Sets that this instance will be a member of a A-MQ Master/Slave topology. Values: (true|false). Optional.

* **esb_type**: Identify each Fuse type. This variable will define the final
	features needed in this instance. The current values permitted:
	* **full**: Full Fuse instance. Default value.
	* **noamq**: Disable A-MQ features.

#### Example playbook

To execute this playbook:

    ansible-playbook -i hosts fuse-install.yaml

Inventory (*host* file):

		[fuse-lab-environment]
		rhel7jboss01
		rhel7jboss02
		rhel7jboss03

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
				- { role: fuse-install, esb_name: 'esb01',  port_offset: '0', nob: 'true' }
				- { role: fuse-install, esb_name: 'esb02',  port_offset: '100', nob: 'true' }

Other alternatives:

* Two Fuse Standalone without a Network of Brokers:

		roles:
			- { role: fuse-install, esb_name: 'esb01',  port_offset: '0', nob: 'false' }
			- { role: fuse-install, esb_name: 'esb02',  port_offset: '100', nob: 'false' }

* One Fuse Standalone with Full profiles and One Fuse Standalone without A-MQ profile

		roles:
			- { role: fuse-install, esb_name: 'esb01',  port_offset: '0', nob: 'true' }
			- { role: fuse-install, esb_name: 'esb02',  port_offset: '100', esb_type: 'noamq' }

### fuse-uninstall role

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
		rhel7jboss01
		rhel7jboss02
		rhel7jboss03

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

### fuse-deploy-bundle role

This role deploys several Application Bundles into a set of Fuse Standalone instances.

The main tasks done are:

* Download artifacts from a Maven Repository Manager
* Copy artifacts into **{{ fuse_home }}/deploy** folder
* Install Features repositories (URL)
* Install Features from URL list of repositories
* Install OSGi Bundles

#### Configuration parameters

Role's execution could be configured with the following variables.

Global Variables are defined in **group_vars/all.yaml** file.:

* **app_home**: Location to store the applications to be deployed before to do it.

		# Applications Home
		app_home: '/opt/fuse/applications'

* **maven_repository_manager**: Maven Repository location to resolve the artifacts
	to be deployed.

		# Maven Repository
		maven_repository_manager: http://rhel7jboss01:8081/nexus/content/groups/public

* **applications**: List of Maven dependencies to be deployed. The artifacts should
	be located using their GAV coordinates.

		# Application List to deploy
		applications:
			-
				groupId: com.redhat.camel
				artifactId: camel-amq-consumer
				version: 1.1.0-SNAPSHOT
			-
				groupId: com.redhat.camel
				artifactId: camel-amq-producer
				version: 1.1.0-SNAPSHOT
			-
				groupId: com.redhat.camel
				artifactId: camel-amq-forwarder
				version: 1.1.0-SNAPSHOT

* **features**: List of Features to be deployed. The artifacts should
	be located using their GAV coordinates. The *name* attribute defines the
	name of the feature to install from the list url defined by the GAV coordinates

		features:
			-
				groupId: com.redhat.fuse.demo
				artifactId: fuse-demo-features
				version: 1.1.0-SNAPSHOT
				name: fuse-demo-features

* **bundles**: List of OSGi bundles to be deployed. The artifacts should
	be located using their GAV coordinates. The *name* attribute defines the
	name of the bundle to install from the list url defined by the GAV coordinates

		# Bundles List to deploy
		bundles:
			-
		  	groupId: com.redhat.fuse.demo
		  	artifactId: camel-cxfrs
		  	version: 1.1.0-SNAPSHOT
		  -
		  	groupId: com.redhat.fuse.demo
		  	artifactId: camel-cxf
		  	version: 1.1.0-SNAPSHOT

Host Variables are defined in playbook:

* **deploy_features**: Sets if features should be installed in this Fuse Instance. Values: (yes|no). Mandatory.

* **deploy_bundles**: 'Sets if bundles should be installed in this Fuse Instance. Values: (yes|no). Mandatory.

* **deploy_applications**: Sets if applications should be installed in this Fuse Instance. Values: (yes|no). Mandatory.

#### Example playbook

To execute this playbook:

    ansible-playbook -i hosts fuse-deploy-bundle.yaml

Inventory (*host* file):

		[fuse-lab-environment]
		rhel7jboss01
		rhel7jboss02
		rhel7jboss03

Playbook (*fuse-deploy-bundle.yaml* file):

		---
		- name: Fuse Deploy Bundles Playbook
			hosts: fuse-lab-environment
			serial: 2
			remote_user: cloud-user
			gather_facts: true
			become: yes
			become_user: root
			become_method: sudo
			roles:
				# Deploying Bundles in Two Fuse Standalone
				- {
		        role: fuse-deploy-bundle,
		        esb_name: 'esb01',
		        deploy_features: 'yes',
		        deploy_bundles: 'no',
		        deploy_applications: 'yes'
		      }
		    - {
		        role: fuse-deploy-bundle,
		        esb_name: 'esb02',
		        deploy_features: 'yes',
		        deploy_bundles: 'yes',
		        deploy_applications: 'yes'
		      }

### fuse-undeploy-bundle role

This role undeploys several Application Bundles from a set of Fuse Standalone instances.

The main tasks done are:

* Remove artifacts from **{{ fuse_home }}/deploy** folder
* Uninstall Features from URL list of repositories
* Uninstall Features repositories (URL)
* Uninstall OSGi Bundles

#### Configuration parameters

Role's execution could be configured with the following variables.

Global Variables are defined in **group_vars/all.yaml** file.:

* **maven_repository_manager**: Maven Repository location to resolve the artifacts
	to be deployed.

		# Maven Repository
		maven_repository_manager: http://rhel7jboss01:8081/nexus/content/groups/public

* **applications_undeploy**: List of Maven dependencies to be undeployed. The artifacts should
	be located using their GAV coordinates.

		# Application List to undeploy
		applications_undeploy:
			-
				groupId: com.redhat.camel
				artifactId: camel-amq-consumer
				version: 1.1.0-SNAPSHOT
			-
				groupId: com.redhat.camel
				artifactId: camel-amq-producer
				version: 1.1.0-SNAPSHOT
			-
				groupId: com.redhat.camel
				artifactId: camel-amq-forwarder
				version: 1.1.0-SNAPSHOT

* **features_undeploy**: List of Maven dependencies to be undeployed.

		features_undeploy:
			-
				groupId: com.redhat.fuse.demo
				artifactId: fuse-demo-features
				version: 1.1.0-SNAPSHOT
				name: fuse-demo-features

* **bundles_undeploy**: List of OSGi bundles to be undeployed

		# Bundles List to deploy
		bundles:
			-
				groupId: com.redhat.fuse.demo
				artifactId: camel-cxfrs
				version: 1.1.0-SNAPSHOT
		  -
				groupId: com.redhat.fuse.demo
				artifactId: camel-cxf
				version: 1.1.0-SNAPSHOT

There aren't host variables to define in this role.

#### Example playbook

To execute this playbook:

    ansible-playbook -i hosts fuse-undeploy-bundle.yaml

Inventory (*host* file):

		[fuse-lab-environment]
		rhel7jboss01
		rhel7jboss02
		rhel7jboss03

Playbook (*fuse-undeploy-bundle.yaml* file):

		---
		- name: Fuse Undeploy Bundles Playbook
			hosts: fuse-lab-environment
			serial: 2
			remote_user: cloud-user
			gather_facts: true
			become: yes
			become_user: root
			become_method: sudo
			roles:
				# Undeploying Bundles in Two Fuse Standalone
				- { role: fuse-undeploy-bundle, esb_name: 'esb01' }
				- { role: fuse-undeploy-bundle, esb_name: 'esb02' }

### fuse-patch role

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
		  version: '6.3.0'
		  patch: '262'
			maven_version: '630262'

* **fuse_patch**: Define the Red Hat JBoss Fuse version and patch to patch into the
	current Fuse Instance runnning. This values will form the path the
	the binaries: */tmp/jboss-fuse-karaf-{{ fuse_patch['version'] }}.redhat-{{ fuse_patch['patch'] }}.zip*

		# Patch Fuse Version and Patch
		fuse_patch:
		  version: '6.3.0'
		  patch: '262'
			maven_version: '630262'

There aren't host variables to define in this role.

#### Example playbook

To execute this playbook:

    ansible-playbook -i hosts fuse-patch.yaml

Inventory (*host* file):

		[fuse-lab-environment]
		rhel7jboss01
		rhel7jboss02
		rhel7jboss03

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

# Main References

* [Product Documentation for Red Hat JBoss Fuse](https://access.redhat.com/documentation/en/red-hat-jboss-fuse/)
* [Ansible Documentation](http://docs.ansible.com/ansible/)
* [How to remove the embedded ActiveMQ Broker from being deployed in JBoss Fuse?](https://access.redhat.com/solutions/1229533)
* [ActiveMQ - Network of Brokers](http://activemq.apache.org/networks-of-brokers.html)
