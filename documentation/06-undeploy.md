# Fuse Undeploy Role

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
		maven_repository_manager: http://rh7server02:8081/repository/maven-public

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
		rhel7server01
		rhel7server02
		rhel7server03

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
