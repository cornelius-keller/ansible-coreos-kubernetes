# Ansible playbooks to install CoreOs and Kubernetes on baremetal servers

This repository contains ansible playbooks to bootstrap and update a kubernetes cluster on bare metal servers on low budget bare metal servers provided by hetzner, kimsufi or ovh.

# Disclaimer
CoreOs, Kubernetes, Etcd, Ceph etc. are highly complex distributed systems. Use at your own risk and be sure you know what you are doing. I don't take any garantee on your data, availability etc.

# State of this documentation:
The documentation is very basic for now. Open sourcing this is a spare time project and so also my time to work on this and updating the documentation is limeted. So please, if something is not working on the first try as you expected, take some time to dig into the playbooks etc. before opening an issue. I'll be happy to merge pull requests also if they are only improving the documentation.

## Features

* Works with every Server provider that provides an api to boot the server in a rescuse linux system over the network. For now hetzner and kimsufi (ovh) are supported.
* Installes coreos on all cluster nodes
* Sets up an etcd cluster on all hosts in the etcd-node group (should be an uneaven number)
* Sets up etcd in proxy monde on the hosts in the etcd-proxy group (should be all other nodes)
* Sets up a VPN between the nodes using flannel and tinc
* Installes kubernetes api server etc  in a ha setup on all nodes in the kubeernetes-master group
* sets up the hosts in the group kubernetes_nodes as kubernetes nodes
* installs the kubeernetes dns addon
* installs the kubectl for you in ~/bin
* configures the ~/.kube/config file for you for cluster access
* Sets up a ceph cluster on top of kubernetes (highly experimental) for block devices.
* Provides a way to update the cloudconfig of your clusrter and reboot wit2hout destroying your ceph cluster.
* Coreos auto updates are applied in a way that the ceph cluster stays alive.
* Supports multiple clusters

## Missing features (ToDo)
* configure centralized logging / kibana for the cluster automatically. For now have a look into the cluster addons folder in the kubernetes project for setting this up.
* add support for glusterfs to have cluseterwide shared voumes
* Add support for configuration switches, f.e with / withoud ceph support.

## Getting started.
### Hetzner
#### Preparations:
* Order machines. Three is a good start. For testing puposes I used cheap ones from the "Serverbidding".
* create an inventory by copying the inventory-hetzner.ini.sample
* In the sample inventory the hosts are grouped in the following way:
    * All three nodes are etcd-nodes
    * There are no etcd proxy nodes. If you want to test etcd proxy nodes with three nodes you can use one host in the etcd node group and two hosts in the etcd-proxy group. But be aware that your cluster is down if the single etcd node is down.
    * Two are kube-apiserver / master nodes
    * One node is a kubernetes node
    * all nodes are ceph monitor nodes (should stay three)
    * all nodes are ceph osds ( can be more )
* If your cluster and your load grows you probably want to have the etcd cluster and the kubernetes master nodes on dedicated hosts.
* upload your private key into the hetzner admin interface and put the fingerprint of the key into the according variable inventory.
* Put your credentials for the hetzner webservices into the according variables in the inventory.
#### Bootstrap cluster

Run:

    ansible-playbook -i inventory-hetzner.ini bootstrap_coreos.yml

After about 10-15 min your cluster is ready and you can access it with kubectl.
#### Update cluster
If you want to update your cluster for example to rollout a new kubernetes version or you want to update the cloud configuration for some other reasons run:

    ansible-playbook -i inventory-hetzner.ini update_cloudconfig.yml


### Directories
The cluster bootstrap creates the following Directories prefixed with your cluster name ( you can set the cluster name in the inventory ). This directories contain important configuration information like generated certificates and certification authorites for etcd and kubernetes and the ceph configuration files.

* `<cluster-name>-etcd-ca-keys`: Contains the CA, the certificates and the pulbic / private keys used to secure etcd. Don't delete unless you dispose the cluster.
* `<cluster-name>-kubernetes-ca`: Contains the CA, the certificates and the pulbic / private keys used to secure kubernets and access the cluster. Don't delete unless you dispose the cluster.
* `<cluster-name>-ceph`: contains the ceph configuration. Can be regenerated from the inventory.



# Known issues
* On kimsufi / ovh you are not able to set your servers into the rescue mode after you installed coreos on them. ( I opend some tickets regarding this and had a long conversation in the tickets about this, but so far nothing changed ) So if there went something wrong and you want to restart from scratch Do the following steps:
    * Login to the kimsufi console
    * Select Reinstall server from the menu and install an arbitray os on the server.
    * Wait for a lot of failure emails and a technican to fix this
    * Once you get an email from the support that your server is reinstalled with the Os you selected you are ready to go again.

# Credits

* The genaral coreos setup is based on https://coreos.com/kubernetes/docs/latest/getting-started.html
* The genearal idea on how to run ceph on top of kubernetes is inspired by https://github.com/AcalephStorage/ceph-docker/tree/kubernetes/examples
* Many thanks to kayrus in the coreos irc channel for patiently answering all my questions.
