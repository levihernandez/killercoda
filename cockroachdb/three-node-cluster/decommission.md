# Decomission a Node

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

* Check the progress of the node shutdown in the log and the `cockroach node status`

```
cockroach node drain 1 --host=localhost:26257 --drain-wait=15m --insecure
```{{exec}}

```
grep 'drain' cockroach-data/cockroach1/logs/cockroach.log
```{{exec}}

* Since the node using the port `26257` is being drained, lets use the next available port `26258` to check the status of the node. The SQL result in the `is_draining` column has registered the `id=1`  as `true` to drain the node.

```
cockroach node status --decommission --insecure --host=localhost:26258
```{{exec}}

* Lastly lets remove the node by decommissioning it, leaving a 2 node cluster (best practices are to always have a 3 node cluster for proper resiliency)

```
cockroach node decommission 1 --insecure --host=localhost:26258
```{{exec}}

Congratulations! You have removed a node from the CockroachDB cluster.