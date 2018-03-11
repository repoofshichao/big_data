## ubuntu16.04搭建Hadoop2.7.2+spark1.6.1+mysql+hive2.0.0伪分布环境

### 准备
hadoop: https://archive.apache.org/dist/hadoop/common/hadoop-2.7.2/
hive:  http://archive.apache.org/dist/hive/hive-2.0.0/
spark: https://archive.apache.org/dist/spark/spark-1.6.1/spark-1.6.1-bin-hadoop2.6.tgz
scala: https://downloads.lightbend.com/scala/2.10.6/scala-2.10.6.tgz

## hadoop 安装

### 配置环境变量
```
export JAVA_HOME=/usr/local/jdk1.8.0_161
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export PATH=${JAVA_HOME}/bin:$JRE_HOME/bin:$PATH

#scala
export SCALA_HOME=/home/jack/app/scala-2.10.6
export PATH=${SCALA_HOME}/bin:$PATH

#spark
export SPARK_HOME=/home/jack/spark1
export PATH=$SPARK_HOME/bin:$PATH

# hadoop2.7.2
export HADOOP_PREFIX=/home/jack/hadoop2
export CLASSPATH=.:$JAVA_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/:$HADOOP_PREFIX/bin:$PATH
export HADOOP_PREFIX PATH CLASSPATH

#Hive
export HIVE_HOME=/home/jack/app/hive-2.0.0
export PATH=$PATH:${HIVE_HOME}/bin
export CLASSPATH=$CLASSPATH.:{HIVE_HOME}/lib

export PATH=/home/jack/hadoop2/bin:/home/jack/hadoop2/sbin:$JAVA_HOME/:$HADOOP_PREFIX/bin:$PATH
```

### 安装rsync
sudo apt-get install rsync

### ssh
sudo apt-get install openssh-server
cd ~/.ssh/   # 若没有该目录，请先执行一次ssh localhost
ssh-keygen -t rsa   # 会有提示，都按回车就可以
cat id_rsa.pub >> authorized_keys  # 加入授权
使用ssh localhost试试能否直接登录

### /etc/hosts
```
127.0.0.1	localhost
127.0.1.1	jack-vm

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

### 修改配置文件：etc/hadoop/hadoop-env.sh
```
export JAVA_HOME=/usr/local/jdk1.8.0_161
```

### 修改文件etc/hadoop/core-site.xml
```
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/home/jack/hadoop2/tmp</value>
        <description>Abase for other temporary directories.</description>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
  <property>
      <name>io.file.buffer.size</name>
      <value>131072</value>
    </property>
   <property>
       <name>hadoop.proxyuser.master.hosts</name>
        <value>*</value>
   </property>
   <property>
       <name>hadoop.proxyuser.master.groups</name>
       <value>*</value>
   </property>
</configuration>
```

### 修改etc/hadoop/hdfs-site.xml:
```
<configuration>
    <property>
            <name>dfs.namenode.name.dir</name>
            <value>file:/home/jack/hadoop2/tmp/dfs/namenode</value>
     </property>
     <property>
            <name>dfs.datanode.data.dir</name>
            <value>file:/home/jack/hadoop2/tmp/dfs/datanode</value>
     </property>

     <property>
            <name>dfs.replication</name>
            <value>1</value>
     </property>
     <property>
            <name>dfs.webhdfs.enabled</name>
            <value>true</value>
     </property>
</configuration>
```

### 修改配置文件mapred-site.xml
```
<configuration>
    <property>
          <name>mapreduce.framework.name</name>
          <value>yarn</value>
    </property>
    <property>
          <name>mapreduce.jobhistory.address</name>
          <value>localhost:10020</value>
    </property>
    <property>
          <name>mapreduce.jobhistory.webapp.address</name>
          <value>localhost:19888</value>
     </property>
</configuration>
```

### 修改配置文件yarn-site.xml
```
<configuration>

<!-- Site specific YARN configuration properties -->
    <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
    </property>
    <property>
      <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
      <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
       <name>yarn.resourcemanager.address</name>
       <value>localhost:8032</value>
    </property>
    <property>
         <name>yarn.resourcemanager.scheduler.address</name>
         <value>localhost:8030</value>
    </property>
    <property>
         <name>yarn.resourcemanager.resource-tracker.address</name>
         <value>localhost:8031</value>
    </property>
<!--resourcemanager 管理器端口-->
     <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>localhost:8033</value>
    </property>
<!--resourcemanager 的 Web 端口，监控 job 的资源调度 -->
    <property>
         <name>yarn.resourcemanager.webapp.address</name>
         <value>localhost:8088</value>
    </property>
</configuration>
```

### 启动
```
source .bashrc 
start-dfs.sh
jps 
```
输出：
14242 Jps
14106 SecondaryNameNode
13922 DataNode------------------(无namenode)

```
hdfs namenode -format
stop-dfs.sh 
start-dfs.sh 
jps 
```
输出：
14919 NameNode-----------------------（namenode）
15407 Jps
15271 SecondaryNameNode
15073 DataNode
```
start-yarn.sh
jps 
```
输出：
15625 NodeManager
14919 NameNode
15271 SecondaryNameNode
15073 DataNode
15937 Jps
15501 ResourceManager

### 验证：
yarn：http://localhost:8088/
hadoop:  http://localhost:50070

## spark安装
### 配置Spark环境变量
上面已经配置过。

### 配置spark-env.sh
将spark-env.sh.template复制为spark-env.sh 
增加如下内容：
```
export JAVA_HOME=/usr/local/jdk1.8.0_161

