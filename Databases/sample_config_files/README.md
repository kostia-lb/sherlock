### Sample config files
This directory hold sample config files for:<br/>
* sysbench (MySQL and PostgreSQL)
* PGbench (PostgreSQL)
* HammerDB (SQL Server)
* YCSB (MongoDB)<br/>

Below you'll find a more detail explanation of the variables and how to use them.<br/>
Some variables starts with the workload name, like SYSBENCH_, PGBENCH_, HAMMERDB_ and YCSB_. These variables are specific to the workload.

```bash
# the sherlock config

#
# workload related
# 

# How long the workload will run in seconds (must be >= 60 for hammerdb)
readonly WORKLOAD_RUNTIME=300

# How many connections to open to each database. for sysbench and pgbench usage
readonly THREADS=2
readonly CLIENTS=2
# When possible to control the output, how often to print the output (in seconds)
readonly OUTPUT_INTERVAL=10

# for PostgreSQL using sysbench 400 tables, each with 1000000 rows is roughly 100G in DB size
readonly SYSBENCH_NUMBER_OF_TABLES=150
readonly SYSBENCH_ROWS_IN_TABLE=10000

# Inserts, Updates and non index updates impact the ratio of read/write. When all 3 are set to 1, the ratio is roughly 70r/30w.
# When all 3 are set to 1, the ratio is roughly 70r/30w. you can increase the numbers to increase writes.
readonly SYSBENCH_NUMBER_OF_INSERTS=1
readonly SYSBENCH_NUMBER_OF_UPDATES=1
readonly SYSBENCH_NUMBER_OF_NON_INDEX_UPDATES=1

readonly SYSBENCH_READ_ONLY=off
readonly SYSBENCH_WRITE_ONLY=off

# For pgbench sherlock support simple, 90r10w, 70r30w, or readonly options.
readonly PGBENCH_WORKLOAD_TYPE=70r30w

# For pgbench you can choose to run the workload WORKLOAD_RUNTIME seconds or set it to transactions and use PGBENCH_RUN_TYPE_VAR to set how many to perform.
readonly PGBENCH_RUN_TYPE=time
readonly PGBENCH_RUN_TYPE_VAR=${WORKLOAD_RUNTIME}

readonly PGBENCH_VACUUM=yes
readonly PGBENCH_QUIET=no
readonly PGBENCH_SCALE=700
readonly PGBENCH_READ_ONLY=no

# The number of warehouses will impact the database size on the disk. 200 is roughly 25GB.
readonly HAMMERDB_WAREHOUSES=50
# How many Virtual Users to use to build the databases. Used in the prepare part, when we're creating the data.
readonly HAMMERDB_BUILD_VU=50
# How many Virtual Users will be used during the actual run.
readonly HAMMERDB_RUN_VU=50
# The rampup time in seconds where transactions are running but not yet calculated, used to warmup storage.
readonly HAMMERDB_RAMPUP=60
# If you want to capture failures (for example, during resiliency testing), set to true to get some feedback from hammerdb on failed users.
readonly HAMMERDB_RAISE_ERROR=false

# YCSB have several workloads, workload a..f, A is write intensive, B is what I mostly use for storage, tweaking it with read and update proportions (see below), C is read only, D & F will test storage cache rather than storage (queries on recently inserted data or Read-Modify-Read), E is for short range queries.
readonly YCSB_WORKLOAD=workloadb

# These variables are pretty self explanatory, this the way in YCSB to control the IO ratio of read/write. As long as you database is a lot bigger than DB_POD_MEM, you will see the ratio in the actual SDS.
readonly YCSB_READPROPORTION=70
readonly YCSB_UPDATEPROPORTION=30

# How many rows in the usertable collection.
readonly YCSB_RECORDCOUNT=1000000
# Since YCSB is more operation count oriented vs time, I suggest you leave the operation count to a very high number because this way we can force YCSB to run by time and collect stats by time.
readonly YCSB_OPERATIONCOUNT=10000000
readonly YCSB_DISTRIBUTION=uniform


# Memory Limit for the benchmark pod.
readonly BENCHMARK_POD_MEM_LIMIT=2Gi


#
# cluster related
#

# The binary used to run the scripts, kubectl or oc
readonly KUBE_CMD=kubectl

# The number of worker nodes to deploy DBs on. if the number is smaller than nodes in WORKERS_LIST_FILE, not all worker nodes will be used.
readonly NUMBER_OF_WORKERS=3

# How many databases to create per worker node
readonly DB_PER_WORKER=2

# List of nodes that will run the databases
readonly WORKERS_LIST_FILE=~/workers

# List of nodes that will run the SDS. Can be the same value as WORKERS_LIST_FILE when running in converge manner.
readonly SDS_LIST_FILE=~/sds_nodes

# Set the project or namespace that will be used to run all the sherlock pods. If NAMESPACE_NAME does not exists, it will be created.
# For convenience, I recommend using postgresql, mysql, mongodb or sqlserver, but these names are not hardcoded and you can use whatever you choose.
readonly NAMESPACE_NAME=mysql 


#
# database related
#

# What type of database we are using: postgresql, mysql, mongodb or sqlserver
readonly DB_TYPE=mysql

# This prefix will be added to any sherlock pod for better detection. 
# For convenience, I recommend using postgresql, mysql, mongodb or sqlserver, but these names are not hardcoded and you can use whatever you choose.
readonly DB_POD_PREFIX=mysql


# This prefix will be added to any PVC created by sherlock for better detection. 
# For convenience, I recommend using postgresql, mysql, mongodb or sqlserver, but these names are not hardcoded and you can use whatever you choose.
readonly DB_PVC_PREFIX=mysql-pvc

# the size of PVC created for each database. make sure your SDS have enough storage to support PVC_SIZE * DB_PER_WORKER * NUMBER_OF_WORKERS
readonly PVC_SIZE=50Gi

# DB_POD_MEM and DB_POD_CPU are basically the values set in the pod for the resources. Sherlock will use both to set limits and requests equally, meaning the database pod will use both these variables when the database starts.
readonly DB_POD_MEM=2Gi
readonly DB_POD_CPU=1

# Self explanatory
readonly DB_NAME=sherlock
readonly DB_USERNAME=sherlock
readonly DB_PASSWORD=sherlock

# For SQL server we must use an admin password that is complex, if you change it, make sure it has at least 8 characters long, lower and upper case and includes a number as well.
readonly SA_PASSWORD=RHsqlserver2019

# What storage class to use to create the PVCs that the databases will used. This will derive from whatever SDS you are testing.
readonly STORAGE_CLASS=my_storage_class


#
# stats related
#

# If set to true, sherlock will deploy tiny containers on each node that appears in WORKERS_LIST_FILE and SDS_LIST_FILE to capture performance such as IOs, CPU, Memory and Network.
# At the end of each run the logs of these pods will be retrieved and you can view the output or the summary of usage.
readonly STATS=true

# How often stats will be collected, in seconds.
readonly STATS_INTERVAL=10

# list of the sds devices (seperated by space) that are used for the SDS, for example, if the OSD devices are /dev/sdd and /dev/sde the value should be "sdd sde"
readonly SDS_DEVICES="nvme0n1 nvme1n1"

# this array holds a list of the sds network interface/s (seperated by space) per worker (worker nodes might have different ethernet names).
declare -A NODE_NETWORK_MAP=(
[rack04-server1]="ens785f0 ens785f1"
[rack04-server2]="ens785f0 ens785f1"
[rack04-server3]="ens785f0 ens785f1"
[rack04-server4]="ens785f2 ens785f3"
[rack04-server5]="ens785f0 ens785f1"
[rack04-server6]="ens785f0 ens785f1"
[rack04-server7]="ens785f0 ens785f1"
[rack04-server8]="ens785f2 ens785f3"
[rack04-server9]="ens785f0 ens785f1"
[rack04-server10]="ens785f0 ens785f1")

# script related
readonly DEBUG=false

# in some cases nodes running the SDS will be tainted, sherlock will *try* to match the first taint and apply it to the stats pods
readonly SDS_NODE_TAINTED=true

# give a name for your storage solution so you can compare
readonly STORAGE_VENDOR=my_storage_vendor

# this variable sets whether the storage runs inside (part of) the K8s cluster or externally (options are internal/external). If internal, sherlock can collect stats on the storage devices that the internal SDS pods uses.
readonly STORAGE_TYPE=external
```
