---
layout: global
title: "MLlib: Main Guide"
displayTitle: "기계학습 라이브러리(MLlib) 가이드"
---

MLlib는 스파크의 기계학습 라이브러리입니다. 이 라이브러리의 목표는 쉽고 확장가능한 실용적인 기계 학습을 하는 것입니다. 고수준에서 다음과 같은 툴을 제공합니다.

* ML 알고리즘 : 분류, 회귀, 클러스터링, 협업 필터링과 같은 학습 알고리즘
* 특성화 : 특성 추출, 변환, 차원 축소, 셀렉션
* 파이프라인 : 구성을 위한 툴, 평가, ML 파이프라인의 조율
* 지속성 : 알고리즘, 모델, 파이프라인의 저장 및 로드 
* 유틸리티 : 선형 대수, 통계, 데이터 핸들링 등.

# 공지사항 : DataFrame 기반 API가 주 API 입니다.

**MLlib Rdd 기반 API는 현재 정비모드 입니다.**
Spark 2.0에서는 `spark.mllib` 패키지의 RDD 기반 API는 정비모드이며 Spark의 주된 기계학습 API는 `spark.ml`패키지에서 제공하는 DataFrame 기반 API 입니다.


*어떤 영향이 있을까요?*

* MLlib는 버그를 수정한 `spark.mllib` 에서 RDD 기반 API를 계속 지원 할 것입니다.
* MLlib는 RDD 기반 API에 새로운 feature를 추가 하지 않을 것입니다.
* Spark 2.x 릴리즈에서는 MLlib는 DataFrame 기반 API에 RDD 기반 API와 비슷한 수준의 feature까지 이를 수 있는 feature를 추가할 것입니다.
* feature가 비슷한 수준에 도달한 후(대략 Spark 2.2), RDD 기반 API는 사용되지 않을 것입니다.
* Spark 3.0 에서 RDD 기반 API는 없어질 것입니다.


*MLlib는 왜 DataFrame 기반 API로 바뀌는 것일까요?*

* DataFrame은 RDD에 비해서 사용 친화적인 API를 제공합니다. DataFrame Spark Datasource, SQL/DataFrame 쿼리, Tungsten 과 Catalyst 최적화, 언어별 획일적인 API은등 많은 이점이 있습니다.
* MLlib의 DataFrame 기반 API는 다수의 언어 및 ML 알고리즘을 아우르는 획일화된 API를 제공합니다.
* DataFrame은 ML 파이프라인, 특히 특성 변환을 용이하게 합니다. 자세한 사항은 [Pipelines guide](ml-pipeline.html) 을 참고하세요.


# 의존성

