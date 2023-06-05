# CockroachDB Workload & Node Failure

```
cockroach workload init movr 'postgresql://root@localhost:26257?sslmode=disable'
```{{exec}}


```
cockroach sql --insecure
```{{exec}}

```
SHOW TABLES FROM movr;
```{{exec}}

```
SELECT * FROM movr.users WHERE city='new york';
```{{exec}}


```
\q
```{{exec}}


```
cockroach workload run movr --duration=1m 'postgresql://root@localhost:26257?sslmode=disable'
```{{exec}}

### Test Resiliency of the CockroachDB Cluster

* Bring down the node with SQL port 26257, from this perspective only an entry point is not available but the cluster continues to run and can be accessed via ports `26258 and 26259`.  Lets run the workload on the SQL port 26258 and check the UI console to review activity in the cluster, you will notice that one node is unavailable.

```
cockroach workload run movr --duration=1m 'postgresql://root@localhost:26258?sslmode=disable'
```{{exec}}

* Bring back up the node. This restores the node, see activity on the CockroachDB console on [CockroachDB UI Port 8081]({{TRAFFIC_HOST1_8081}}) or [CockroachDB UI Port 8082]({{TRAFFIC_HOST1_8082}})

```
cockroach start --insecure --store=cockroach-data/cockroach1 --listen-addr=localhost:26257 --http-addr=0.0.0.0:8080 --join=localhost:26257,localhost:26258,localhost:26259 --background
```{{exec}}

* Finally lets run some SQL commands to see what is different in CockroachDB with the purpose of seeing data distribution and flexibility to configure the cluster.

* Display the ranges information for `movr` database tables:

```
cockroach sql --insecure
```{{exec}}

```
SHOW RANGES FROM DATABASE movr;
```{{exec}}

* Get the CockroachDB version installed in the cluster, cluster settings can be extracted in two ways:

```
SELECT variable, value FROM [SHOW ALL CLUSTER SETTINGS] WHERE variable IN ('version');
```{{exec}}

```
SHOW CLUSTER SETTING version;
```{{exec}}

Next we will proceed to remove and decommission a node. We will be left with 2 nodes, which are still fully functional. For best practices, ensure that your cluster has the appropriate resources to be resilient, minimum 3 nodes.