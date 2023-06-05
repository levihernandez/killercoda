## Decomission a Node

See Best Practices for node decommission: [Node Removal](https://www.cockroachlabs.com/docs/stable/node-shutdown.html)

There are two steps for removing a node from the cluster:

* Node Drain
* Node Decomission

### Node Drain

It is important to drain a node before complete removal to decommission load balancer connections as a best practice. In the event of a node failure, the application side can rely on the application connection retry method and/or the load balancer. We are covering the highlights to retire a node with a 3 node cluster. In practice, the architecture must have a way to add new nodes when a node fails or 5 nodes minimum to account for time to restore a node, architectures and use cases may vary depending on the need for multi-region.

```
SET CLUSTER SETTING server.shutdown.drain_wait = '8s';
```{{exec}}

```
SET CLUSTER SETTING server.time_until_store_dead = '15m0s';
```{{exec}}

```
\q
```{{exec}}



* Best practices dictate that our cluster must have minimal number of nodes in order to stay healthy. We will add a fourth node for scalability here to ensure we can decommission the first node.
This ensures that the first node's data is transitioned to the new node. Observe how data moves to the new node in [CockroachDB UI Port 8083]({{TRAFFIC_HOST1_8083}}/#/metrics/replication/cluster) `Replicas per Node` graph.

```
cockroach start --insecure --store=cockroach-data/cockroach4 --listen-addr=localhost:26260 --http-addr=0.0.0.0:8083 --join=localhost:26258,localhost:26259,localhost:26260 --background
```{{exec}}

We have assigned `--http-addr=0.0.0.0:8083` on this node to expose it to a public IP. To access the third node console, go to [CockroachDB UI Port 8083]({{TRAFFIC_HOST1_8083}}) after initializing the cluster.



* Check the progress of the node shutdown in the log and the `cockroach node status`

```
cockroach node drain 1 --host=localhost:26257 --drain-wait=15m --insecure
```{{exec}}

* The log provides a sequence of events during the draining of the node:

```
grep 'drain' cockroach-data/cockroach1/logs/cockroach.log
```{{exec}}



* Since the node using the port `26257` is being drained, lets use the next available port `26258` to check the status of the node. The SQL result in the `is_draining` column has registered the `id=1`  as `true` to drain the node.

```
cockroach node status --decommission --insecure --host=localhost:26258
```{{exec}}

* Lastly lets remove the node by decommissioning it [CockroachDB UI Port 8083]({{TRAFFIC_HOST1_8083}}/#/overview/list) and notice the replicas are the same across every node. Your data just rebalanced with `zero downtime`.

```
cockroach node decommission 1 --insecure --host=localhost:26258
```{{exec}}

Congratulations! You have removed a node from the CockroachDB cluster.