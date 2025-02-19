# demo-k3s-cluster
A demonstration k3s cluster with a full *arr suite.

## Prerequisites
**Note:** All steps in this guide are performed on Ubuntu 22.04. User experience may differ.

* Install pip (Assuming python3 is already installed): `sudo apt-get install python3-pip`
* Install the most recent version of Ansible from pip: `pip3 install --user ansible`
* Export the local bin path: `export PATH=~/.local/bin:$PATH`
* Install the required Ansible dependencies using Ansible Galaxy (`ansible-galaxy install -r requirements.yml`)
* 1 or more Raspberry Pis with Raspbian or some other Debian-based OS installed

Once you have the IP of your nodes, update the [hosts](./inventories/k3s/hosts.yml) file with the IP addresses of your nodes accordingly.

## Initialize the Cluster
To create the master node for the cluster, run the following command:

```shell
ansible-playbook -i inventories/k3s --user $USER --ask-pass --ask-vault-pass --tags k3s-cluster setup-k3s.yml
```

This will prompt you for
* Your SSH password for the master node
* A vault password (any password you like to encrypt the secrets in the vault)

You need the vault password because once the master node is created, the cluster token will be fetched from the master node
and encrypted and placed inside the [group_vars](./inventories/k3s/group_vars/all.yml) file. This will allow Ansible
to then automate the creation and configuration of the worker nodes.

**Note:** Ansible detects whether or not to initialize a master node solely based on the presence of the `k3s_cluster_token` in your
group vars. If you need to run the command again to create a new master node, you must first remove the token from the group vars.

This command will do the following:
* Install kubectl and helm locally so you can control your cluster
* Install docker on the pi
* Install kubectl on the pi
* Edit your boot options to enable virtualization and enable the required features
* Disable swap memory
* Reboot
* Setup the K3s master node
* Pull down the config to your local machine and make it so you can directly control the cluster via kubectl
* It places this config in `/home/<USER>/.kube/config`
* It will install NGINX as the ingress controller and LB
* Encrypt the cluster token and add it to your ansible group_vars so you can later create children nodes
* Install the Kubernetes dashboard so you have a nice UI to use

## Adding a Child Node to the Cluster
To add a child node to the cluster, simply run the same command again once you've created the master node:

```shell
ansible-playbook -i inventories/k3s --user $USER --ask-pass --ask-vault-pass --tags k3s-cluster setup-k3s.yml
```

## Deploy Some Services
This repo comes prepared with some services that can deploy to your cluster to start playing with K3s. These services are all the traditional [Servarrs](https://wiki.servarr.com/).
To deploy all the included services, run the following command:

```shell
ansible-playbook -i inventories/k3s --user $USER --ask-pass --tags apps setup-k3s.yml
```

You can control which services are deployed using the following tags:

* bazarr
* jellyfin
* lidarr
* nzbget
* overseerr
* plex
* prowlarr
* radarr
* readarr
* sonarr
* tautulli
* transmission

**Example:** To deploy Plex and Overseerr, you'd use the following command:

```shell
ansible-playbook -i inventories/k3s --user $USER --ask-pass --tags plex,overseerr setup-k3s.yml
```

All the resource definitions are created via [kompose](https://kompose.io/) based on the included `docker-compose.yml` files. They are created simply with `kompose convert`.

## Additional Features to Deploy (Optional)
If you look at the [cluster configuration role](./roles/k3s/tasks/main.yml), you'll see that I commented out a few things like setting up an nfs persistent volume provisioner and the Longhorn block storage system. You can try those out if you like but there's no guarantee that those will work because it's been a hot minute since I've run the Longhorn setup specifically. The NFS one should work just fine though.

## Additional Notes
* Some images are only available on ARM64 nodes, and thus some services, such as the [Plex](./roles/plex/files/plex-deployment.yml) service use node
  affinity to ensure they're only deployed on ARM64 nodes. This is done to avoid deployment errors when the image isn't available for a specific node's platform.

## Creator
* [Alex Clarke](https://github.com/Dark-Alex-17)
