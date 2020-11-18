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
```


## Create resources

```# ansible-playbook playbooks/manage-resources.yml ```

## Create fileserver

```# ansible-playbook playbooks/fileserver.yml -i auth/hcloud.yml ```

## Generate ign file from fileserver

```# ./create_ignites.sh ```

## Reboot openshift node in rescue mode

```# ansible-playbook playbooks/restart-rescue.yml ```

## Prepare hosts for Openshift

```# ansible-playbook playbooks/openshift4.yml -i hcloud```

## Create DNS Record

```api; api-int; *.apps ```

## Reboot hosts and boot on coreOS

```# ansible-playbook playbooks/restart-hosts.yml ```
