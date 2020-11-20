# hcloud-ocp4

Install ocp4 baremetal cluster on hetzner cloud with Ansible

## Prerequisites

```# pip3 install ansible hcloud ```

- Create ssh-key

```# ssh-keygen -f auth/ssh-key -t ed25519 ```

- Create auth/hcloud.yml

```
plugin: hcloud
keyed_groups:
- key: labels
  prefix: label
token: <your hetzner token>

pull_secret: '<your pull_secret>'
ssh_pub_key: '<your ssh_pub_key>'

base_domain: <your base domain>
cluster_name: <your cluster name>
```


## Create resources

```# ansible-playbook playbooks/manage_cluster.yml -i auth/hcloud.yml ```

## Create fileserver

```# ansible-playbook playbooks/fileserver.yml -i auth/hcloud.yml ```

## Generate ign file from fileserver

```# ./create_ignites.sh ```

## Reboot openshift node in rescue mode

```# ansible-playbook playbooks/restart-rescue.yml -i auth/hcloud.yml ```

## Prepare hosts for Openshift

```# ansible-playbook playbooks/openshift4.yml -i auth/hcloud.yml ```

## Create DNS Record

```api; api-int; *.apps ```

## Reboot hosts and boot on coreOS

```# ansible-playbook playbooks/restart-hosts.yml -i auth/hcloud.yml ```

## Manual remove bootstrap server
Check that the loadbalancer gives green for the masters


## Hetzner Cloud Volumes CSI driver
oc apply -f https://raw.githubusercontent.com/slauger/csi-driver/openshift/deploy/kubernetes/hcloud-csi-openshift.yml

The driver provided with hetzner was not able to run under 4.6.3/4.6.4
https://github.com/hetznercloud/csi-driver



