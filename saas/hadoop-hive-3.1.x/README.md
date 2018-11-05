# OverOps Hadoop 3.1.x and Hive 3.1.x Example
This is an example of using OverOps to monitor a single node Apache Hadoop cluster with Apache Hive.  The `docker-compose.yml` contains the following services:
* `namenode` - Apache Hadoop NameNode
* `datanode` - Apache Hadoop DataNode
* `resourcemanager` - Apache Hadoop YARN Resource Manager
* `nodemanager` - Apache Hadoop YARN Node Manager
* `historyserver` - Apache Hadoop YARN Timeline Manager
* `hs2` - Apache Hive HiveServer2
* `metastore` - Apache Hive Metastore
* `metastore-db` - Postgres DB that supports the Apache Hive Metastore
* `collector` - an OverOps Collector running in a dedicated container (aka Remote Collector)
* `sidecar` - an OverOps Agent running in a dedicated container whose directory is exposed as a Docker `volume` mount

## Getting Started
To begin, you must first create a file called `overops-key.env` and place it in the same directory as the `Dockerfile`.  Below is a sample `overops-key.env` file.

### overops-key.env
```properties
TAKIPI_SECRET_KEY=your-very-own-overops-secret-key
```

## Configuration
Hadoop configuration parameters are provided by the following `.env` files.  Ultimately these values are written to the appropriate Hadoop XML configuration file.  For Example, properties beginning with the following keys map the following files:
* `CORE_CONF_*` > `core-site.xml`
* `HDFS_CONF_*` > `hdfs-site.xml`
* `HIVE_SITE_CONF_*` > `hive-site.xml`
* `YARN_CONF_*` > `yarn-site.xml`

Key names use the following character conversions:
* a single underscore `_` equals dot `.`
* a double underscore `__` equals a single underscore `_`
* a triple underscore `___` equals a dash `-`

For example, the key `HDFS_CONF_dfs_namenode_datanode_registration_ip___hostname___check` would result in the property `dfs.namenode.datanode.registration.ip-hostname-check` being written to `hdfs-site.xml`.

Another example, the key `YARN_CONF_yarn_resourcemanager_resource__tracker_address` would result in the property `yarn.resourcemanager.resource_tracker.address` being written to `yarn-site.xml`.

Exiting configuration files and their default values are listed below.  Please note the value for `YARN_CONF_yarn_nodemanager_resource_memory___mb` assumes that your docker host has atleast 8gb of memory.  Feel free to modify as necessary. 

### core.env
```properties
CORE_CONF_fs_defaultFS=hdfs://namenode:9820
CORE_CONF_hadoop_http_staticuser_user=root

HDFS_CONF_dfs_namenode_datanode_registration_ip___hostname___check=false
HDFS_CONF_dfs_permissions_enabled=false
HDFS_CONF_dfs_webhdfs_enabled=true
```

### hive.env
```properties
HIVE_SITE_CONF_javax_jdo_option_ConnectionURL=jdbc:postgresql://metastore-db/metastore
HIVE_SITE_CONF_javax_jdo_option_ConnectionDriverName=org.postgresql.Driver
HIVE_SITE_CONF_javax_jdo_option_ConnectionUserName=hive
HIVE_SITE_CONF_javax_jdo_option_ConnectionPassword=hive
HIVE_SITE_CONF_hive_server2_transport_mode=binary
HIVE_SITE_CONF_hive_execution_engine=tez
HIVE_SITE_CONF_datanucleus_schema_autoCreateAll=false
HIVE_SITE_CONF_datanucleus_autoStartMechanismMode=ignored
```

### yarn.env
```properties
YARN_CONF_yarn_log___aggregation___enable=true
YARN_CONF_yarn_log_server_url=http://historyserver:8188/applicationhistory/logs/

YARN_CONF_yarn_resourcemanager_address=resourcemanager:8032
YARN_CONF_yarn_resourcemanager_hostname=resourcemanager
YARN_CONF_yarn_resourcemanager_recovery_enabled=true
YARN_CONF_yarn_resourcemanager_resource__tracker_address=resourcemanager:8031
YARN_CONF_yarn_resourcemanager_scheduler_address=resourcemanager:8030
YARN_CONF_yarn_resourcemanager_store_class=org.apache.hadoop.yarn.server.resourcemanager.recovery.FileSystemRMStateStore
YARN_CONF_yarn_resourcemanager_system___metrics___publisher_enabled=true

YARN_CONF_yarn_timeline___service_enabled=true
YARN_CONF_yarn_timeline___service_generic___application___history_enabled=true
YARN_CONF_yarn_timeline___service_hostname=historyserver

YARN_CONF_yarn_nodemanager_resource_memory___mb=6144
YARN_CONF_yarn_nodemanager_aux___services=mapreduce_shuffle
YARN_CONF_yarn_nodemanager_aux___services_mapreduce__shuffle_class=org.apache.hadoop.mapred.ShuffleHandler
```

### metastore.env
```properties
METASTORE_SITE_CONF_metastore_thrift_uris=thrift://metastore:9083
```

### overops-agent.env
```properties
JAVA_TOOL_OPTIONS=-agentpath:/sidecar/lib/libTakipiAgent.so
TAKIPI_COLLECTOR_HOST=collector
TAKIPI_COLLECTOR_PORT=6060
```

## Docker Compose

### Start the Containers
```bash
docker-compose up
```

### Stop and Destroy the Containers
```bash
docker-compose down
```

## Testing
Once all services are up you can create a simple hive table to test functionality.  For example:

```bash
$ docker-compose exec hs2 bash
# /opt/hive/bin/beeline -u jdbc:hive2://localhost:10000
> CREATE TABLE pokes (foo INT, bar STRING);
> LOAD DATA LOCAL INPATH '/opt/hive/examples/files/kv1.txt' OVERWRITE INTO TABLE pokes;
> SELECT * FROM pokes;
> !quit
```

## Docker Images
* Hadoop NameNode - [timveil/docker-hadoop-namenode:3.1.x](https://hub.docker.com/r/timveil/docker-hadoop-namenode/)
* Hadoop DataNode - [timveil/docker-hadoop-datanode:3.1.x](https://hub.docker.com/r/timveil/docker-hadoop-datanode/)
* YARN Resource Manager - [timveil/docker-hadoop-resourcemanager:3.1.x](https://hub.docker.com/r/timveil/docker-hadoop-resourcemanager/)
* YARN Node Manager - [timveil/docker-hadoop-nodemanager:3.1.x](https://hub.docker.com/r/timveil/docker-hadoop-nodemanager/)
* YARN Timeline Server - [timveil/docker-hadoop-historyserver:3.1.x](https://hub.docker.com/r/timveil/docker-hadoop-historyserver/)
* Hive Hiverserver2 - [timveil/docker-hadoop-hive-hs2:3.1.x](https://hub.docker.com/r/timveil/docker-hadoop-hive-hs2/)
* Hive Metastore - [timveil/docker-hadoop-hive-metastore:3.1.x](https://hub.docker.com/r/timveil/docker-hadoop-hive-metastore/)
* Hive Metastore Postgres DB - [timveil/docker-hadoop-hive-metastore-db:3.1.x](https://hub.docker.com/r/timveil/docker-hadoop-hive-metastore-db/)
* Remote Collector - [timveil/oo-docker-remote-collector](https://hub.docker.com/r/timveil/oo-docker-remote-collector/)
* Agent Sidecar - [timveil/oo-docker-agent-sidecar](https://hub.docker.com/r/timveil/oo-docker-agent-sidecar/)
