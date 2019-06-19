# docker-swarm

This playbook, sets up a docker swarm cluster. I usually test a small cluster configuration using docker-machine using virtualbox driver. `docker-machine` does not need docker enginer to be installed and also it uses tinycore linux using boot2docker.iso.

## Prerequisites 
Before running this playbook ensure that all the master and worker nodes ready. If you are using any of CentOS/RedHat/Debian/Ubuntu, the playbook will install docker-ce on the nodes.

### Steps to be followed to prepare infrastructure to be used in case of docker-machine
If you are using `docker-machine` for testing, then the installation part will be ignored as docker engine is pre-installed. However, since the tinycore linux as a part of `boot2docker.iso`, which `docker-machine` uses, does not have python, you would need to install python on the nodes.

#### Create Machines 
Create the master and worker nodes using virtualbox driver

```bash
$ docker-machine create --driver virtualbox master1
$ docker-machine create --driver virtualbox worker1
$ docker-machine create --driver virtualbox worker2
```

#### Install Python on the nodes
To install python use the tiny core linux package manager which is present in the nodes - 

```bash
$ for node in $(docker-machine ls -q); do docker-machine ssh $node "tce-load -wi python"; done
```

#### Update hosts file and inventory
If you need to access the docker nodes using hostname, then populate the hosts file on each node (or use DNS etc). 
Populate the inventory file with the Swarm masters and workers

## Variables Used
The default file in `defaults/main.yml` uses the following variables in the playbook. I prefer to use the specified network/ip for master advertisement because [ansible_default_ipv4 is not what you think](https://medium.com/opsops/ansible-default-ipv4-is-not-what-you-think-edb8ab154b10).

* **master_subnet**: This is the subnet on which master nodes of the swarm cluster resides. The code will pick the first of ip addresses on the first master found in the inventory.

## Playbook run

The following are some of the ways this entire playbook is intended to run:

### Full Run

This sets up the docker enginer, creates a swarm cluster if needed, add master and workers.

```bash
$ ansible-playbook -i inventory deploy-swarm.yml
```

## Limitations
Unfortunately this is a quick fix playbook and hence limited to only **Single Master** setup and we need to do a full run every time we need to add a worker node. 

I have added ignore_errors at various places to take care of the fact that if a node is already a worker, then it won't fail the entire playbook.

It is definitely possible to write a comprihensive playbook to address these scenerios, but perhaps at a later date. And I intend to modify this playbook to use docker_swarm ansible module, rather than shell commands.