

# Adding and removing nodes to an existing CoreOS etcd2 cluster using etcdctl**

 

Now that our [3-node etcd2 cluster is up and running in Amazon EC2 using CoreOS](https://www.jorgeacetozi.com.br/single-post/setting-up-an-etcd-cluster-in-aws-using-coreos) it’s time to learn how to add or remove nodes in the cluster dynamically.

 

etcdctl tool

 

Etcdctl is a client tool written in Go that comes in handy while manipulating an etcd cluster. It's commonly used to perform usual operations such as setting, updating or removing keys, although it's also possible through the etcd rest api. For a complete reference to these basic commands see the [official docs](https://github.com/coreos/etcd/tree/master/etcdctl).

 

Dynamically adding a node to the cluster

 

Adding and removing nodes to an existing etcd2 cluster is an operation that's done in runtime and therefore it doesn't require any downtime.

 

To dynamically add a node, first we need to create a new instance in EC2 that will join the cluster as an etcd member. Clone one of the nodes that we've set up earlier by right-clicking the instance in EC2 console and then launch the new instance. Once it's up, copy it's IP address.

 

Now ssh any of the already working nodes and issue the following command to turn etcd aware that a new node will join the cluster:

$ etcdctl member add my_new_node_name http://ip_address_here:2380

 

Copy the generated output and go back to EC2 console. Now we need to update the new instance with these provided information and reboot it. Right-click the new instance and stop it. Wait until the instance is stopped and right-click it again > Instance Settings > View/Change User Data. Now replace the output values you've copied in the following cloud-init file:

 

\#cloud-config

hostname: etcd-node
coreos:
  etcd2:
​    name: my_new_node_name
​    advertise-client-urls: http://$private_ipv4:2379
​    initial-advertise-peer-urls: http://$private_ipv4:2380
​    listen-client-urls: http://0.0.0.0:2379
​    listen-peer-urls: http://$private_ipv4:2380
​    initial-cluster: paste_here_the_output_you_copied
​    initial-cluster-state: "existing"
  units:
​    - name: etcd2.service
​      command: start

Note that name, initial-cluster and initial-cluster-state properties were added while the discovery token was removed. That's because now we must inform the already existing nodes in the cluster and not the discovery token. Save it and start the instance.

 

Now ssh the new node and issue the journalctl command to etcd2 service to see if everything is ok with etcd:

$ journalctl -fu etcd2

 

Now verify that the cluster contains four healthy nodes:

$ etcdctl cluster-health

 

That's it! You've just added a new node to your existing 3-node cluster in only a few minutes. In the future tutorials we'll understand Raft Consensus Algorithm as well as the pros and cons of adding/removing nodes to the cluster overall with respect to write/read performance and fault tolerance.

 

Dynamically removing a node in the cluster

 

If you followed the above instructions, you ended up with a 4-node etcd cluster. Now let's dynamically remove the node we've just added.

 

SSH any old node and find out the node id of the node you've just added by comparing its ip address:

$ etcdctl member list

 

Now you've got the its node_id, issue the following statement to remove the node from the etcd cluster:

$ etcdctl member remove node_id

 

Done! Now verify that there are only 3 nodes in the cluster again:

$ etcdctl cluster-health

 

Important Considerations

 

Once you fire the etcdctl member add/remove commands, you instantaneously change the cluster's quorum. That means that if you have a 3-node cluster and issue the command etcdctl member add three times repeatedly without in fact having added these nodes to the cluster, you'll end up with a failed cluster (and all your commands will fail, including etcdctl member remove) because you'll have 6 "declared" nodes with 3 "failed nodes", which is greater then the limit of 2 failures for a cluster with 6 nodes. See the fault tolerance table in [CoreOS admin guide docs](https://coreos.com/etcd/docs/latest/v2/admin_guide.html) for more details.

 

In order to avoid this kind of problem, be sure to add or remove one node at a time and assert that the operation was completed successfully.

Tags:

[coreos](https://www.jorgeacetozi.com/blog/tag/coreos)

[etcd](https://www.jorgeacetozi.com/blog/tag/etcd)

[etcdctl](https://www.jorgeacetozi.com/blog/tag/etcdctl)

###### 