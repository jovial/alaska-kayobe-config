====================
Kayobe Configuration
====================

Kayobe configuration for the ALaSKA/P-cubed system. There are two partitions in
the system - *production*, and *alt-1*. The *alt-1* partition is used as a
development and test system. There was previously an *alt-2* partition, but
this is no longer in service.

Overview
========

This repository provides configuration for the `kayobe
<https://opendev.org/x/kayobe>`_ project. It is intended to encourage
version control of site configuration.

Kayobe enables deployment of containerised OpenStack to bare metal.

Containers offer a compelling solution for isolating OpenStack services, but
running the control plane on an orchestrator such as Kubernetes or Docker
Swarm adds significant complexity and operational overheads.

The hosts in an OpenStack control plane must somehow be provisioned, but
deploying a secondary OpenStack cloud to do this seems like overkill.

Kayobe stands on the shoulders of giants:

* OpenStack bifrost discovers and provisions the cloud
* OpenStack kolla builds container images for OpenStack services
* OpenStack kolla-ansible delivers painless deployment and upgrade of
  containerised OpenStack services

To this solid base, kayobe adds:

* Configuration of cloud host OS & flexible networking
* Management of physical network devices
* A friendly openstack-like CLI

All this and more, automated from top to bottom using Ansible.

* Documentation: https://kayobe.readthedocs.io/en/latest/
* Source: https://opendev.org/x/kayobe
* Bugs: https://storyboard.openstack.org/#!/project/openstack/kayobe-config
* IRC: #openstack-kayobe