export SCALA_HOME=/home/jack/app/scala-2.10.6
export SPARK_WORKER_MEMORY=1g
export SPARK_MASTER=jack-vm
export SPARK_MASTER_PORT=7077
export SPARK_MASTER_WEBUI=8099
export SPARK_WORKER_CORES=1
export HADOOP_CONF_DIR=/home/jack/hadoop2/etc/hadoop
```

### 测试
pyspark

## Hive2.0.0安装
### mysql安装
$sudo apt-get install mysql-server
登录mysql：$mysql -u root -p
建立数据库hive：mysql>create database hive;
mysql>show databases;//查看创建；

这里一定要把hive数据库的字符集修改为latin1，而且一定要在hive初次启动的时候就修改字符集 （否则就等着删除操作的时候死掉吧）
mysql>alter database hive character set latin1;

创建hive用户,并授权：mysql>grant all on hive.* to hive@'%'  identified by 'hive';

更新：mysql>flush privileges; 

查询mysql的版本：mysql>select version();

下载mysql的JDBC驱动包： http://dev.mysql.com/downloads/connector/j/

下载mysql-connector-java-xxx.tar.gz ，复制msyql的JDBC驱动包到Hive的lib目录下。

## Hive安装
### 环境配置
见.bashrc
### 配置hive-env.sh文件
复制hive-env.sh.template，修改hive-env.sh文件
```
HADOOP_HOME=/home/jack/hadoop2
export HIVE_CONF_DIR=/home/jack/app/hive-2.0.0/conf
export HIVE_AUX_JARS_PATH=/home/jack/app/hive-2.0.0/lib
```

### 配置hive-site.xml文件
cp hive-default.xml.template hive-site.xml
添加如下内容
```
  <property>
   <name>hive.metastore.local</name>
   <value>true</value>
   <description>使用本机mysql服务器存储元数据。这种存储方式需要在本地运行一个mysql服务器</description>
  </property>
  <property>
   <name>javax.jdo.option.ConnectionURL</name>
   <value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true</value>
   <description> 使用的数据库?charcherEncoding=UTF-8</description>
  </property>
  <property>
   <name>javax.jdo.option.ConnectionDriverName</name>
   <value>com.mysql.jdbc.Driver</value>
   <description>使用的链接方式</description>
  </property>

  <property>
   <name>javax.jdo.option.ConnectionUserName</name>
   <value>hive</value>
   <description>mysql用户名</description>
  </property>
  <property>
   <name>javax.jdo.option.ConnectionPassword</name>
   <value>root</value>
  </property>
  <property>
  <name>hive.metastore.warehouse.dir</name>
  <value>/user/hive/warehouse</value>
  <description>元数据存放的地放，需要在本地（不是hdfs中）新建这个目录</description>
  </property>

  <property>
    <name>hive.exec.scratdir</name>
    <value>/user/hive/tmp</value>
    <description> hive的数据临时文件目录，需要在本地新建这个目录HDFS root scratch dir for Hive jobs which gets created with write all (733) permission. For each connecting user, an HDFS scratch dir: ${hive.exec.scratchdir}/&lt;username&gt; is created, with ${hive.scratch.dir.permission}.</description>
  </property>

  <property>
    <name>hive.querylog.location</name>
    <value>/user/hive/log</value>
    <description>这个是用于存放hive相关日志的目录</description>
  </property>

  <property>
    <name>hive.cli.print.current.db</name>
    <value>true</value>
  </property>
```

### 建立存储目录：
$ hadoop fs -mkdir -p  /user/hive/tmp
$ hadoop fs -mkdir -p /user/hive/log
$ hadoop fs -mkdir -p /user/hive/warehouse
$ hadoop fs -chmod g+w   /user/hive/tmp

$ hadoop fs -chmod g+w   /user/hive/log
$ hadoop fs -chmod g+w   /user/hive/warehouse    /usr/hive/tmp

### 配置 hive-log4j.proprties
```
property.hive.log.dir = /home/jack/app/hive-2.0.0/log
property.hive.log.file = hive.log
```

### 初始化数据库
$ schematool -initSchema -dbType mysql -userName=hive -passWord=hive

### 查看mysql
systemctl status mysql //mysql运行中
确认mysql的mysql-connector-java-xxx-bin.jar 包放到 hive-2.0.0/lib目录下

### 启动metastore
 hive --service metastore > /tmp/hive_metastore.log 2>&1 &
 
### 启动Hadoop服务：
$sbin/start-dfs.sh
$sbin/start-yarn.sh

### 启动hive
$hive

### 处理报错：
===> ${system:java.io.tmpdir%7D/$%7Bhive.session.id%7D_resources 
原因：hive-site.xml 中的目录没有配置
解决办法： 
在主机某目录下新建一个文件夹 例如 /home/hive/tmpdir 
编辑$HIVE_HOME/conf/hive-site.xml文件找到含有那一串字符的地方替换成刚才建立的目录就行了。

### 验证hive
create table test(id int, name string) row format delimited FIELDS TERMINATED BY ',';

mysql -u hive -p
mysql> select* from TBLS;

Hadoop中查看生成的文件：

$ hdfs dfs -ls /user/hive/warehouse/
Found 1 items
drwxrwxrwx   -  supergroup          0 2016-04-24 11:53 /user/hive/warehouse/test