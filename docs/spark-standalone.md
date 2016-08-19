---
layout: global
title: Spark Standalone Mode
---

* 이 것은 컨텐츠의 목차가 될 것이다 (이 텍스트는 폐기 될 것이다).
{:toc}

In addition to running on the Mesos or YARN cluster managers, Spark also provides a simple standalone deploy mode. You can launch a standalone cluster either manually, by starting a master and workers by hand, or use our provided [launch scripts](#cluster-launch-scripts). It is also possible to run these daemons on a single machine for testing.

Mesos나 YARN 클러스터 관리자에서 실행하기 위해, 스파크는 간단한 스탠드얼론(독립형) 배포 모드를 제공한다. 마스터와 워커를 시작하거나 제공되는 [실행 스크립트](#cluster-launch-scripts)를 사용하여 스탠드얼론 클러스터를 독립적으로 실행할 수 있다. 또한, 테스트를 위해 싱글 머신에서 데몬으로 실행할 수 있다.

# Installing Spark Standalone to a Cluster
# 클러스터에 스파크 스탠드얼론 설치하기

To install Spark Standalone mode, you simply place a compiled version of Spark on each node on the cluster. You can obtain pre-built versions of Spark with each release or [build it yourself](building-spark.html).

스파크 스탠드얼론 모드를 실행하려면 클러스터의 각 노드에 컴파일된 스파크 버전을 위치시키면 된다. 스파크의 pre-built 버전(바이너리)을 다운로드 하거나 [자신이 직접 빌드](building-spark.html)를 할 수 있다.

# Starting a Cluster Manually
# 수동으로 클러스터 시작하기

You can start a standalone master server by executing:
쉘을 실행하여 스탠드얼론 마스터를 시작할 수 있다:

    ./sbin/start-master.sh

Once started, the master will print out a `spark://HOST:PORT` URL for itself, which you can use to connect workers to it,
or pass as the "master" argument to `SparkContext`. You can also find this URL on
the master's web UI, which is [http://localhost:8080](http://localhost:8080) by default.

마스터가 시작되면, 자신에 대한 `spark://HOST:PORT` URL을 출력할 것이다, 워커를 연결하거나 `SparkContext`에 "master" 인수를 보내거나 할 때 URL을 사용할 있다. 마스터의 웹 UI에서 이 URL을 확인 할 수 있다. 기본값은 [http://localhost:8080](http://localhost:8080) 이다.

Similarly, you can start one or more workers and connect them to the master via:
마찬가지로, 쉡 명령을 사용하여 하나 이상의 워커를 시작할 수 있고 마스터로 연결할 수 있다:

    ./sbin/start-slave.sh <master-spark-URL>

Once you have started a worker, look at the master's web UI ([http://localhost:8080](http://localhost:8080) by default).
You should see the new node listed there, along with its number of CPUs and memory (minus one gigabyte left for the OS).

워커가 시작되면 마스터의 웹 UI[http://localhost:8080](http://localhost:8080)에서 확인할 수 있다.
CPU, 메모리들과 함께 새로운 노드들을 확인할 수 있다. (minus one gigabyte left for OS).

Finally, the following configuration options can be passed to the master and worker:
최종적으로, 다음 설정 옵션들을 마스터와 워커로 보낼 수 있다:

<table class="table">
  <tr><th style="width:21%">Argument</th><th>Meaning</th></tr>
  <tr>
    <td><code>-h HOST</code>, <code>--host HOST</code></td>
    <td>Hostname to listen on
    호스트명</td>
  </tr>
  <tr>
    <td><code>-i HOST</code>, <code>--ip HOST</code></td>
    <td>Hostname to listen on (deprecated, use -h or --host)
    호스트명 (deprecated, use -h or --host)</td>
  </tr>
  <tr>
    <td><code>-p PORT</code>, <code>--port PORT</code></td>
    <td>Port for service to listen on (default: 7077 for master, random for worker)
    서비스에 대한 포트 (기본값: 마스터 - 7077, 워커 - 랜덤)</td>
  </tr>
  <tr>
    <td><code>--webui-port PORT</code></td>
    <td>Port for web UI (default: 8080 for master, 8081 for worker)
    웹 UI에 대한 포트 (기본값: 마스터 - 8080, 워커 - 8081)</td>
  </tr>
  <tr>
    <td><code>-c CORES</code>, <code>--cores CORES</code></td>
    <td>Total CPU cores to allow Spark applications to use on the machine (default: all available); only on worker
    머신에서 사용하기 위해 스파크 애플리케이션에 허용되는 CPU 코어 합계 (기본값: 사용가능한 전체); 워커에서만</td>
  </tr>
  <tr>
    <td><code>-m MEM</code>, <code>--memory MEM</code></td>
    <td>Total amount of memory to allow Spark applications to use on the machine, in a format like 1000M or 2G (default: your machine's total RAM minus 1 GB); only on worker
    머신에서 사용하기 위해 스파크 애플리케이션에 허용되는 메모리 합계, 1000M 또는 2G와 같은 형태로 사용한다. (기본값: 머신의 RAM 마이너스 1GB); 워커에서만</td>
  </tr>
  <tr>
    <td><code>-d DIR</code>, <code>--work-dir DIR</code></td>
    <td>Directory to use for scratch space and job output logs (default: SPARK_HOME/work); only on worker
    scratch 공간과 job 아웃풋 로그에 대해 사용하기 위한 디렉토리 (기본값: SPARK_HOME/work); 워커에서만</td>
  </tr>
  <tr>
    <td><code>--properties-file FILE</code></td>
    <td>Path to a custom Spark properties file to load (default: conf/spark-defaults.conf)
    커스텀 스파크 속성 파일에 대한 경로 (기본값: conf/spark-defaults.conf)</td>
  </tr>
</table>


# Cluster Launch Scripts
# 클러스터 실행 스크립트

To launch a Spark standalone cluster with the launch scripts, you should create a file called conf/slaves in your Spark directory,
which must contain the hostnames of all the machines where you intend to start Spark workers, one per line.
If conf/slaves does not exist, the launch scripts defaults to a single machine (localhost), which is useful for testing.
Note, the master machine accesses each of the worker machines via ssh. By default, ssh is run in parallel and requires password-less (using a private key) access to be setup.
If you do not have a password-less setup, you can set the environment variable SPARK_SSH_FOREGROUND and serially provide a password for each worker.

실행 스크립트로 스파크 스탠드얼론을 실행하기 위해, 스파크 디렉토리 하위에 conf/slaves 파일을 생성한다, 스파크 워커를 실행하려는 모든 머신에 한 줄에 하나씩 호스트 이름을 포함시켜야 한다.
만약 conf/slaves 파일이 존재하지 않으면, 실행 스크립트는 싱글머신을 (localhost) 기본으로 실행한다, 이 것은 테스트에 유용하다.
주의, 마스터 머신은 ssh를 통해 워커 머신들에 접근을 한다. 기본적으로 ssh는 병렬로 실행되고, 설정을 위해 비밀번호 없이(private key 사용) 접근 하도록 해야 한다.
만약 비밀번호 없이 접근하도록 구성되어 있지 않다면, SPARK_SSH_FOREGROUND 환경변수를 설정할 수 있고 각 워커에 대한 직렬 비밀번호를 제공해야 한다.


Once you've set up this file, you can launch or stop your cluster with the following shell scripts, based on Hadoop's deploy scripts, and available in `SPARK_HOME/sbin`:

conf/slaves 파일을 설정한 후, 다음 쉘 스크립트들을 사용해서 클러스터를 실행하거나 중지할 수 있고, 하둡의 배포 스크립트 기반, 그리고 `SPARK_HOME/sbin`에서 사용할 수 있다 :

- `sbin/start-master.sh` - Starts a master instance on the machine the script is executed on.
- `sbin/start-slaves.sh` - Starts a slave instance on each machine specified in the `conf/slaves` file.
- `sbin/start-slave.sh` - Starts a slave instance on the machine the script is executed on.
- `sbin/start-all.sh` - Starts both a master and a number of slaves as described above.
- `sbin/stop-master.sh` - Stops the master that was started via the `bin/start-master.sh` script.
- `sbin/stop-slaves.sh` - Stops all slave instances on the machines specified in the `conf/slaves` file.
- `sbin/stop-all.sh` - Stops both the master and the slaves as described above.

- `sbin/start-master.sh` - 스크립트가 실행된 머신에 마스터 인스턴스를 시작한다.
- `sbin/start-slaves.sh` - `conf/slaves` 파일에 지정된 각 머신에 슬레이브 인스턴스를 시작한다.
- `sbin/start-slave.sh` - 스크립트가 실행된 머신에 슬레이브 인스턴스를 시작한다.
- `sbin/start-all.sh` - 위에 기술한 마스터와 슬레이브 인스턴스들을 시작한다.
- `sbin/stop-master.sh` - `bin/start-master.sh` 스크립트를 통해 시작된 마스터를 중지한다.
- `sbin/stop-slaves.sh` - `conf/slaves` 파일에 지정된 머신의 모든 스레이브를 중지한다.
- `sbin/stop-all.sh` - 위에 기술한 마스터와 슬레이브 인스턴스들을 중지한다.

Note that these scripts must be executed on the machine you want to run the Spark master on, not your local machine.

이 스크립트들은 로컬 머신이 아니라 실행하려는 스파크 마스터 머신에서 실행되어야 한다.

You can optionally configure the cluster further by setting environment variables in `conf/spark-env.sh`. Create this file by starting with the `conf/spark-env.sh.template`, and _copy it to all your worker machines_ for the settings to take effect. The following settings are available:

`conf/spark-env.sh` 안에 환경변수를 설정하여 클러스터를 옵션으로 설정할 수 있다. `conf/spark-env.sh.template`을 사용하여 파일을 사용하여 생성하고 모든 워커 머신에 적용하기 위해 복사한다. 다음 설정들을 사용할 수 있다:  

<table class="table">
  <tr><th style="width:21%">Environment Variable</th><th>Meaning</th></tr>
  <tr>
    <td><code>SPARK_MASTER_HOST</code></td>
    <td>Bind the master to a specific hostname or IP address, for example a public one.
    특정 호스트명이나 IP 주소로 마스터를 바인드한다, 예를 들어 공개된 하나</td>
  </tr>
  <tr>
    <td><code>SPARK_MASTER_PORT</code></td>
    <td>Start the master on a different port (default: 7077).
    다른 포트로 마스터를 시작한다. (기본값 7077)</td>
  </tr>
  <tr>
    <td><code>SPARK_MASTER_WEBUI_PORT</code></td>
    <td>Port for the master web UI (default: 8080).
    마스터 웹 UI를 위한 포트 (기본값 8080)</td>
  </tr>
  <tr>
    <td><code>SPARK_MASTER_OPTS</code></td>
    <td>Configuration properties that apply only to the master in the form "-Dx=y" (default: none). See below for a list of possible options.
    "-Dx=y" 형태로 마스터에만 적용하는 설정 속성들 (기본값: none). 가능한 옵션들의 목록을 아래에서 확인할  수 있다.</td>
  </tr>
  <tr>
    <td><code>SPARK_LOCAL_DIRS</code></td>
    <td>
    Directory to use for "scratch" space in Spark, including map output files and RDDs that get
    stored on disk. This should be on a fast, local disk in your system. It can also be a
    comma-separated list of multiple directories on different disks.
    스파크에서 "scratch" 공간에 대해 사용하기 위한 디렉토리, map 아웃풋 파일들과 디스크에 저장된 RDDs를 포함한다. 로컬 디스크에 두어야 빠르다. 또한 다른 디스크들의 여러 디렉토리 목록을 콤마 구분자로 지정할 수 있다.
    </td>
  </tr>
  <tr>
    <td><code>SPARK_WORKER_CORES</code></td>
    <td>Total number of cores to allow Spark applications to use on the machine (default: all available cores).
    머신에서 사용하기 위해 스파크 애플리케이션에 허용되는 코어 합계 (기본값: 사용가능한 코어 전체)</td>
  </tr>
  <tr>
    <td><code>SPARK_WORKER_MEMORY</code></td>
    <td>Total amount of memory to allow Spark applications to use on the machine, e.g. <code>1000m</code>, <code>2g</code> (default: total memory minus 1 GB); note that each application's <i>individual</i> memory is configured using its <code>spark.executor.memory</code> property.
    머신에서 사용하기 위해 스파크 애플리케이션에 허용되는 메모리 합계. 예를들어 <code>1000m</code>, <code>2G</code> (기본값: 합계 메모리 마이너스 1GB); 각 애플리케이션의 <i>개별적</i> 메모리는 <code>spark.executor.memory</code> 속성을 사용하여 설정된다.</td>
  </tr>
  <tr>
    <td><code>SPARK_WORKER_PORT</code></td>
    <td>Start the Spark worker on a specific port (default: random).</td>
  </tr>
  <tr>
    <td><code>SPARK_WORKER_WEBUI_PORT</code></td>
    <td>Port for the worker web UI (default: 8081).</td>
  </tr>
  <tr>
    <td><code>SPARK_WORKER_DIR</code></td>
    <td>Directory to run applications in, which will include both logs and scratch space (default: SPARK_HOME/work).</td>
  </tr>
  <tr>
    <td><code>SPARK_WORKER_OPTS</code></td>
    <td>Configuration properties that apply only to the worker in the form "-Dx=y" (default: none). See below for a list of possible options.</td>
  </tr>
  <tr>
    <td><code>SPARK_DAEMON_MEMORY</code></td>
    <td>Memory to allocate to the Spark master and worker daemons themselves (default: 1g).</td>
  </tr>
  <tr>
    <td><code>SPARK_DAEMON_JAVA_OPTS</code></td>
    <td>JVM options for the Spark master and worker daemons themselves in the form "-Dx=y" (default: none).</td>
  </tr>
  <tr>
    <td><code>SPARK_PUBLIC_DNS</code></td>
    <td>The public DNS name of the Spark master and workers (default: none).</td>
  </tr>
</table>

**Note:** The launch scripts do not currently support Windows. To run a Spark cluster on Windows, start the master and workers by hand.

**주의:** 실행 스크립트는 현재 윈도우에서 지원되지 않는다. 윈도우에서 스파크 클러스터를 실행하기 위해 마스터와 워커를 수동으로 시작해야 한다.

SPARK_MASTER_OPTS supports the following system properties:

SPARK_MASTER_OPTS는 다음의 시스템 속성들을 제공한다:

<table class="table">
<tr><th>Property Name</th><th>Default</th><th>Meaning</th></tr>
<tr>
  <td><code>spark.deploy.retainedApplications</code></td>
  <td>200</td>
  <td>
    The maximum number of completed applications to display. Older applications will be dropped from the UI to maintain this limit.<br/>
  </td>
</tr>
<tr>
  <td><code>spark.deploy.retainedDrivers</code></td>
  <td>200</td>
  <td>
   The maximum number of completed drivers to display. Older drivers will be dropped from the UI to maintain this limit.<br/>
  </td>
</tr>
<tr>
  <td><code>spark.deploy.spreadOut</code></td>
  <td>true</td>
  <td>
    Whether the standalone cluster manager should spread applications out across nodes or try
    to consolidate them onto as few nodes as possible. Spreading out is usually better for
    data locality in HDFS, but consolidating is more efficient for compute-intensive workloads. <br/>
  </td>
</tr>
<tr>
  <td><code>spark.deploy.defaultCores</code></td>
  <td>(infinite)</td>
  <td>
    Default number of cores to give to applications in Spark's standalone mode if they don't
    set <code>spark.cores.max</code>. If not set, applications always get all available
    cores unless they configure <code>spark.cores.max</code> themselves.
    Set this lower on a shared cluster to prevent users from grabbing
    the whole cluster by default. <br/>
  </td>
</tr>
<tr>
  <td><code>spark.worker.timeout</code></td>
  <td>60</td>
  <td>
    Number of seconds after which the standalone deploy master considers a worker lost if it
    receives no heartbeats.
  </td>
</tr>
</table>

SPARK_WORKER_OPTS supports the following system properties:

SPARK_WORKER_OPTS는 다음의 시스템 속성들을 제공한다:

<table class="table">
<tr><th>Property Name</th><th>Default</th><th>Meaning</th></tr>
<tr>
  <td><code>spark.worker.cleanup.enabled</code></td>
  <td>false</td>
  <td>
    Enable periodic cleanup of worker / application directories.  Note that this only affects standalone
    mode, as YARN works differently. Only the directories of stopped applications are cleaned up.
  </td>
</tr>
<tr>
  <td><code>spark.worker.cleanup.interval</code></td>
  <td>1800 (30 minutes)</td>
  <td>
    Controls the interval, in seconds, at which the worker cleans up old application work dirs
    on the local machine.
  </td>
</tr>
<tr>
  <td><code>spark.worker.cleanup.appDataTtl</code></td>
  <td>7 * 24 * 3600 (7 days)</td>
  <td>
    The number of seconds to retain application work directories on each worker.  This is a Time To Live
    and should depend on the amount of available disk space you have.  Application logs and jars are
    downloaded to each application work dir.  Over time, the work dirs can quickly fill up disk space,
    especially if you run jobs very frequently.
  </td>
</tr>
</table>

# Connecting an Application to the Cluster

# 애플리케이션에서 클러스터 연결

To run an application on the Spark cluster, simply pass the `spark://IP:PORT` URL of the master as to the [`SparkContext`
constructor](programming-guide.html#initializing-spark).

스파크 클러스터에서 애플리케이션을 실행하기 위해 마스터의 `spark://IP:PORT` URL을 [`SparkContext` 생성자](programming-guide.html#initializing-spark)로 보낸다.

To run an interactive Spark shell against the cluster, run the following command:

해당 클러스터에 대화형 스파크 쉘을 실행하려면 다음의 명령을 실행한다:

    ./bin/spark-shell --master spark://IP:PORT

You can also pass an option `--total-executor-cores <numCores>` to control the number of cores that spark-shell uses on the cluster.

클러스터에서 스파크 쉘이 사용하는 코어를 제어하기 위해 `--totoal-executor-cores <numCores>` 옵션을 보낼 수 있다.

# Launching Spark Applications

The [`spark-submit` script](submitting-applications.html) provides the most straightforward way to
submit a compiled Spark application to the cluster. For standalone clusters, Spark currently
supports two deploy modes. In `client` mode, the driver is launched in the same process as the
client that submits the application. In `cluster` mode, however, the driver is launched from one
of the Worker processes inside the cluster, and the client process exits as soon as it fulfills
its responsibility of submitting the application without waiting for the application to finish.

If your application is launched through Spark submit, then the application jar is automatically
distributed to all worker nodes. For any additional jars that your application depends on, you
should specify them through the `--jars` flag using comma as a delimiter (e.g. `--jars jar1,jar2`).
To control the application's configuration or execution environment, see
[Spark Configuration](configuration.html).

Additionally, standalone `cluster` mode supports restarting your application automatically if it
exited with non-zero exit code. To use this feature, you may pass in the `--supervise` flag to
`spark-submit` when launching your application. Then, if you wish to kill an application that is
failing repeatedly, you may do so through:

    ./bin/spark-class org.apache.spark.deploy.Client kill <master url> <driver ID>

You can find the driver ID through the standalone Master web UI at `http://<master url>:8080`.

# Resource Scheduling

The standalone cluster mode currently only supports a simple FIFO scheduler across applications.
However, to allow multiple concurrent users, you can control the maximum number of resources each
application will use.
By default, it will acquire *all* cores in the cluster, which only makes sense if you just run one
application at a time. You can cap the number of cores by setting `spark.cores.max` in your
[SparkConf](configuration.html#spark-properties). For example:

{% highlight scala %}
val conf = new SparkConf()
             .setMaster(...)
             .setAppName(...)
             .set("spark.cores.max", "10")
val sc = new SparkContext(conf)
{% endhighlight %}

In addition, you can configure `spark.deploy.defaultCores` on the cluster master process to change the
default for applications that don't set `spark.cores.max` to something less than infinite.
Do this by adding the following to `conf/spark-env.sh`:

{% highlight bash %}
export SPARK_MASTER_OPTS="-Dspark.deploy.defaultCores=<value>"
{% endhighlight %}

This is useful on shared clusters where users might not have configured a maximum number of cores
individually.

# Monitoring and Logging

Spark's standalone mode offers a web-based user interface to monitor the cluster. The master and each worker has its own web UI that shows cluster and job statistics. By default you can access the web UI for the master at port 8080. The port can be changed either in the configuration file or via command-line options.

In addition, detailed log output for each job is also written to the work directory of each slave node (`SPARK_HOME/work` by default). You will see two files for each job, `stdout` and `stderr`, with all output it wrote to its console.


# Running Alongside Hadoop

You can run Spark alongside your existing Hadoop cluster by just launching it as a separate service on the same machines. To access Hadoop data from Spark, just use a hdfs:// URL (typically `hdfs://<namenode>:9000/path`, but you can find the right URL on your Hadoop Namenode's web UI). Alternatively, you can set up a separate cluster for Spark, and still have it access HDFS over the network; this will be slower than disk-local access, but may not be a concern if you are still running in the same local area network (e.g. you place a few Spark machines on each rack that you have Hadoop on).


# Configuring Ports for Network Security

Spark makes heavy use of the network, and some environments have strict requirements for using
tight firewall settings. For a complete list of ports to configure, see the
[security page](security.html#configuring-ports-for-network-security).

# High Availability

By default, standalone scheduling clusters are resilient to Worker failures (insofar as Spark itself is resilient to losing work by moving it to other workers). However, the scheduler uses a Master to make scheduling decisions, and this (by default) creates a single point of failure: if the Master crashes, no new applications can be created. In order to circumvent this, we have two high availability schemes, detailed below.

## Standby Masters with ZooKeeper

**Overview**

Utilizing ZooKeeper to provide leader election and some state storage, you can launch multiple Masters in your cluster connected to the same ZooKeeper instance. One will be elected "leader" and the others will remain in standby mode. If the current leader dies, another Master will be elected, recover the old Master's state, and then resume scheduling. The entire recovery process (from the time the first leader goes down) should take between 1 and 2 minutes. Note that this delay only affects scheduling _new_ applications -- applications that were already running during Master failover are unaffected.

Learn more about getting started with ZooKeeper [here](http://zookeeper.apache.org/doc/trunk/zookeeperStarted.html).

**Configuration**

In order to enable this recovery mode, you can set SPARK_DAEMON_JAVA_OPTS in spark-env by configuring `spark.deploy.recoveryMode` and related spark.deploy.zookeeper.* configurations.
For more information about these configurations please refer to the configurations (doc)[configurations.html#deploy]

Possible gotcha: If you have multiple Masters in your cluster but fail to correctly configure the Masters to use ZooKeeper, the Masters will fail to discover each other and think they're all leaders. This will not lead to a healthy cluster state (as all Masters will schedule independently).

**Details**

After you have a ZooKeeper cluster set up, enabling high availability is straightforward. Simply start multiple Master processes on different nodes with the same ZooKeeper configuration (ZooKeeper URL and directory). Masters can be added and removed at any time.

In order to schedule new applications or add Workers to the cluster, they need to know the IP address of the current leader. This can be accomplished by simply passing in a list of Masters where you used to pass in a single one. For example, you might start your SparkContext pointing to ``spark://host1:port1,host2:port2``. This would cause your SparkContext to try registering with both Masters -- if ``host1`` goes down, this configuration would still be correct as we'd find the new leader, ``host2``.

There's an important distinction to be made between "registering with a Master" and normal operation. When starting up, an application or Worker needs to be able to find and register with the current lead Master. Once it successfully registers, though, it is "in the system" (i.e., stored in ZooKeeper). If failover occurs, the new leader will contact all previously registered applications and Workers to inform them of the change in leadership, so they need not even have known of the existence of the new Master at startup.

Due to this property, new Masters can be created at any time, and the only thing you need to worry about is that _new_ applications and Workers can find it to register with in case it becomes the leader. Once registered, you're taken care of.

## Single-Node Recovery with Local File System

**Overview**

ZooKeeper is the best way to go for production-level high availability, but if you just want to be able to restart the Master if it goes down, FILESYSTEM mode can take care of it. When applications and Workers register, they have enough state written to the provided directory so that they can be recovered upon a restart of the Master process.

**Configuration**

In order to enable this recovery mode, you can set SPARK_DAEMON_JAVA_OPTS in spark-env using this configuration:

<table class="table">
  <tr><th style="width:21%">System property</th><th>Meaning</th></tr>
  <tr>
    <td><code>spark.deploy.recoveryMode</code></td>
    <td>Set to FILESYSTEM to enable single-node recovery mode (default: NONE).</td>
  </tr>
  <tr>
    <td><code>spark.deploy.recoveryDirectory</code></td>
    <td>The directory in which Spark will store recovery state, accessible from the Master's perspective.</td>
  </tr>
</table>

**Details**

* This solution can be used in tandem with a process monitor/manager like [monit](http://mmonit.com/monit/), or just to enable manual recovery via restart.
* While filesystem recovery seems straightforwardly better than not doing any recovery at all, this mode may be suboptimal for certain development or experimental purposes. In particular, killing a master via stop-master.sh does not clean up its recovery state, so whenever you start a new Master, it will enter recovery mode. This could increase the startup time by up to 1 minute if it needs to wait for all previously-registered Workers/clients to timeout.
* While it's not officially supported, you could mount an NFS directory as the recovery directory. If the original Master node dies completely, you could then start a Master on a different node, which would correctly recover all previously registered Workers/applications (equivalent to ZooKeeper recovery). Future applications will have to be able to find the new Master, however, in order to register.
