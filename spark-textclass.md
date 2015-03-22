# SimpleTextClassificationPipeline

The following sets up the Spark evaluation environment.

```r
options(java.parameters = c("-Xmx1G", "-XX:MaxPermGen=512M"))
library(rJava)
sparkHome <- Sys.getenv("SPARK_HOME", "/usr/local/Cellar/apache-spark/1.3.0/libexec")
#sparkClasspath <- system2(file.path(sparkHome, "bin/compute-classpath.sh"), stdout = TRUE)
#.jinit ourselves to bring logging inside
.jinit(list.files(file.path(sparkHome, "lib"), full.names = TRUE))
library(jvmr)
library(knitr)
scala <- scalaInterpreter()
knit_engines$set(sparkr = function(options) {
  code <- paste(options$code, collapse = "\n")
  output <- capture.output(interpret(scala, code, echo.output = options$results != "hide"))
  engine_output(options, options$code, output)
})
read_chunk("SimpleTextClassificationPipeline.scala", labels = "textclass")
```

Now we can run the entire pipeline now

```sparkr
/*
 * Based on: https://raw.githubusercontent.com/apache/spark/master/examples/src/main/scala/org/apache/spark/examples/ml/SimpleTextClassificationPipeline.scala
 *
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

//package org.apache.spark.examples.ml

import scala.beans.BeanInfo

import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.ml.Pipeline
import org.apache.spark.ml.classification.LogisticRegression
import org.apache.spark.ml.feature.{HashingTF, Tokenizer}
import org.apache.spark.mllib.linalg.Vector
import org.apache.spark.sql.{Row, SQLContext}

@BeanInfo
case class LabeledDocument(id: Long, text: String, label: Double)

@BeanInfo
case class Document(id: Long, text: String)

/**
 * A simple text classification pipeline that recognizes "spark" from input text. This is to show
 * how to create and configure an ML pipeline. Run with
 * {{{
 * bin/run-example ml.SimpleTextClassificationPipeline
 * }}}
 */
//object SimpleTextClassificationPipeline {

//  def setup(args: Array[String]) {
    val conf = new SparkConf().setAppName("SimpleTextClassificationPipeline").setMaster("local")
    val sc = new SparkContext(conf)
    val sqlContext = new SQLContext(sc)
    import sqlContext.implicits._

    // Prepare training documents, which are labeled.
    val training = sc.parallelize(Seq(
      LabeledDocument(0L, "a b c d e spark", 1.0),
      LabeledDocument(1L, "b d", 0.0),
      LabeledDocument(2L, "spark f g h", 1.0),
      LabeledDocument(3L, "hadoop mapreduce", 0.0)))

    // Configure an ML pipeline, which consists of three stages: tokenizer, hashingTF, and lr.
    val tokenizer = new Tokenizer()
      .setInputCol("text")
      .setOutputCol("words")
    val hashingTF = new HashingTF()
      .setNumFeatures(1000)
      .setInputCol(tokenizer.getOutputCol)
      .setOutputCol("features")
    val lr = new LogisticRegression()
      .setMaxIter(10)
      .setRegParam(0.01)
    val pipeline = new Pipeline()
      .setStages(Array(tokenizer, hashingTF)) //, lr))

    // Fit the pipeline to training documents.
    val model = pipeline.fit(training.toDF())

    // Prepare test documents, which are unlabeled.
    val test = sc.parallelize(Seq(
      Document(4L, "spark i j k"),
      Document(5L, "l m n"),
      Document(6L, "mapreduce spark"),
      Document(7L, "apache hadoop")))

    // Make predictions on test documents.
    val results = model.transform(test.toDF())
    //  .select("id", "text", "probability", "prediction")
    //  .collect()

    //results
    //  .foreach { case Row(id: Long, text: String, prob: Vector, prediction: Double) =>
    //    println(s"($id, $text) --> prob=$prob, prediction=$prediction")
    //  }

//    sc.stop()
//  }
//}
```

```
## Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
## SLF4J: Class path contains multiple SLF4J bindings.
## SLF4J: Found binding in [jar:file:/usr/local/Cellar/apache-spark/1.3.0/libexec/lib/spark-assembly-1.3.0-hadoop2.4.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
## SLF4J: Found binding in [jar:file:/usr/local/Cellar/apache-spark/1.3.0/libexec/lib/spark-examples-1.3.0-hadoop2.4.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
## SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
## SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
## 15/03/21 20:27:25 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
## 15/03/21 20:27:26 INFO Slf4jLogger: Slf4jLogger started
## 15/03/21 20:27:26 INFO Server: jetty-8.y.z-SNAPSHOT
## 15/03/21 20:27:26 INFO AbstractConnector: Started SocketConnector@0.0.0.0:60936
## 15/03/21 20:27:26 INFO Server: jetty-8.y.z-SNAPSHOT
## 15/03/21 20:27:26 INFO AbstractConnector: Started SelectChannelConnector@0.0.0.0:4040
## import scala.beans.BeanInfo
## import org.apache.spark.{SparkConf, SparkContext}
## import org.apache.spark.ml.Pipeline
## import org.apache.spark.ml.classification.LogisticRegression
## import org.apache.spark.ml.feature.{HashingTF, Tokenizer}
## import org.apache.spark.mllib.linalg.Vector
## import org.apache.spark.sql.{Row, SQLContext}
## defined class LabeledDocument
## defined class Document
## conf: org.apache.spark.SparkConf = org.apache.spark.SparkConf@687f6b72
## sc: org.apache.spark.SparkContext = org.apache.spark.SparkContext@5673ef7
## sqlContext: org.apache.spark.sql.SQLContext = org.apache.spark.sql.SQLContext@3d56cce6
## import sqlContext.implicits._
## training: org.apache.spark.rdd.RDD[LabeledDocument] = ParallelCollectionRDD[0] at parallelize at <console>:59
## tokenizer: org.apache.spark.ml.feature.Tokenizer ...[1] "Java-Object{[id: bigint, text: string, words: array<string>, features: vecto]}"
```

Let
