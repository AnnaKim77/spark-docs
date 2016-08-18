#Apache Spark 2 번역 참여 방법

어서 오세요~ 환영합니다. 스파크2 번역에 참여하시는 분들을 위한 소개입니다.
기본적으로 https://spark.apache.org/docs/latest/ 문서에 걸려있는 모든 링크에 대해 번역 하는 것을 목적으로 합니다.
사전 지식으로 git, github 사용법을 이해하셔야 합니다. 마크다운(md) 문서 작성에 대해서도 이해가 필요하지만 실제 md 파일을 열어보시면 직관적으로 바로 아실 수 있기 때문에 따로 공부하지 않으셔도 될 것 같습니다.

### 슬랙을 사용합니다.

https://sparkdoc.slack.com
슬랙에 초대를 원하시는 분은 gmail 주소를 oopchoi@gmail.com 으로 성함과 함께 보내주세요. 

git, github 공부 추천 링크 : http://nolboo.kim/blog/2013/10/06/github-for-beginner/

### 1. https://github.com/oopchoi/spark-docs 접속해서 Fork
### 2. 자신의 원격 깃헙에서 로컬로 클론(체크아웃)
### 3. docs 경로에 있는 md 파일을 번역하고 커밋, 푸쉬
### 4. 풀리퀘스트

주의) 여러 사람이 공동으로 작업하기 때문에 같은 파일을 수정하면 충돌이 납니다. 따라서 반드시 이슈를 등록 하고 진행중인 번역 파일에 대해 작성해주셔야 합니다.

이슈 등록 하는 방법 : https://medium.com/@oopchoi/%EC%9D%B4%EC%8A%88-%EB%93%B1%EB%A1%9D-%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95-7ff328584fb7#.ikbghwtt7

번역을 마치고 풀리퀘스트로 머지가 완료되면 이슈를 close 하시면 됩니다.

문서 빌드 방법 : 현재 API docs 부분을 빌드하면 에러가 발생하므로

`$ SKIP_API=1 jekyll build`

를 사용해서 빌드하시면 기본 문서들은 생성이 잘 됩니다.

빌드 방법 : https://medium.com/@oopchoi/%EB%A7%A5os%EC%97%90%EC%84%9C-spark-docs-%EB%B9%8C%EB%93%9C%ED%95%98%EA%B8%B0-d8794ea09c3e#.7mgtxkf08

# 빌드 완료버전 

번역 최종 버전 보기
http://edd.exem-oss.org/docs/spark2/_site/index.html 

# Apache Spark

Spark is a fast and general cluster computing system for Big Data. It provides
high-level APIs in Scala, Java, Python, and R, and an optimized engine that
supports general computation graphs for data analysis. It also supports a
rich set of higher-level tools including Spark SQL for SQL and DataFrames,
MLlib for machine learning, GraphX for graph processing,
and Spark Streaming for stream processing.

<http://spark.apache.org/>


## Online Documentation

You can find the latest Spark documentation, including a programming
guide, on the [project web page](http://spark.apache.org/documentation.html)
and [project wiki](https://cwiki.apache.org/confluence/display/SPARK).
This README file only contains basic setup instructions.

## Building Spark

Spark is built using [Apache Maven](http://maven.apache.org/).
To build Spark and its example programs, run:

    build/mvn -DskipTests clean package

(You do not need to do this if you downloaded a pre-built package.)

You can build Spark using more than one thread by using the -T option with Maven, see ["Parallel builds in Maven 3"](https://cwiki.apache.org/confluence/display/MAVEN/Parallel+builds+in+Maven+3).
More detailed documentation is available from the project site, at
["Building Spark"](http://spark.apache.org/docs/latest/building-spark.html).
For developing Spark using an IDE, see [Eclipse](https://cwiki.apache.org/confluence/display/SPARK/Useful+Developer+Tools#UsefulDeveloperTools-Eclipse)
and [IntelliJ](https://cwiki.apache.org/confluence/display/SPARK/Useful+Developer+Tools#UsefulDeveloperTools-IntelliJ).

## Interactive Scala Shell

The easiest way to start using Spark is through the Scala shell:

    ./bin/spark-shell

Try the following command, which should return 1000:

    scala> sc.parallelize(1 to 1000).count()

## Interactive Python Shell

Alternatively, if you prefer Python, you can use the Python shell:

    ./bin/pyspark

And run the following command, which should also return 1000:

    >>> sc.parallelize(range(1000)).count()

## Example Programs

Spark also comes with several sample programs in the `examples` directory.
To run one of them, use `./bin/run-example <class> [params]`. For example:

    ./bin/run-example SparkPi

will run the Pi example locally.

You can set the MASTER environment variable when running examples to submit
examples to a cluster. This can be a mesos:// or spark:// URL,
"yarn" to run on YARN, and "local" to run
locally with one thread, or "local[N]" to run locally with N threads. You
can also use an abbreviated class name if the class is in the `examples`
package. For instance:

    MASTER=spark://host:7077 ./bin/run-example SparkPi

Many of the example programs print usage help if no params are given.

## Running Tests

Testing first requires [building Spark](#building-spark). Once Spark is built, tests
can be run using:

    ./dev/run-tests

Please see the guidance on how to
[run tests for a module, or individual tests](https://cwiki.apache.org/confluence/display/SPARK/Useful+Developer+Tools).

## A Note About Hadoop Versions

Spark uses the Hadoop core library to talk to HDFS and other Hadoop-supported
storage systems. Because the protocols have changed in different versions of
Hadoop, you must build Spark against the same version that your cluster runs.

Please refer to the build documentation at
["Specifying the Hadoop Version"](http://spark.apache.org/docs/latest/building-spark.html#specifying-the-hadoop-version)
for detailed guidance on building for a particular distribution of Hadoop, including
building for particular Hive and Hive Thriftserver distributions.

## Configuration

Please refer to the [Configuration Guide](http://spark.apache.org/docs/latest/configuration.html)
in the online documentation for an overview on how to configure Spark.
