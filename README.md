# Replication and Fault Tolerance with Apache Cassandra

This demo provides a step-by-step guide and console commands to show how
[Apache Cassandra](https://cassandra.apache.org/_/cassandra-basics.html) handles data replication and ensures fault tolerance in a distributed environment.


## Demo

### Creating a KeySpace and a Table

Start `cqlsh`, the Cassandra console:

```bash
CREATE KEYSPACE demo_keyspace WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3};
```

This command creates a keyspace named "DemoKeyspace" with a simple replication strategy and a replication factor of 3. This means that the data will be replicated to three nodes.
nodes.

### Creates a table in the keyspace

```bash
USE demo_keyspace;
CREATE TABLE demo_table (id UUID PRIMARY KEY, name TEXT, age INT);
```

### Add and query data

```bash
INSERT INTO demo_table (id, name, age) VALUES (uuid(), 'John Doe', 25);
SELECT * FROM demo_table;
```

### Verify replication

```bash
CONSISTENCY QUORUM;
SELECT * FROM demo_table WHERE id = <id>;
```

By setting consistency in QUORUM, we are ensuring that at least most of the nodes are responsive before we consider reading or writing nodes respond before considering the read or write as successful. You can run SELECT queries to verify that data is read correctly from multiple nodes.


### Simulating a failure

Stop a Cassandra node by:

```bash
$ docker compose stop <node>
```


### Reactivation of the stopped node

```bash
$ nodetool resumehandoff <node_id>
```

You can monitor the synchronization process using ``nodetool`:

```bash
$ nodetool status
```

## Consistency Levels

Consistency levels control how data consistency is ensured across nodes in the cluster. There are various consistency levels available, and choosing a specific level affects the trade-off between consistency and availability. Here are the main consistency levels in Cassandra:

ANY: This consistency level indicates that it is sufficient for the write or read to occur on at least one node. There is no guarantee that the data will be immediately replicated to other nodes.

ONE: The operation is considered successful if it is performed on at least one node responsible for the partition. This consistency level provides low latency and is suitable for situations where immediate consistency is not critical.

TWO, THREE, QUORUM: These levels require the operation to be performed on a specific number of nodes owning the partition. In the case of QUORUM, a majority of nodes is required (number of nodes / 2 + 1). These levels offer a balance between consistency and availability.

ALL: The operation is considered successful only if it is performed on all nodes responsible for the partition. It provides the highest consistency but may result in higher latency and lower availability, as all nodes must be available to complete the operation.

LOCAL_ONE, LOCAL_QUORUM, EACH_QUORUM: These levels are used in clusters distributed across multiple datacenters. They allow specifying consistency levels for operations affecting local nodes or requiring the participation of nodes in different datacenters.

SERIAL and LOCAL_SERIAL: These levels are used in conjunction with lightweight transaction (LWT) read and write operations, providing a mechanism to ensure consistency across transaction-like operations.

The choice of consistency level depends on the specific requirements of the application, considering factors such as desired consistency, tolerated latency, and availability. It is important to understand the performance and latency implications when selecting a particular consistency level, as higher consistency levels may result in lower availability and longer response times.
