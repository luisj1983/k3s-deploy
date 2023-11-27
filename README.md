# k3s-deploy
Automatically deploy a k3s Kubernetes cluster using Ansible.

## Description

This is an Ansible playbook to deploy a [k3s](https://docs.k3s.io/) cluster with a single control node and multiple worker nodes.

It requires that you specify two Ansible Inventory groups:
- cluster_control_plane
- cluster

The Playbook will select the _first_ inventory item in the `cluster_control_plane` group and install the k3s server role; it will then enumerate through all members of the `cluster` group, install the k3s agent role on those nodes and join those agents to the aforementioned server.

The playbook also optionally imports a task which will create a kubeconfig file on your Ansible control node (where you are executing the playbook) allowing you to access the cluster using `kubectl` etc.

## Usage

```bash
ansible-playbook -K main.yml
```

## Variables
### `wait_after_install` (bool: true/false)
This variable will wait for a number of seconds to allow time for cluster node installations to complete and fully spin up.
### `wait_after_install_time_secs` (integer/seconds)
This variable provides the time in seconds that should be used when `wait_after_install` is set to true.


## FAQ/Issues
1. I don't want to use the provided Ansible Inventory group names.<br>

It should be easy enough to do a find and replace on those group names to use your own. Alternatively, use the Ansible [child group relationship](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#grouping-groups-parent-child-group-relationships). <br>For example:

```yaml
cluster_control_plane:
  children:
    your_preferred_group_here:
cluster:
  children:
    your_other_preferred_group_here:
```
2. My worker nodes are not labelled/tainted with any roles.
That's a Kubernetes issues since you cannot apply those sorts of labels at cluster creatiion-time; instead you have to do it as a post-install task. I hope to add that soon.

# To-do
- [ ] Post-install labelling of nodes
- [ ] Generate kubeconfig on local Ansible node