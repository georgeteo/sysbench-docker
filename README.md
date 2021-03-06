# sysbench on Docker #

Sysbench 1.0.17 on Docker environment.

## Example Usage ##

### Requirement ###

Create a MySQL database and user for sysbench:

```bash
mysql> CREATE SCHEMA sbtest;
mysql> CREATE USER sbtest@'%' IDENTIFIED BY 'password';
mysql> GRANT ALL PRIVILEGES ON sbtest.* to sbtest@'%';
```

Or simply use the respective MySQL's image environment variables to create the database and user when running the MySQL container.

### Docker ###

Prepare the sysbench database:

```bash
$ docker run \
--rm=true \
--name=sb-prepare \
severalnines/sysbench \
sysbench \
--db-driver=mysql \
--oltp-table-size=100000 \
--oltp-tables-count=24 \
--threads=1 \
--mysql-host=10.0.0.51 \
--mysql-port=3306 \
--mysql-user=sbtest \
--mysql-password=password \
/usr/share/sysbench/tests/include/oltp_legacy/parallel_prepare.lua \
run
```

Run the benchmark for MySQL:

```bash
$ docker run \
--name=sb-run \
severalnines/sysbench \
sysbench \
--db-driver=mysql \
--report-interval=2 \
--mysql-table-engine=innodb \
--oltp-table-size=100000 \
--oltp-tables-count=24 \
--threads=64 \
--time=99999 \
--mysql-host=10.0.0.51 \
--mysql-port=3306 \
--mysql-user=sbtest \
--mysql-password=password \
/usr/share/sysbench/tests/include/oltp_legacy/oltp.lua \
run
```


### Kubernetes ###

You can run sysbench data preparation as a Job:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sysbench-prepare
spec:
  template:
    metadata:
      name: sysbench-prepare
    spec:
      containers:
      - name: sysbench-prepare
        image: severalnines/sysbench
        command:
        - sysbench
        - --db-driver=mysql
        - --oltp-table-size=100000
        - --oltp-tables-count=24
        - --threads=1
        - --mysql-host=galera
        - --mysql-port=3306
        - --mysql-user=sbtest
        - --mysql-password=password
        - /usr/share/sysbench/tests/include/oltp_legacy/parallel_prepare.lua
        - run
      restartPolicy: Never
```

Post the job to Kubernetes:

```
$ kubectl create -f sysbench-prepare-job.yml
```

Once the sysbench database is prepared, run the benchmark for MySQL through a Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: sysbench
  name: sysbench
spec:
  containers:
  - command:
    - sysbench
    - --db-driver=mysql
    - --report-interval=2
    - --mysql-table-engine=innodb
    - --oltp-table-size=100000
    - --oltp-tables-count=24
    - --threads=64
    - --time=99999
    - --mysql-host=galera
    - --mysql-port=3306
    - --mysql-user=sbtest
    - --mysql-password=password
    - /usr/share/sysbench/tests/include/oltp_legacy/oltp.lua
    - run
    image: severalnines/sysbench
    name: sysbench
  restartPolicy: Never
```

Post the job to Kubernetes:

```
$ kubectl create -f sysbench-pod.yml
```

See the sysbench reporting output using ``kubectl``:

```bash
$ kubectl logs -f sysbench
```
