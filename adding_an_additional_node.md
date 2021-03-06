# Adding An Additional Node

In this lab you will learn how to increase your cluster by adding an additional node. Adding an additional node can be done without taking the cluster offline and it is non invasive.

## Step 1

The first thing you need to do is prep the host by following the [host prep guide](https://docs.openshift.com/container-platform/latest/install_config/install/host_preparation.html)

Things you need to do can be (but not limited to)

* Host Registration
* Base RPM installation
* Docker installation/configuration
* SSH Keys configuration

Once you have preped the host; you should test connection with the following command (assuming you named your host `node3.example.com`

```
ansible all -s --limit node3.example.com -m shell -a "hostname"
node3.example.com | SUCCESS | rc=0 >>
node3.example.com
```

## Step 2

You can add new hosts to your cluster by running the `scaleup.yml` playbook. This playbook queries the master, generates and distributes new certificates for the new hosts, then runs the configuration playbooks on the new hosts only. 

Start by making sure the latest playbooks are on the master

```
yum update atomic-openshift-utils -y
```

Edit your `/etc/ansible/hosts` file and add `new_nodes` to the `[OSEv3:children]` section:

For example:

```
[OSEv3:children]
masters
nodes
new_nodes
```

Next, create this `[new_nodes]` section much like an existing section, specifying host information for any new hosts you want to add. The bottom of your ansible host file should look like this.

```
[nodes]
master.example.com openshift_node_labels="{'region': 'infra', 'zone': 'default'}"
node1.example.com openshift_node_labels="{'region': 'primary', 'zone': 'z1'}"
node2.example.com  openshift_node_labels="{'region': 'primary', 'zone': 'z2'}"

[new_nodes]
node3.example.com  openshift_node_labels="{'region': 'primary', 'zone': 'z3'}"
```

## Step 3

You are now ready to run the `scaleup.yml` playbook. Specify a `-i /path/to/hostfile` if you need to. If your hosts file is in the default location (`/etc/ansible/hosts`) then you do not need to specify.

```
ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/byo/openshift-node/scaleup.yml
```

After this has finished; verify the installation

```
oc get nodes
NAME                        STATUS    AGE
master.example.com          Ready     47d
node1.example.com           Ready     47d
node3.example.com           Ready     47d
```

Finally, move any hosts you had defined in the `[new_nodes]` section into their appropriate section (but leave it in place) so that subsequent runs using this inventory file are aware of the nodes but do not handle them as new nodes. For example; change the file to look like this.

```
[nodes]
master.example.com openshift_node_labels="{'region': 'infra', 'zone': 'default'}"
node1.example.com openshift_node_labels="{'region': 'primary', 'zone': 'z1'}"
node2.example.com  openshift_node_labels="{'region': 'primary', 'zone': 'z2'}"
node3.example.com  openshift_node_labels="{'region': 'primary', 'zone': 'z3'}"

[new_nodes]
```

## Conclusion

In this lab you learned how to scale up your cluster to include an additional node.
