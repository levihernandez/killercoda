# Download, Install, Test CockroachDB

Cockroach Labs has designed the installation process as a straight forward approach to increase productivity.

* First set a CockroachDB Release, choose one from the list found at [Install CockroachDB on Linux](https://www.cockroachlabs.com/docs/v23.1/install-cockroachdb-linux#install-binary)

```
export crdb_release="v23.1.2"; export architecture=$(dpkg --print-architecture)
```{{exec}}

* Download, decompress, and install the CockroachDB binary

```
cd /home/ubuntu; curl https://binaries.cockroachdb.com/cockroach-${crdb_release}.linux-${architecture}.tgz | tar -xz
```{{exec}}

* Copy the binary to the system bin in order to install it

```
cp -i cockroach-${crdb_release}.linux-${architecture}/cockroach /usr/local/bin/; which cockroach
```{{exec}}

* So far we have installed the binary, at this point we can deploy the first demo instance of CockroachDB and test the `movr` database and run some queries.

```
cockroach demo
```{{exec}}

* Run a couple of queries on the sample `movr` database

```
SHOW DATABASES;
```{{exec}}

```
USE movr;
```{{exec}}

```
SHOW TABLES;
```{{exec}}

```
SELECT * FROM users LIMIT 5;
```{{exec}}

* Exit the CockroachDB SQL Console

```
\q
```{{exec}}

Next, we will deploy the three nodes and access the CockroachDB UI Console