# Site Software Infrastructure Setup

This repository installs Kubernetes and installs a MySQL database.  It is intended to set up these resources at a new site.

## Terminology

- "Ansible control node" is the computer (such as a laptop or a Jenkins worker) that is running Ansible.

## How to run

### One-time setup

1. Install python
1. Install poetry
1. Install dependencies

    poetry install
    poetry run ansible-galaxy install -r requirements.yml


### Running

1. Have the list of server addresses (could be IP addresses or hostnames, as long as they're reachable from the Ansible control node) stored in a file.
1. Run the following command from the Ansible control node:

        poetry run ansible-playbook -i INVENTORY_FILE site.yml -b

### Assumptions

We assume the following:
- The physical computers are already powered on and networked.  They are all on one local network.  They allow ingress on TCP port 22 from the local network, and egress to the public internet.
- The target nodes are running Ubuntu.
- The Ansible control node computer is on the same local network as the target nodes (for example, an operator laptop at the site, a bastion host located on-site, etc).
- This setup assumes that the input is a list of server addresses for the kubernetes nodes.  All nodes are assumed to be interchangeable.
