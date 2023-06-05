## CockroachDB Quickstart - Self-Hosted

The current workshop will help you deploy an insecure (no-SSL) CockroachDB cluster step by step.

Objectives:

* Download the latest binary (or a specific release). See releases listed in [CockroachDB Github](https://github.com/cockroachdb/cockroach/tags)
* Deploy 3 nodes, one by one, in the same host. Best practices are to deploy one CockroachDB instance per VM.
    * Monitor the activity of the cluster via either UI port:
        * [CockroachDB UI Port 8080]({{TRAFFIC_HOST1_8080}})
        * [CockroachDB UI Port 8081]({{TRAFFIC_HOST1_8081}})
        * [CockroachDB UI Port 8082]({{TRAFFIC_HOST1_8082}})
* Run the `movr` small workload & SQL queries that list the activity in the cluster
* Remove a node from the cluster
* Summary
