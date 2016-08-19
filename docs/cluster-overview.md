---
layout: global
title: Cluster Mode Overview
---

This document gives a short overview of how Spark runs on clusters, to make it easier to understand
the components involved. Read through the [application submission guide](submitting-applications.html)
to learn about launching applications on a cluster.

이 문서는 클러스터에서 스파크를 어떻게 실행하는지 짧은 개념을 설명하고, 컴포넌트들에 대한 이해를 쉽게 할 수 있도록 만든다. 클러스터에서 애플리케이션을 실행하기 위해 알고 싶다면 [애플리케이션 제출 안내](submitting-application.html)를 참고하라. 

# Components

# 컴포넌트

Spark applications run as independent sets of processes on a cluster, coordinated by the `SparkContext`
object in your main program (called the _driver program_).

스파크 애플리케이션은 클러스터에 독립적인 프로세스의 셋으로 실행된다. (_driver program_ 이라는) 메인 프로그램 안에서 `SparkContext`에 의해 처리된다.

Specifically, to run on a cluster, the SparkContext can connect to several types of _cluster managers_
(either Spark's own standalone cluster manager, Mesos or YARN), which allocate resources across
applications. Once connected, Spark acquires *executors* on nodes in the cluster, which are
processes that run computations and store data for your application.
Next, it sends your application code (defined by JAR or Python files passed to SparkContext) to
the executors. Finally, SparkContext sends *tasks* to the executors to run.

명확하게, 클러스터에서 실행하기 위해, SparkContext는 몇몇 _cluster managers_ 의 유형(스파크 자신의 스탠드얼롬 클러스터 매니저, Mesos나 YARN)에 연결할 수 있고, 애플리케이션들에 자원 할당을 한다. 연결되면, 스파크는 클러스터의 노드들에 있는 *executors*를 얻게 되는데, 애플리케이션에 대한 데이터를 처리하고 저장하는 프로세스들이다.
다음으로, executors로 애플리케이션 코드(정의된 JAR 또는 Python 파일을 SparkContext로 보낸다)를 보낸다. 최종적으로, SparkContext는 실행하기 위한 *tasks* 를 executors로 보낸다.

<p style="text-align: center;">
  <img src="img/cluster-overview.png" title="스파크 클러스터 컴포넌트" alt="스파크 클러스터 컴포넌트" />
</p>

There are several useful things to note about this architecture:

이 아키텍처에 대해 주의해야 할 유용한 몇 가지 것들이 있다:


1. Each application gets its own executor processes, which stay up for the duration of the whole
   application and run tasks in multiple threads. This has the benefit of isolating applications
   from each other, on both the scheduling side (each driver schedules its own tasks) and executor
   side (tasks from different applications run in different JVMs). However, it also means that
   data cannot be shared across different Spark applications (instances of SparkContext) without
   writing it to an external storage system.
   
   각 애플리케이션은 자신의 executor 프로세스들을 얻고, 애플리케이션 전체 기간 동안 깨어있고 멀티 쓰레드로 태스크를 실행한다. 이 것은 스케줄링 측면(각 드라이버는 자신의 tasks를 스케줄링한다)과 executor 측면(서로 다른 애플리케이션들로부터 tasks는 다른 JVM에서 실행된다)에서도 각각 서로로 부터 애플리케이션을 격리시키는 잇점을 갖는다. 하지만, 그것은 외부 스토리지 시스템에 쓰지 않고서는 서로 다른 스파크 애플리케이션들간에 데이터를 공유할 수 없다는 의미이기도 하다.  
   
2. Spark is agnostic to the underlying cluster manager. As long as it can acquire executor
   processes, and these communicate with each other, it is relatively easy to run it even on a
   cluster manager that also supports other applications (e.g. Mesos/YARN).
   
   스파크는 근본적인 클러스터 매니저에 얽메인다. executor 프로세스를 얻어내고 서로간에 통신할 수 있는 것과 마찬가지로, 다른 애플리케이션(예를 들어 Mesos/YARN)을 지원하는 클러스터 매니저에서 실행하는 것 역시 상대적으로 쉽다. 
   
3. The driver program must listen for and accept incoming connections from its executors throughout
   its lifetime (e.g., see [spark.driver.port in the network config
   section](configuration.html#networking)). As such, the driver program must be network
   addressable from the worker nodes.
   
   드라이버 프로그램은 수명 전반에 걸쳐 executors로 부터 들어오는 연결을 귀기울이고 수락해야만 한다. (예를 들어, [네트워크 설정 부분에서 spark.driver.port](configuration.html#networking)). 드라이버 프로그램은 워커 노드들로 부터 네트워크 주소 지정이 가능해야 한다.
   
4. Because the driver schedules tasks on the cluster, it should be run close to the worker
   nodes, preferably on the same local area network. If you'd like to send requests to the
   cluster remotely, it's better to open an RPC to the driver and have it submit operations
   from nearby than to run a driver far away from the worker nodes.
   
   드라이버가 클러스터에서 tasks를 스케줄링하기 때문에, 동일한 로컬 네트워크 영역에서 워커 노드들에 가까이에서 실행하는 것이 바람직하다. 만약 원격의 클러스터로 요청을 보내려면 드라이버로 RPC를 열고, 워커 노드들로 부터 멀리 있는 드라이버를 실행하는 것보다 가까운 곳으로 부터 연산을 하도록 제출하는 것이 좋다. 

# Cluster Manager Types

# 클러스터 매니저 유형들

The system currently supports three cluster managers:

현재 시스템에서 세 가지 클러스터 매니저를 지원한다:

* [Standalone](spark-standalone.html) -- a simple cluster manager included with Spark that makes it
  easy to set up a cluster.
  
  [스탠드얼론](spark-standalone.html) -- 스파크 클러스터 셋업을 위한 간단한 클러스터 매니저
  
* [Apache Mesos](running-on-mesos.html) -- a general cluster manager that can also run Hadoop MapReduce
  and service applications.
  
  [아파치 Mesos](running-on-mesos.html) -- 하둡 맵리듀스를 실행할 수 있고 애플리케이션을 서비스할 수 있는 일반적인 클러스터 매니저
  
* [Hadoop YARN](running-on-yarn.html) -- the resource manager in Hadoop 2.

  [하둡 YARN](running-on-yarn.html) -- 하둡 2 안에 있는 자원 관리자


# Submitting Applications

# 애플리케이션 제출하기

Applications can be submitted to a cluster of any type using the `spark-submit` script.
The [application submission guide](submitting-applications.html) describes how to do this.

애플리케이션은 `spark-submit` 스크립트를 사용하여 특정 유형의 클러스터로 제출될 수 있다.
[애플리케이션 제출 가이드](submitting-application.html)는 어떻게 제출하는지 방법을 설명한다.

# Monitoring

# 모니터링

Each driver program has a web UI, typically on port 4040, that displays information about running
tasks, executors, and storage usage. Simply go to `http://<driver-node>:4040` in a web browser to
access this UI. The [monitoring guide](monitoring.html) also describes other monitoring options.

각 드라이버 프로그램은 웹 UI를 갖는다, 전형적으로 4040 포트를 사용한다, 실행중인 tasks, executors 그리고 스토리지 사용량에 대한 정보를 보여준다. 간단하게 웹 브라우저에서 `http://<driver-node>:4040`를 접근할 수 있다. [모니터링 가이드](monitoring.html)은 다른 모니터링 옵션들을 설명한다.

# Job Scheduling

# 잡 스케줄링

Spark gives control over resource allocation both _across_ applications (at the level of the cluster
manager) and _within_ applications (if multiple computations are happening on the same SparkContext).
The [job scheduling overview](job-scheduling.html) describes this in more detail.

스파크는 _across_ 애플리케이션들(클러스터 매니저 수준에서)과 _within_ 애플리케이션들(다중 연산이 같은 SparkContext에서 일어난다면)에 대해 자원 할당을 통해 제어할 수 있다.
[잡 스케줄링 개요](job-scheduling.html)은 자세한 내용을 설명한다.

# Glossary

# 용어사전

The following table summarizes terms you'll see used to refer to cluster concepts:

다음 테이블은 클러스터 개념을 참조하기 위해 사용되는 용어들을 요약한다:

<table class="table">
  <thead>
    <tr><th style="width: 130px;">Term</th><th>Meaning</th></tr>
  </thead>
  <tbody>
    <tr>
      <td>Application</td>
      <td>User program built on Spark. Consists of a <em>driver program</em> and <em>executors</em> on the cluster.</td>
    </tr>
    <tr>
      <td>Application jar</td>
      <td>
        A jar containing the user's Spark application. In some cases users will want to create
        an "uber jar" containing their application along with its dependencies. The user's jar
        should never include Hadoop or Spark libraries, however, these will be added at runtime.
      </td>
    </tr>
    <tr>
      <td>Driver program</td>
      <td>The process running the main() function of the application and creating the SparkContext</td>
    </tr>
    <tr>
      <td>Cluster manager</td>
      <td>An external service for acquiring resources on the cluster (e.g. standalone manager, Mesos, YARN)</td>
    </tr>
    <tr>
      <td>Deploy mode</td>
      <td>Distinguishes where the driver process runs. In "cluster" mode, the framework launches
        the driver inside of the cluster. In "client" mode, the submitter launches the driver
        outside of the cluster.</td>
    </tr>
    <tr>
      <td>Worker node</td>
      <td>Any node that can run application code in the cluster</td>
    </tr>
    <tr>
      <td>Executor</td>
      <td>A process launched for an application on a worker node, that runs tasks and keeps data in memory
        or disk storage across them. Each application has its own executors.</td>
    </tr>
    <tr>
      <td>Task</td>
      <td>A unit of work that will be sent to one executor</td>
    </tr>
    <tr>
      <td>Job</td>
      <td>A parallel computation consisting of multiple tasks that gets spawned in response to a Spark action
        (e.g. <code>save</code>, <code>collect</code>); you'll see this term used in the driver's logs.</td>
    </tr>
    <tr>
      <td>Stage</td>
      <td>Each job gets divided into smaller sets of tasks called <em>stages</em> that depend on each other
        (similar to the map and reduce stages in MapReduce); you'll see this term used in the driver's logs.</td>
    </tr>
  </tbody>
</table>

<table class="table">
  <thead>
    <tr><th style="width: 130px;">용어</th><th>의미</th></tr>
  </thead>
  <tbody>
    <tr>
      <td>애플리케이션</td>
      <td>스파크에서 빌드된 사용자 프로그램. 클러스터에서 <em>드라이버 프로그램</em>과 <em>executors</em>으로 구성된다.</td>
    </tr>
    <tr>
      <td>애플리케이션 jar</td>
      <td>
        jar는 사용자의 스파크 애플리케이션을 포함하고 있다.
        어떤 사용자들은 종속성과 함께 그들의 애플리케이션을 포함하고 있는 "uber jar"를 생성하고 싶어하는 경우도 있다.
        하둡이나 스파크 라이브러리는 사용자의 jar에 포함하지 않고, 실행타임에 추가될 것이다.</td>
    </tr>
    <tr>
      <td>드라이버 프로그램</td>
      <td>SparkContext를 생성하고 애플리케이션의 메인 함수를 실행하는 프로세스</td>
    </tr>
    <tr>
      <td>클러스터 매니저</td>
      <td>클러스터(예를 들어 스탠드얼론 매니저, Mesos, YARN)에서 자원을 얻기 위한 외부 서비스</td>
    </tr>
    <tr>
      <td>배포 모드</td>
      <td>실행하는 드라이버 프로세스를 구별한다.
        "cluster" 모드에서는 프레임워크가 클러스터의 내부에서 드라이버를 시작한다.
        "client" 모드에서는 제출자가 클러스터의 외부에서 드라이버를 시작한다.</td>
    </tr>
    <tr>
      <td>워커 노드</td>
      <td>클러스터에서 애플리케이션을 실행 할 수 있는 특정 노드</td>
    </tr>
    <tr>
      <td>집행자(Executor)</td>
      <td>하나의 프로세스는 워커노드에서 애플리케이션을 위해 시작된다. tasks를 시작하고 메모리나 디스크 스토리지에 데이터를 보관한다. 각 애플리케이션은 자신의 집행자를 갖는다.</td>
    </tr>
    <tr>
      <td>작업(Task)</td>
      <td>하나의 집행자로 보낼 하나의 작업 단위</td>
    </tr>
    <tr>
      <td>일(Job)</td>
      <td>스파크 액션에 대한 응답으로 양산되는 여러 개의 작업으로 구성된 병렬 연산 (예를 들어 <code>save</code>, <code>collect</code>) 드라이버의 로그에서 이 용어를 보게 될 것이다.</td>
    </tr>
    <tr>
      <td>단계(Stage)</td>
      <td>각 job은 서로 종속적인 <em>stages</em>라는 더 작은 작업들(tasks)의 세트로 구분된다. (MapReduce안에 맵과 리듀스 단계로 구분되는 것처럼 비슷하다) 드라이버 로그에서 이 용어를 보게 될 것이다.</td>
    </tr>
  </tbody>
</table>
