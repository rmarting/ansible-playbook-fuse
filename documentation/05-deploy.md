# Fuse Deploy Role

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
		maven_repository_manager: http://rh7server02:8081/repository/maven-public

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
		rhel7server01
		rhel7server02
		rhel7server03

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
