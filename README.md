# payments-processing

NiFi Setup

Load xml template from nifi directory
CMA-OD-Alerts.xml for data generation and alerting demo
Enable all Controller Services


Schema Registry 
```json
Schema Name: Transaction2

{
 "type": "record",
 "name": "Transaction",
 "namespace": "com.cloudera.example",
 "doc": "This is a sample transaction",
 "fields": [
  {
   "name": "timestamp_ms",
   "doc": "Timestamp of the collected readings.",
   "type": "long"
  },
  {
   "name": "created_at",
   "doc": "Transaction identification number.",
   "type": "string"
  },
  {
   "name": "amount",
   "doc": "Transaction amount.",
   "type": "int"
  },
  {
   "name": "text",
   "doc": "Text",
   "type": "string"
  },
  {
   "name": "id_customer",
   "doc": "Customer id",
   "type": "long"
  }
 ]
}
```

Impala Setup
Create transactions table to query transactions in real time as they land in Kudu

```sql
CREATE TABLE transactions
(
 timestamp_ms BIGINT,
 created_at STRING,
 amount INT,
 text STRING,
 id_customer BIGINT,
 PRIMARY KEY (timestamp_ms)
)
PARTITION BY HASH PARTITIONS 16
STORED AS KUDU
TBLPROPERTIES ('kudu.num_tablet_replicas' = '1');
```

Kafka Topic Creation:

customer_balance_alert
customer_balance
cma_od_alert_response
cma_od_alert_request

Flink Setup
Load jar file and job.properties file to /tmp on your cluster node.

NOTE: update the job.properties file to include your Kafka Broker URL

Run Demo
Start all NiFi Processors except “Microservice Push (Broken Schema)” until needed
Run Flink Job. Run as admin OR create HDFS home directory for user under /user 

flink run --jobmanager yarn-cluster --detached --parallelism 2 --yarnname HeapMonitor /tmp/flink-transactions-1.0-SNAPSHOT.jar /tmp/job.properties

Log into Hue as admin to query default.transactions table. 
NOTE: If NiFi PutKudu processor complains that “key already present” - change Kudu Operation Type config parameter to INSERT_IGNORE
