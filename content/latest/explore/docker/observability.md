<h2 id="docker-create-universe">1. Create universe</h2>

If you have a previously running local universe, destroy it using the following.

```sh
$ ./yb-docker-ctl destroy
```

Start a new local universe with replication factor 3.

```sh
$ ./yb-docker-ctl create  --rf 3
```

<h2 id="docker-run-sample-app">2. Run sample key-value app</h2>

Pull the [yb-sample-apps](https://github.com/YugaByte/yb-sample-apps) docker container. This container has built-in Java client programs for various workloads including SQL inserts and updates.

```sh
$ docker pull yugabytedb/yb-sample-apps
```

Run a simple key-value workload in a separate shell.

```sh
$ docker run --name yb-sample-apps --hostname yb-sample-apps --net yb-net yugabytedb/yb-sample-apps --workload CassandraKeyValue \
  --nodes yb-tserver-n1:9042 \
  --num_threads_write 1 \
  --num_threads_read 4
```


<h2 id="docker-prepare-config">3. Prepare Prometheus config file</h2>

Copy the following into a file called `yugabytedb.yml`. Move this file to the `/tmp` directory so that we can bind the file to the Prometheus container later on.

```sh
global:
  scrape_interval:     5s # Set the scrape interval to every 5 seconds. Default is every 1 minute.
  evaluation_interval: 5s # Evaluate rules every 5 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# YugaByte DB configuration to scrape Prometheus time-series metrics 
scrape_configs:
  - job_name: 'yugabytedb'
    metrics_path: /prometheus-metrics

    static_configs:
      - targets: ['yb-master-n1:7000', 'yb-master-n2:7000', 'yb-master-n3:7000']
        labels:
          group: 'yb-master'

      - targets: ['yb-tserver-n1:9000', 'yb-tserver-n2:9000', 'yb-tserver-n3:9000']
        labels:
          group: 'yb-tserver'

      - targets: ['yb-tserver-n1:11000', 'yb-tserver-n2:11000', 'yb-tserver-n3:11000']
        labels:
          group: 'yedis'

      - targets: ['yb-tserver-n1:12000', 'yb-tserver-n2:12000', 'yb-tserver-n3:12000']
        labels:
          group: 'ycql'

      - targets: ['yb-tserver-n1:13000', 'yb-tserver-n2:13000', 'yb-tserver-n3:13000']
        labels:
          group: 'ysql'
```

<h2 id="docker-start-server">4. Start Prometheus server</h2>

Start the Prometheus server as below. The `prom/prometheus` container image will be pulled from the Docker registry if not already present on the localhost.

```sh
$ docker run \
	-p 9090:9090 \
	-v /tmp/yugabytedb.yml:/etc/prometheus/prometheus.yml \
	--net yb-net \
    prom/prometheus
```

Open the Prometheus UI at http://localhost:9090 and then navigate to the Targets page under Status.

![Prometheus Targets](/images/ce/prom-targets-docker.png)

<h2 id="docker-analyze-metrics">5. Analyze key metrics</h2>

On the Prometheus Graph UI, you can now plot the read/write throughput and latency for the `CassandraKeyValue` sample app. As we can see from the [source code](https://github.com/YugaByte/yugabyte-db/blob/master/java/yb-loadtester/src/main/java/com/yugabyte/sample/apps/CassandraKeyValue.java) of the app, it uses only SELECT statements for reads and INSERT statements for writes (aside from the initial CREATE TABLE). This means we can measure throughput and latency by simply using the metrics corresponding to the SELECT and INSERT statements.


Paste the following expressions into the Expression box and click Execute followed by Add Graph.

### Throughput


> Read IOPS

```sh
sum(irate(handler_latency_yb_cqlserver_SQLProcessor_SelectStmt_count[1m]))
```
![Prometheus Read IOPS](/images/ce/prom-read-iops.png)

>  Write IOPS

```sh
sum(irate(handler_latency_yb_cqlserver_SQLProcessor_InsertStmt_count[1m]))
```
![Prometheus Read IOPS](/images/ce/prom-write-iops.png)

### Latency


>  Read Latency (in microseconds)

```sh
avg(irate(handler_latency_yb_cqlserver_SQLProcessor_SelectStmt_sum[1m])) / avg(irate(handler_latency_yb_cqlserver_SQLProcessor_SelectStmt_count[1m]))
```
![Prometheus Read IOPS](/images/ce/prom-read-latency.png)


> Write Latency (in microseconds)

```sh
avg(irate(handler_latency_yb_cqlserver_SQLProcessor_InsertStmt_sum[1m])) / avg(irate(handler_latency_yb_cqlserver_SQLProcessor_InsertStmt_count[1m]))
```
![Prometheus Read IOPS](/images/ce/prom-write-latency.png)

<h2 id="docker-clean-up">6. Clean up (optional)</h2>

Optionally, you can shutdown the local cluster created in Step 1.

```sh
$ ./yb-docker-ctl destroy
```
