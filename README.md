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

## Red Hat Fuse Versions

The master branch is always updated to the lates version of Red Hat Fuse. The following branches
are developed for specific versions of Red Hat Fuse:

* [Red Hat Fuse 7.2](https://github.com/rmarting/ansible-playbook-fuse/tree/fuse-7.2)
* [Red Hat JBoss Fuse 6.3](https://github.com/rmarting/ansible-playbook-fuse/tree/fuse-6.3)

## Documentation

Each role is described in its own documentation page as:

* [Common Configuration](./documentation/01-common.md)
* [Install](./documentation/02-install.md)
* [Uninstall](./documentation/03-uninstall.md)
* [Patch](./documentation/04-patch.md)
* [Deploy Applications](./documentation/05-deploy.md)
* [Undeploy Applications](./documentation/06-undeploy.md)

# Main References

* [Product Documentation for Red Hat Fuse 7.2](https://access.redhat.com/documentation/en-us/red_hat_fuse/7.2/)
* [Fuse Maintenance Schedule](https://access.redhat.com/articles/2939351#header70Top)
* [Ansible Documentation](http://docs.ansible.com/ansible/)
* [How to create an offline Maven repository for Fuse 7?](https://access.redhat.com/solutions/3746581)