MLlib는 수치적 처리 최적화를 위한 [netlib-java](https://github.com/fommil/netlib-java)에 의존하는 [Breeze](http://www.scalanlp.org/) 라는 선형 대수 패키지를 사용합니다.
만약 실행시간에 네이티브 라이브러리[^1]가 없다면, 경고 메세지를 받을 것이며 pure JVM이 대신 사용될 것입니다. 

런타임 독점 바이너리 라이센스 이슈 때문이라면, 우리는 기본적으로 `netlib-java`의 네이티브 프록시를 포함하지 않습니다.
`netlib-java` 구현/ Breeze를 시스템 최적화 바이너리로 사용하기 위해서는 `com.github.fommil.netlib:all:1.1.2` (혹은 스파크 구성에 `-Pnetlib-lgpl`)을 프로젝트의 의존으로 포함하고,  
플랫폼의 추가적인 설치가이드는 [netlib-java](https://github.com/fommil/netlib-java) 문서를 읽어보세용~

Python에서 MLlib를 사용할 때 [NumPy](http://www.numpy.org) 1.4 버전이나 더 최신 버전이 필요합니다.


[^1]: 시스템 최적화 네이티브의 이점 및 배경에 관해서 알고 싶다면, Sam Halliday's ScalaX talk [High Performance Linear Algebra in Scala](http://fommil.github.io/scalax14/#/)을 참고하세요.


# 마이그레이션 가이드

MLlib는 아직 개발중에 있습니다. `Experimental`/`DeveloperApi`가 붙은 API는 추후에 나올 릴리즈에서 변경되고, 마이그레이션 가이드는 릴리즈 중간중간 모든 변경사항을 기술할 것입니다.


## 1.6에서 2.0까지

### 변경된 사항

Spark 2.0에서 변경된 사항은 하단에 강조되어 있습니다.

**DataFrame 기반 API를 위한 선형 대수 클래스**

Spark의 선형 대수 의존성은 새 프로젝트인 `mllib-local`로 옮겨졌습니다.
(참고 : [SPARK-13944](https://issues.apache.org/jira/browse/SPARK-13944)).
선형 대수 클래스는 새로운 패키지 `spark.ml.linalg`에 복사되었습니다. DataFrame 기반 API인 `spark.ml`은 `spark.ml.linalg`에 의존하며, 
다양한 모형 클래스에서 주로 변경이 되었습니다.
(전체 리스트 참고 : [SPARK-14810](https://issues.apache.org/jira/browse/SPARK-14810)).

**Note:** `spark.mllib`의 RDD 기반 API는 `spark.mllib.linalg`패키지에 계속 의존합니다.

_벡터와 행렬로의 변환_

대부분의 파이프라인 구성요소들이 로딩을 위해 백워드 호환을 지원하는 동안, Spark 2.0 이전 버전에서 벡터와 행렬을 포함하는 `DataFrames`과 파이프라인
은 새로운 `spark.ml` 벡터와 행렬 타입으로 마이그레이션이 필요합니다.
`spark.mllib.linalg`에서 `spark.ml.linalg`로 `DataFrame`열 변환을 위한 유틸리티는 `spark.mllib.util.MLUtils`에서 확인할 수 있습니다.

벡터와 행렬 단일 변환을 위한 다른 유틸리티도 있습니다. `ml.linalg` 타입으로 변환하기 위한 `mllib.linalg.Vector` / `mllib.linalg.Matrix`,
`mllib.linalg`타입으로 변환하기 위한 `mllib.linalg.Vectors.fromML` / `mllib.linalg.Matrices.fromML`의 `asML`방법을 사용합니다. 


<div class="codetabs">
<div data-lang="scala"  markdown="1">

{% highlight scala %}
import org.apache.spark.mllib.util.MLUtils

// convert DataFrame columns
val convertedVecDF = MLUtils.convertVectorColumnsToML(vecDF)
val convertedMatrixDF = MLUtils.convertMatrixColumnsToML(matrixDF)
// convert a single vector or matrix
val mlVec: org.apache.spark.ml.linalg.Vector = mllibVec.asML
val mlMat: org.apache.spark.ml.linalg.Matrix = mllibMat.asML
{% endhighlight %}

더 자세한 것은 [`MLUtils` Scala docs](api/scala/index.html#org.apache.spark.mllib.util.MLUtils$)에 있습니다.
</div>

<div data-lang="java" markdown="1">

{% highlight java %}
import org.apache.spark.mllib.util.MLUtils;
import org.apache.spark.sql.Dataset;

// convert DataFrame columns
Dataset<Row> convertedVecDF = MLUtils.convertVectorColumnsToML(vecDF);
Dataset<Row> convertedMatrixDF = MLUtils.convertMatrixColumnsToML(matrixDF);
// convert a single vector or matrix
org.apache.spark.ml.linalg.Vector mlVec = mllibVec.asML();
org.apache.spark.ml.linalg.Matrix mlMat = mllibMat.asML();
{% endhighlight %}

더 자세한 것은 [`MLUtils` Java docs](api/java/org/apache/spark/mllib/util/MLUtils.html)에 있습니다.
</div>

<div data-lang="python"  markdown="1">

{% highlight python %}
from pyspark.mllib.util import MLUtils

# convert DataFrame columns
convertedVecDF = MLUtils.convertVectorColumnsToML(vecDF)
convertedMatrixDF = MLUtils.convertMatrixColumnsToML(matrixDF)
# convert a single vector or matrix
mlVec = mllibVec.asML()
mlMat = mllibMat.asML()
{% endhighlight %}

더 자세한 것은 [`MLUtils` Python docs](api/python/pyspark.mllib.html#pyspark.mllib.util.MLUtils)에 있습니다.
</div>
</div>

**Deprecated methods removed**

Several deprecated methods were removed in the `spark.mllib` and `spark.ml` packages:

* `setScoreCol` in `ml.evaluation.BinaryClassificationEvaluator`
* `weights` in `LinearRegression` and `LogisticRegression` in `spark.ml`
* `setMaxNumIterations` in `mllib.optimization.LBFGS` (marked as `DeveloperApi`)
* `treeReduce` and `treeAggregate` in `mllib.rdd.RDDFunctions` (these functions are available on `RDD`s directly, and were marked as `DeveloperApi`)
* `defaultStategy` in `mllib.tree.configuration.Strategy`
* `build` in `mllib.tree.Node`
* libsvm loaders for multiclass and load/save labeledData methods in `mllib.util.MLUtils`

A full list of breaking changes can be found at [SPARK-14810](https://issues.apache.org/jira/browse/SPARK-14810).


### 사용되지 않거나 달라진 사항

**사용되지 않는 것**

`spark.mllib`와 `spark.ml`패키지에서 사용되지 않는 사항들 :

* [SPARK-14984](https://issues.apache.org/jira/browse/SPARK-14984):
 `spark.ml.regression.LinearRegressionSummary`에서 `model`필드는 사용되지 않습니다.
* [SPARK-13784](https://issues.apache.org/jira/browse/SPARK-13784):
 `spark.ml.regression.RandomForestRegressionModel`과 `spark.ml.classification.RandomForestClassificationModel`에서
 `getNumTrees`로 인하여 `numTrees`파라미터는 사용되지 않습니다.
* [SPARK-13761](https://issues.apache.org/jira/browse/SPARK-13761):
 `spark.ml.param.Params`에서 `validateParams`은 사용되지 않습니다. `transformSchema`에 재정의 되는 메서드의 모든 기능들을 옮겼습니다.
* [SPARK-14829](https://issues.apache.org/jira/browse/SPARK-14829):
 `spark.mllib`패키지의 `LinearRegressionWithSGD`, `LassoWithSGD`, `RidgeRegressionWithSGD`, `LogisticRegressionWithSGD`은 사용되지 않습니다.
 `spark.ml.regression.LinearRegression`과 `spark.ml.classification.LogisticRegression` 사용을 권장합니다.
* [SPARK-14900](https://issues.apache.org/jira/browse/SPARK-14900):
 `spark.mllib.evaluation.MulticlassMetrics`에서 `precision`, `recall`, `fMeasure`은 사용되지 않고 `accuracy`가 대체합니다.
* [SPARK-15644](https://issues.apache.org/jira/browse/SPARK-15644):
 `spark.ml.util.MLReader`와 `spark.ml.util.MLWriter`에서 `context`은 `session`으로 대체됩니다.
* `spark.ml.feature.ChiSqSelectorModel`에서, `ChiSqSelectorModel`가 사용되지 않기 때문에 `setLabelCol`은 사용되지 않습니다.

**달라진 사항**

`spark.mllib`와 `spark.ml`패키지에서 달라진 사항들 :

* [SPARK-7780](https://issues.apache.org/jira/browse/SPARK-7780):
 이항 분류 문제를 위해 `spark.mllib.classification.LogisticRegressionWithLBFGS`은 직접적으로 `spark.ml.classification.LogisticRegression`을 호출합니다.
 `spark.mllib.classification.LogisticRegressionWithLBFGS`의 달라진 사항들 : 
    * L1/L2 Updater를 포함한 이항 분류 모형을 훈련시킬 때, 절편은 제약되지 않습니다.
    * 제약이 없는 구성이면, 특성 스케일링 여부과 관계 없이 훈련은 같은 수렴률에 따른 동일한 해결안을 제공합니다.    
* [SPARK-13429](https://issues.apache.org/jira/browse/SPARK-13429):
 `spark.ml.classification.LogisticRegression`의 일관되고 더 나은 결과를 제공을 하기위해, 
 `spark.mllib.classification.LogisticRegressionWithLBFGS`: `convergenceTol`의 디폴트 값은 1E-4에서 1E-6으로 달라집니다.
* [SPARK-12363](https://issues.apache.org/jira/browse/SPARK-12363):
 결과물을 바꿀 가능성이 있는 `PowerIterationClustering`의 버그를 수정하였습니다.
* [SPARK-13048](https://issues.apache.org/jira/browse/SPARK-13048):
  `EM` 최적화를 사용하는 `LDA`는 체크포인트가 사용될 시, 최종 체크포인트의 디폴트로 제공됩니다.
* [SPARK-12153](https://issues.apache.org/jira/browse/SPARK-12153):
 `Word2Vec`의 문장 바운더리는 이전에는 정확하게 핸들링하지 못했지만 지금은 신뢰할수 있습니다.  
* [SPARK-10574](https://issues.apache.org/jira/browse/SPARK-10574):
  `spark.ml`과 `spark.mllib`의 `HashingTF`은 `MurmurHash3`을 해쉬 알고리즘의 디폴트로 사용합니다.
* [SPARK-14768](https://issues.apache.org/jira/browse/SPARK-14768):
 PySpark `Param`에 대한 논쟁이 있는 `expectedType`은 제거되었습니다.
* [SPARK-14931](https://issues.apache.org/jira/browse/SPARK-14931):
 스칼라 파이프라인과 파이썬 사이에 맞지 않았던 디폴트 `Param` 값은 바뀌었습니다.
* [SPARK-13600](https://issues.apache.org/jira/browse/SPARK-13600):
`QuantileDiscretizer` 는 `spark.sql.DataFrameStatFunctions.approxQuantile`을 사용하여 분리점을 찾습니다. (이전엔 전통적인 샘플링 로직을 이용하였습니다.)
 동일한 입력 데이터와 파라미터를 사용하지만 결과물은 다를수 있습니다.
 

## 이전 스파크 버전들

이전 마이그레이션 가이드는 [on this page](ml-migration-guides.html)에 있습니다.

---
