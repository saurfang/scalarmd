# R Markdown with Scala/Spark

This shows you how to use [knitr language engine](http://yihui.name/knitr/demo/engines/) to bring Spark data analytics to R Markdown documents.

> [R Markdown](http://rmarkdown.rstudio.com/) is an authoring format that enables easy creation of dynamic documents, presentations, and reports from R.

> [Apache Spark™](http://spark.apache.org/) is a fast and general engine for large-scale data processing.

## Requiremetns

### jvmr
We use the wonderful [jvmr](http://dahl.byu.edu/software/jvmr/) package from [David B. Dahl](https://github.com/dbdahl). It allows you evaluate Java/Scala expression and it provides an interpreter interface that builds upon [rJava](https://www.rforge.net/rJava/).

While CRAN has archived `jvmr`, you can still install it by:
```r
devtools::install_github("cran/jvmr")
```
The latest version use Scala 2.11.2. Since Spark still compiles with 2.10 by default, you might want to install `jvmr` by:
```r
devtools::install_url("http://cran.r-project.org/src/contrib/Archive/jvmr/jvmr_1.0.4.tar.gz")
```

### Java/Spark
Of course you will need Java and Spark as usual. We pick up the Spark location by `SPARK_HOME` environment variable. [SparkR](https://github.com/amplab-extras/SparkR-pkg) might make this part easier and more powerful in the future.

## Overview
The motivation behind this is that R remains great in data analytics and statistical modeling however unable to handle large dataset is something at root of R language design. Aside from the analytics capabilities, R is also wonderful in terms of visualization and interactivity. Spark excels in terms of scalability but lacks the interactivity. We wonder if we can get the best of both world.

It turns out this can be as simple as this:
```r
library(jvmr)
library(knitr)
scala <- scalaInterpreter()
knit_engines$set(scalar = function(options) {
  code <- paste(options$code, collapse = "\n")
  output <- capture.output(interpret(scala, code, echo.output = TRUE))
  engine_output(options, options$code, output)
})
```
This pipes Scala expressions into interpreter and return results which can be potentially coerced into R datatype.

## Examples

### [scala.Rmd](scala.md)
A quick example on how this works with regular Scala interpreter.

### [spark-textclass.Rmd](spark-textclass.md) 
reproduce the results from [SimpleTextClassificationPipeline](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/ml/SimpleTextClassificationPipeline.scala) and plots a ROC curve.

*Note: This doesn't work just yet. `jvmr` and `rJava` either hangs indefinitely or throws `java.lang.OutOfMemoryError: PermGen space` exeception.*

## Related Works
* [SparkR](https://github.com/amplab-extras/SparkR-pkg)
* [ISpark](https://github.com/tribbloid/ISpark)
* [H2O](http://0xdata.com/)
* [Databricks Cloud](https://databricks.com/product/databricks-cloud)
