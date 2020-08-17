# cassandra-app
Connecting Cassandra Application


How to install ccm 

https://www.datastax.com/blog/2013/01/ccm-development-tool-creating-local-cassandra-clusters

### Introduction
CCM (Cassandra Cluster Manager) is a tool written by Sylvain Lebresne that creates multi-node cassandra clusters on the local machine. It is great for quickly setting up clusters for development and testing, and is the foundation that the cassandra distributed tests (dtests) are built on. In this post I will give an introduction to installing and using ccm.

### Installing CCM
CCM depends on the cql and PyYAML PyPI packages. For example, on ubuntu you can install them like this:

sudo apt-get install -y python-pip; sudo pip install cql PyYAML
The recommend source of CCM is Sylvain’s git repo:

'git clone https://github.com/pcmanus/ccm.git'
Then install it like this:

'cd ccm; sudo ./setup.py install; cd .. '
Creating a CCM cluster
At a minimum, setting up a CCM cluster requires the cassandra version, the number of nodes, and the name of the cluster. Here is an example that sets up and starts a 3-node cluster named “test” on cassandra version 1.2.0:

ccm create --version 1.2.0 --nodes 3 --start test
CCM will download the specified version of cassandra to ~/.ccm/repository/. It will customize three .yaml files and specify separate data directories and log file locations. Data for each node will be stored in ~/.ccm/<cluster_name>/<node>/. By default the thrift interfaces for the nodes are 127.0.0.1, 127.0.0.2, etc...

CCM supports a few notable alternatives to downloading a packaged version of cassandra. You can specify a tag or branch in the git repository like this: --version git:trunk, and ccm will download and compile the source. Also, you can specify the location of a cassandra directory that you have already compiled using --cassandra-dir instead of --version.

A new node can be bootstrapped with the "ccm add" command, as in the following example.

ccm add --itf 127.0.0.4 --jmx-port 7400 -b node4
Note that I've used "127.0.0.4" for the thrift interfaces because .1, .2, and .3 were already taken for the first three nodes. The jmx port was set to 7400 because 7100, 7200, and 7300 were likewise used for the first three nodes. The node has now been added to the cluster, and can be started with "ccm node4 start".

### CCM from the command line
CCM allows you to create multiple clusters. Only one cluster is active at a time, and all ccm commands will apply to the active cluster. Switch clusters with the "ccm switch" command.

Many commands apply to a single node. For example, "ccm node1 ring" connects to node1 and displays the nodetool ring info for the cluster, while "ccm node1 showlog" displays the cassandra log for node1. Node are named "node1", "node2", etc. When you are finished with the cluster, "ccm remove" will shut it down and delete all the data.

The log for any node can be accessed with "ccm <node> showlog". You can connect to a node with "ccm <node> cqlsh", or using the thrift interface 127.0.0.<node_number>.

There are many more ccm commands that are not covered here. "ccm" (without any arguments) will display all available ccm commands, and "ccm <command> -h" will show how to use the command.

### Upgrading a cluster
Upgrading is an important operation in a cassandra cluster. The upgrade procedure in a ccm cluster is similar to any other cassandra cluster. Shut down a node, upgrade it to the new version of cassandra, update the cassandra.yaml file as needed, and then start the node again. Here is an example of how to upgrade a ccm cluster on 1.1.8 to the git repo branch cassandra-1.2.0:

ccm node1 stop
ccm node1 setdir --cassandra-version git:cassandra-1.2.0
ccm node1 updateconf 'partitioner: org.apache.cassandra.dht.RandomPartitioner'
ccm node1 start
ccm node2 stop...
...and so on for the other nodes. We've updated the partitioner in the conf because the default partitioner changed between cassandra 1.1 and 1.2. Note that this upgrade was done in a rolling fashion, one node at a time. It could also be done all at once by excluding the node from all the commands above, which will cause the commands to apply cluster-wide.

### Conclusion
CCM is a great tool for quickly setting up a cluster on your local machine for development or testing. Some ccm commands apply to individual nodes, some to the whole cluster, and some can work either way. This post has only covered a subset of the available features off ccm, the full documentation can be found by running "ccm" to see a list of commands, and "ccm <command> -h" for information on an individual command.