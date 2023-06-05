## CockroachDB Workload & Node Failure

* Initialize the CockroachDB workload called `movr`

```
cockroach workload init movr 'postgresql://root@localhost:26257?sslmode=disable'
```{{exec}}

* Access the SQL console to see the `movr` database and run a query

```
cockroach sql --insecure
```{{exec}}

```
SHOW TABLES FROM movr;
```{{exec}}

```
SELECT * FROM movr.users WHERE city='new york';
```{{exec}}

* Exit the CockroachDB SQL console

```
\q
```{{exec}}

* Run a workload with duration of `1m` against the cluster via the first node with SQL port `26257`

```
cockroach workload run movr --duration=1m 'postgresql://root@localhost:26257?sslmode=disable'
```{{exec}}

* Check the activity at [CockroachDB UI Port 8081]({{TRAFFIC_HOST1_8081}}/#/metrics/overview/cluster) to determine the Service Latency with 3 nodes.

### Test Resiliency of the CockroachDB Cluster

* Bring down the node with SQL port 26257, from this perspective only an entry point is not available but the cluster continues to run and can be accessed via ports `26258 and 26259`.  Lets run the workload on the SQL port 26258 and check the UI console to review activity in the cluster, you will notice that one node is unavailable.

> Kill the process id for the first CockroachDB node

```
kill -9 $(ps -eaf | grep -v "grep" | grep cockroach1 | awk '{print $2}')
```{{exec}}

* See the result of the node crash in [CockroachDB UI Port 8081]({{TRAFFIC_HOST1_8081}})

* Execute a new workload on the next available SQL port and notice that CockroachDB continues to run:

```
cockroach workload run movr --duration=1m 'postgresql://root@localhost:26258?sslmode=disable'
```{{exec}}

* See the Cluster activity during the workload execution at [CockroachDB UI Port 8081]({{TRAFFIC_HOST1_8081}}/#/metrics/overview/cluster)

Notice that the SQL statements provide activity for the two remaining nodes and also give the Service Latency percentiles while the node is down.


* Bring back up the node and see the activity in the UI console: [CockroachDB UI Port 8081]({{TRAFFIC_HOST1_8081}}) or [CockroachDB UI Port 8082]({{TRAFFIC_HOST1_8082}})

```
cockroach start --insecure --store=cockroach-data/cockroach1 --listen-addr=localhost:26257 --http-addr=0.0.0.0:8080 --join=localhost:26257,localhost:26258,localhost:26259 --background
```{{exec}}

* Finally lets run some SQL commands to see cluster settings and database ranges.

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

Next we will proceed to remove and decommission a node. We will be left with 2 nodes but we will add a new one, which are still fully functional. For best practices, ensure that your cluster has the appropriate resources to be resilient, minimum 3 nodes.