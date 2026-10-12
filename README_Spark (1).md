# Implementation of Apache Spark

# Commands

## Check / install Java

```bash
java -version
sudo apt-get install default-jdk
```

## Check / install Scala

```bash
scala -version
sudo apt-get install scala
```

## Extract and install Spark

```bash
cd Downloads
ls
tar xvf spark-1.6.1-bin-hadoop2.6.tgz
ls

sudo su -
cd /home/himanshu/Downloads/
mv spark-1.6.1-bin-hadoop2.6 /usr/local/spark
exit
```

Check the move worked (shown in the screenshot):

```bash
cd /usr/local/spark
ls
```

## Set environment variable

Add this line to `~/.bashrc`, then reload it:

```bash
export PATH=$PATH:/usr/local/spark/bin
```

```bash
gedit ~/.bashrc
source ~/.bashrc
```

## Start Spark shell

```bash
spark-shell
```

## Word count in Spark shell

```scala
val inputfile = sc.textFile("input.txt")
val counts = inputfile.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey(_ + _)
counts.saveAsTextFile("output")
exit
```

## View output

```bash
cd output
ls
cat part-00000
```

---

# Code

## input.txt

```text
In as name to here them deny wise this. As rapid woody my he me which. Men but they fail shew just wish next put. Led all visitor musical calling nor her. Within coming figure sex things are. Pretended concluded did repulsive education smallness yet yet described. Had countryman his pressed shewing. No gate dare rose he. Eyes year if miss he as upon.
```

## SparkWordCount.scala

```scala
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._
import org.apache.spark._

object SparkWordCount {
  def main(args: Array[String]) {
    val sc = new SparkContext("local", "Word Count", "/usr/local/spark", Nil, Map(), Map())
    val input = sc.textFile("input.txt")
    val count = input.flatMap(line => line.split(" "))
                     .map(word => (word, 1))
                     .reduceByKey(_ + _)
    count.saveAsTextFile("outfile")
    System.out.println("OK");
  }
}
```
