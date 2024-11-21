# Site Software Infrastructure Setup

This repository installs Kubernetes and installs a MySQL database.  It is intended to set up these resources at a new site.

## How to run

### One-time setup

1. Install python at the version specified in .python-version.  You can use whatever python installation method you prefer.  One way to do it is with pyenv:
    1. Install pyenv by following the instructions in https://github.com/pyenv/pyenv#installation for your operating system.
    2. Install python:

            cd ansible
            pyenv install `cat .python-version`
1. Install poetry by following the instructions in https://python-poetry.org/docs/#installing-with-the-official-installer for your operating system.
1. Install dependencies

        poetry install
        poetry run ansible-galaxy install -r requirements.yml
1. Configure your AWS credentials in ~/.aws/credentials.  You can do this using `aws configure` on the command line, or manually editing the file like so:

        [default]
        aws_access_key_id = PASTE_KEY_HERE
        aws_secret_access_key = PASTE_SECRET_HERE
    - Note: If you want to use a profile other than `default`, then you will also need to pass `-e aws_profile=YOUR_AWS_PROFILE` to the `ansible-playbook` command below.
    - Note that these credentials will be copied into a kubernetes secret for use in backups.  You are advised to use credentials that provide limited access.


### Running

1. Have the list of server addresses stored in an inventory file.  The file should have the following format:

        [kube_control_plane]
        hostname-1
        hostname-2

        [etcd]
        hostname-1
        hostname-2
        hostname-3

        [kube_node]
        hostname-1
        hostname-2
        hostname-3

        [k8s_cluster]
        hostname-1
        hostname-2
        hostname-3
1. Run the following command from the Ansible control node:

        poetry run ansible-playbook -i INVENTORY_FILE site.yml -b

### Taking a one-off backup

    # Pick one of the control nodes to set as the value of target_host
    export target_host=PASTE_KUBERNETES_CONTROL_NODE

    scp kubernetes/mysql-one-off-backup.yml "${target_host}:/tmp" && ssh "${target_host}" 'cat /tmp/mysql-one-off-backup.yml | sudo kubectl apply -f -'

## Assumptions

We assume the following:
- The physical computers are already powered on and networked.  They are all on one local network.  They allow ingress on TCP port 22 from the local network, and egress to the public internet.
- The target nodes are running Ubuntu.
- The Ansible control node (that is, the computer running Ansible) has ssh access and passwordless sudo access to the target nodes.
- The Ansible control node computer is on the same local network as the target nodes (for example, an operator laptop at the site, a bastion host located on-site, etc).
- This setup assumes that the input is a list of server addresses for the kubernetes nodes.  All nodes are assumed to be interchangeable.

## Architecture diagram

Here is a high-level diagram of the layout of the system:

![diagram showing a kubernetes cluster in the lower layer and a MySQL cluster in the upper layer](https://raw.githubusercontent.com/ldanz/heirloom_interview_challenge_lisadanz/refs/heads/main/mysql_on_k8s_diagram.png)
