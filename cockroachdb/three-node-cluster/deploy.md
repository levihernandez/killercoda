# Deploy a 3 Node Cluster

Creating a sandbox environment in CockroachDB is as simple as following the next steps:

* Deploy node 1 in the background & access the CockroachDB UI Console

```
cockroach start --insecure --store=cockroach-data/cockroach1 --listen-addr=localhost:26257 --http-addr=0.0.0.0:8080 --join=localhost:26257,localhost:26258,localhost:26259 --background
```{{exec}}

We have assigned `--http-addr=0.0.0.0:8080` on this node to expose it to a public IP. To access the first node console, go to [CockroachDB UI Port 8080]({{TRAFFIC_HOST1_8080}}) after initializing the cluster.

* Deploy node 2 in the background & access the CockroachDB UI Console

```
cockroach start --insecure --store=cockroach-data/cockroach2 --listen-addr=localhost:26258 --http-addr=0.0.0.0:8081 --join=localhost:26257,localhost:26258,localhost:26259 --background
```{{exec}}

We have assigned `--http-addr=0.0.0.0:8081` on this node to expose it to a public IP. To access the second node console, go to [CockroachDB UI Port 8081]({{TRAFFIC_HOST1_8081}}) after initializing the cluster.

* Deploy node 3 in the background & access the CockroachDB UI Console

```
cockroach start --insecure --store=cockroach-data/cockroach3 --listen-addr=localhost:26259 --http-addr=0.0.0.0:8082 --join=localhost:26257,localhost:26258,localhost:26259 --background
```{{exec}}

We have assigned `--http-addr=0.0.0.0:8082` on this node to expose it to a public IP. To access the third node console, go to [CockroachDB UI Port 8082]({{TRAFFIC_HOST1_8082}}) after initializing the cluster.

* Initialize the cluster

So far, each node is running but will show as non functional until the cluster is initialized with the command below:

```
cockroach init --insecure --host=localhost:26257
```{{exec}}

Initializing the cluster can be performed against any CockroachDB SQL port (as given in this example it could be `26257, 26258, or 26259`).

CockroachDB eliminates the need for a single Master, single write node. All nodes are entry points to reads and writes.

* Lastly, access the CockroachDB to run a few SQL commands through any SQL port: `26257, 26258, or 26259`

```
cockroach sql --insecure --host=localhost:26257
```{{exec}}

```
SHOW DATABASES;
```{{exec}}

As you can see, the `movr` database we saw during the first demo does not exits. We can proceed to create it via a workload to test the cluster next. Let's quit the CockroachDB SQL console for now.

```
\q
```{{exec}}

* Optional: Although not covered in depth on this workshop, the most practical way to access the cluster would be through load balancers. See an example with [`cockroach gen haproxy`](https://www.cockroachlabs.com/docs/stable/cockroach-gen.html), which reads your cluster configuration and auto generates the haproxy.cfg with:

```
cockroach gen haproxy --insecure --host=localhost --port=26257
```{{exec}}

Install and set HAProxy to run in the background

```
apt install -y haproxy
```{{exec}}

```
haproxy -f haproxy.cfg -D
```{{exec}}

Because we are running haproxy and all 3 instances of CockroachDB in the same VM, the load balancer cannot bind to a port in use by CockroachDB. This scenario works when CockroachDB runs on dedicated nodes or K8s.

`[ALERT] 155/142030 (23803) : Starting proxy psql: cannot bind socket [0.0.0.0:26257]`

Next, we will create a TPCC workload and run it as well as test the failure of a node.