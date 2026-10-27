# k3s-deploy
Automatically deploy a k3s Kubernetes cluster using Ansible.

## Description

This is an Ansible playbook to deploy a [k3s](https://docs.k3s.io/) cluster with a single control node and multiple worker nodes.

Optionally, it will create a kubeconfig file on your Ansible control node (i.e. where you are running `ansible-playbook` from) to access the cluster.

It requires that you specify two Ansible Inventory groups:
- cluster_control_plane
- cluster

The Playbook will select the _first_ inventory item in the `cluster_control_plane` group and install the k3s server role; it will then enumerate through all members of the `cluster` group, install the k3s agent role on those nodes and join those agents to the aforementioned server.

The playbook also optionally imports a task which will create a kubeconfig file on your Ansible control node (where you are executing the playbook) allowing you to access the cluster using `kubectl` etc.

Optionally, the playbook will now label your worker/data-plane nodes with the node label and or any labels you wish.

## Usage

Install the required Ansible collection:

```bash
ansible-galaxy collection install -r requirements.yml
```

Run the playbook:

```bash
ansible-playbook -K main.yml --inventory=hosts
```

For a pre-deployment check, add `--check`. This validates inventory, the labeling
dependency, and server/agent template rendering using a placeholder join token.
It gathers host facts but skips installation, configuration writes,
service changes, credential reads, kubeconfig import, and labeling. It does not
verify successful installation, API access, or node registration. Hosts must still
be reachable and allow the configured privilege escalation for fact gathering.

## Variables
### `k3s_api_endpoint` (optional hostname or IPv4 address)
Sets the API address used by agents and the exported kubeconfig. Specify only the
hostname or IPv4 address, without `https://` or a port; the playbook uses port 6443.
When omitted, the playbook uses the first control-plane host's `ansible_host`,
falling back to its inventory name. The resolved address is also added to the
server certificate's `tls-san` list.

Set this variable for the whole cluster (for example in `group_vars/all.yml`) so
the server and agents use the same endpoint:

```yaml
k3s_api_endpoint: k3s.example.com
```

The address must resolve and be reachable from both agents and your Ansible
control node. Configuration changes restart existing K3s services on the next run.

### `wait_after_install` (bool: true/false)
This variable will wait for a number of seconds to allow time for cluster node installations to complete and fully spin up.
### `wait_after_install_time_secs` (integer/seconds)
This variable provides the time in seconds that should be used when `wait_after_install` is set to true.
### `import_kube_config` (bool: true/false)
This enables the task to create a local kubeconfig on the Ansible control node that will allow you to access the cluster.
`import_kube_config` is set to false by default as it will not attempt to preserve or merge any existing kubeconfig file.
### `label_nodes` (bool: true/false)
This will invoke the task in `tasks/label_nodes.yml` which will apply Kubernetes labels to your `cluster` inventory group (i.e. the worker/data-plane). By default it applies the "node" label but examples are commented out to show how to apply other labels.
By default this is set to false.
Labeling requires `import_kube_config: true`. The playbook validates this before
deployment and uses the kubeconfig imported by the earlier stage at
`~/.kube/config`, running the local labeling task without privilege escalation.

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
2. ~~My worker nodes are not labelled/tainted with any roles.
That's a Kubernetes issue since you cannot apply those sorts of labels at cluster creation-time; instead you have to do it as a post-install task. I hope to add that soon.~~

# To-do
- [x] Post-install labelling of nodes
- [x] Generate kubeconfig on local Ansible node
