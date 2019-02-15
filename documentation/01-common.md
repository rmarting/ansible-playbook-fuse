# Common Configuration

## Ansible Requirements

This playbook has tested with the next Ansible versions:

		ansible 2.2.1.0
		ansible 2.3.0.0
		ansible 2.3.1.0
		ansible 2.7.5

The playbook has to be executed with **root permission**, using the **root user** or
via **sudo** because it will install packages and configure the hosts.

## RHEL Subscription Required

This role is prepared to be executed in a RHEL 7 Server. Subscription is required to
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
  			folder: '/tmp/'

* **fuse**: Define the Red Hat JBoss Fuse version and patch to install. This values will
	form the path the the binaries: */tmp/fuse-karaf-{{ fuse['version'] }}.fuse-{{ fuse['version'] }}{{ fuse['patch'] }}.redhat-{{ fuse['maven_version'] }}.zip*

		# Fuse Version and Patch
		fuse:
			version: '7.2.0'
			patch: '035'
			maven_version: '00001'

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
		fuse_base: '/opt/fuse7'

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
		maven_repository_manager: http://rh7server02:8081/repository/maven-public/

		#mutiple maven repository
		maven_repository:
		- 
			url: 'http://rh7server02:8081/repository/maven-releases/'
			id: 'local.release'
			extra: ''
		- url: 'http://rh7server02:8081/repository/maven-snapshots/'
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
