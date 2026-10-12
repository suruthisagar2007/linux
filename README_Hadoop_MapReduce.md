# Hadoop Installation and MapReduce Word Count

# Commands

## Install prerequisites

```bash
sudo apt update
sudo apt install ssh
sudo apt install openjdk-11-jdk
java --version
```

## Download and extract Hadoop

```bash
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
tar -xzf hadoop-3.3.6.tar.gz
ls
mv hadoop-3.3.6 hadoop
```

## Create data directories

```bash
cd hadoop
ls
mkdir data
cd data
mkdir -p {datanode,namenode}
ls
```

## Find JAVA_HOME and HADOOP_HOME

```bash
dirname $(dirname $(readlink -f $(which java)))
cd ..
pwd
```

## Set environment variables

Add the exports (see Code section) to `~/.bashrc`, then reload:

```bash
gedit ~/.bashrc
source ~/.bashrc
```

## Edit Hadoop config files

Add the contents (see Code section) to each file:

```bash
cd ~/hadoop/etc/hadoop
ls
gedit hadoop-env.sh
gedit core-site.xml
gedit hdfs-site.xml
gedit mapred-site.xml
gedit yarn-site.xml
```

## Set up passwordless SSH

```bash
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 640 ~/.ssh/authorized_keys
```

## Format the namenode (first time only)

```bash
hdfs namenode -format
```

## Start Hadoop services

```bash
start-dfs.sh
start-yarn.sh
jps
```

Web UI: `http://localhost:9870`

If `start-dfs.sh` gives an error, reinstall ssh, restart the machine, and try again:

```bash
sudo apt install ssh
# restart the VM, then:
start-dfs.sh
```

## Stop Hadoop services (before shutting down)

```bash
stop-yarn.sh
stop-dfs.sh
```

---

## MapReduce: create project and source file

```bash
hadoop version
mkdir ~/wordcount
cd ~/wordcount
nano WordCount.java
```

Paste `WordCount.java` (see Code section).

## Compile

```bash
export HADOOP_CLASSPATH=$(hadoop classpath)
mkdir wordcount_classes
javac -classpath $HADOOP_CLASSPATH -d wordcount_classes WordCount.java
```

## Package into a JAR

```bash
jar -cvf wordcount.jar -C wordcount_classes/ .
```

## Prepare input data in HDFS

```bash
echo "hello hadoop hello world hadoop is fun" > input.txt
start-dfs.sh
hdfs dfs -mkdir -p /user/$USER/wordcount/input
hdfs dfs -put input.txt /user/$USER/wordcount/input
```

## Run the job

```bash
hadoop jar wordcount.jar WordCount \
/user/$USER/wordcount/input \
/user/$USER/wordcount/output
```

## View the result

```bash
hdfs dfs -cat /user/$USER/wordcount/output/part-r-00000
```

## Re-run (delete old output first)

```bash
hdfs dfs -rm -r /user/$USER/wordcount/output
```

## Monitor job (optional)

```
http://localhost:8088    (ResourceManager)
http://localhost:19888   (JobHistory)
```

---

# Code

## ~/.bashrc

```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export HADOOP_HOME=/home/osboxes/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
```

## hadoop-env.sh

```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

## core-site.xml

Place inside `<configuration>` tags:

```xml
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
</property>
```

## hdfs-site.xml

Place inside `<configuration>` tags:

```xml
<property>
    <name>dfs.replication</name>
    <value>3</value>
</property>
<property>
    <name>dfs.name.dir</name>
    <value>/home/osboxes/hadoop/data/namenode</value>
</property>
<property>
    <name>dfs.data.dir</name>
    <value>/home/osboxes/hadoop/data/datanode</value>
</property>
```

## mapred-site.xml

Place inside `<configuration>` tags:

```xml
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
```

## yarn-site.xml

Place inside `<configuration>` tags:

```xml
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
<property>
    <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
```

## WordCount.java

```java
import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {

    /* Our 'mapper' class emits (word, "1") pairs,
     * for later counting by our 'reducer' class.
     */
    public static class TokenizerMapper
                    extends Mapper<Object, Text, Text, IntWritable> {

        public void map(Object key, Text value, Context context)
                        throws IOException, InterruptedException {

            StringTokenizer itr = new StringTokenizer( value.toString() );

            while ( itr.hasMoreTokens() ) {
                word.set( itr.nextToken() );
                context.write(word, one);
            }
        }

        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();
    }

    /* Our 'reducer' class receives a sequence of (word, "1") pairs
     * and for each word in the sequence, adds up its "1"s...
     */
    public static class IntSumReducer
                    extends Reducer<Text,IntWritable,Text,IntWritable> {

        public void reduce(Text key, Iterable<IntWritable> values, Context context)
                        throws IOException, InterruptedException {
            int sum = 0;

            for (IntWritable val : values) {
                sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }

        private IntWritable result = new IntWritable();
    }

    /* The Driver: sets up and runs the job */
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "word count");

        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```
