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

- Choose your type of cluster in inventory and set it in playbooks/manage_cluster.yml

## Create resources

```# ansible-playbook playbooks/manage_cluster.yml -i auth/hcloud.yml ```

## Create fileserver

```# ansible-playbook playbooks/fileserver.yml -i auth/hcloud.yml ```

## Login into fileserver

``` ssh -i auth/ssh-key root@<fileserver-ip> ```

If you want to modify your install-config.yaml you can do it now.

### Generate ignition config files

```# ./create_ignites.sh ```

## Reboot openshift node in rescue mode

```# ansible-playbook playbooks/restart-rescue.yml -i auth/hcloud.yml ```

## Prepare hosts for Openshift

```# ansible-playbook playbooks/openshift4.yml -i auth/hcloud.yml ```

## Create DNS Record

```api; api-int; *.apps ```

``` api.<cluster_name>.<base_domain> A <lb-ocp4-api IP>  ttl 1min```

``` api-int.<cluster_name>.<base_domain> A <lb-ocp4-api IP> ttl 1min ```

``` *.apps.<cluster_name>.<base_domain> A <lb-ocp4-apps IP> ttl 1min ```

## Reboot hosts and boot on coreOS

```# ansible-playbook playbooks/restart-hosts.yml -i auth/hcloud.yml ```

## Remove bootstrap server when target is down in Hetzner

```# ansible-playbook playbooks/manage_cluster.yml -i auth/hcloud.yml --extra-vars "state_bootstrap=absent" ```

## Login into fileserver

``` ssh -i auth/ssh-key root@<fileserver-ip> ```

### When using workers, do sign the csr
tip: when the workers are not activated, this could be the issue.


``` # export KUBECONFIG=/root/<cluster_name>/auth/kubeconfig ```

``` # oc get csr ```

``` # oc get csr -o name | xargs oc adm certificate approve ```

### Poor guy cluster specific

``` # oc patch etcd cluster -p='{"spec": {"unsupportedConfigOverrides": {"useUnsupportedUnsafeNonHANonProductionUnstableEtcd": true}}}' --type=merge ```

``` # oc patch authentications.operator.openshift.io cluster -p='{"spec": {"unsupportedConfigOverrides": {"useUnsupportedUnsafeNonHANonProductionUnstableOAuthServer": true}}}' --type=merge ```

## Hetzner Cloud Volumes CSI driver
 oc apply -f https://raw.githubusercontent.com/buuhsmead/csi-driver/master/deploy/kubernetes/hcloud-csi-openshift-1.5.1.yml

Based on
 https://raw.githubusercontent.com/slauger/csi-driver/openshift/deploy/kubernetes/hcloud-csi-openshift.yml

The driver provided with hetzner was not able to run under 4.6.3/4.6.4
https://github.com/hetznercloud/csi-driver
