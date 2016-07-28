
在windows平台安装hadoop2.6.x与Spark


# 为什么要在Windows平台上运行?

因为我的开发测试在Windows机子上,相关软件都安装好了,再为Hadoop的编译打包搞个Linux虚拟机,硬盘空间又要浪费一大片.
使用Linux是更为方便,因为很多软件都可以直接命令式的下载安装,不像Windows那样到处找地方下载.



# 安装配置HADOOP

Windows平台编译打包Hadoop可以根据官方说明文档,但需要VS2012的专业版的环境,还需要下载WindowsSDK,所以就先放弃了.
于是上网找了预编译的版本,虽然是2.6.0,但是用来测试应该是足够了.

## 环境变量配置

根据官方文档进行配置,启动单节点(伪分布式).

主要配置目录为 HADOOP_HOME/etc/hadoop,下面为配置文件及内容

### HDFS配置

hadoop-env.cmd (文件尾添加下面的行)

    set HADOOP_PREFIX=c:\deploy
    set HADOOP_CONF_DIR=%HADOOP_PREFIX%\etc\hadoop
    set YARN_CONF_DIR=%HADOOP_CONF_DIR%
    set PATH=%PATH%;%HADOOP_PREFIX%\bin
    
core-site.xml

    <configuration>
      <property>
        <name>fs.default.name</name>
        <value>hdfs://0.0.0.0:19000</value>
      </property>
    </configuration>

hdfs-site.xml

    <configuration>
      <property>
        <name>dfs.replication</name>
        <value>1</value>
      </property>
    </configuration>

slaves (确认是否是下面的内容)

    localhost
    


### YARN配置

mapred-site.xml

    <configuration>
       <property>
         <name>mapreduce.job.user.name</name>
         <value>%USERNAME%</value>
       </property>
       <property>
         <name>mapreduce.framework.name</name>
         <value>yarn</value>
       </property>
      <property>
        <name>yarn.apps.stagingDir</name>
        <value>/user/%USERNAME%/staging</value>
      </property>
      <property>
        <name>mapreduce.jobtracker.address</name>
        <value>local</value>
      </property>
    </configuration>

yarn-site.xml

    <configuration>
      <property>
        <name>yarn.server.resourcemanager.address</name>
        <value>0.0.0.0:8020</value>
      </property>
      <property>
        <name>yarn.server.resourcemanager.application.expiry.interval</name>
        <value>60000</value>
      </property>
      <property>
        <name>yarn.server.nodemanager.address</name>
        <value>0.0.0.0:45454</value>
      </property>
      <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
      </property>
      <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
      </property>
      <property>
        <name>yarn.server.nodemanager.remote-app-log-dir</name>
        <value>/app-logs</value>
      </property>
      <property>
        <name>yarn.nodemanager.log-dirs</name>
        <value>/dep/logs/userlogs</value>
      </property>
      <property>
        <name>yarn.server.mapreduce-appmanager.attempt-listener.bindAddress</name>
        <value>0.0.0.0</value>
      </property>
      <property>
        <name>yarn.server.mapreduce-appmanager.client-service.bindAddress</name>
        <value>0.0.0.0</value>
      </property>
      <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
      </property>
      <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>-1</value>
      </property>
      <property>
        <name>yarn.application.classpath</name>
        <value>%HADOOP_CONF_DIR%,%HADOOP_COMMON_HOME%/share/hadoop/common/*,%HADOOP_COMMON_HOME%/share/hadoop/common/lib/*,%HADOOP_HDFS_HOME%/share/hadoop/hdfs/*,%HADOOP_HDFS_HOME%/share/hadoop/hdfs/lib/*,%HADOOP_MAPRED_HOME%/share/hadoop/mapreduce/*,%HADOOP_MAPRED_HOME%/share/hadoop/mapreduce/lib/*,%HADOOP_YARN_HOME%/share/hadoop/yarn/*,%HADOOP_YARN_HOME%/share/hadoop/yarn/lib/*</value>
      </property>
    </configuration>

## 启动

### 启动HDFS

先格式化HDFS

    %HADOOP_PREFIX%\bin\hdfs namenode -format
    
启动节点

    %HADOOP_PREFIX%\sbin\start-dfs.cmd
    
### 启动YARN

    %HADOOP_PREFIX%\sbin\start-yarn.cmd
    
    
# 安装配置Spark

## 下载

下载安装Spark1.6.2的关联hadoop指定版本的包,此文我们使用下列的包:

    spark-1.6.2-bin-hadoop2.6

直接解压,同时保证环境变量中有HADOOP_HOME指向HADOOP的安装目录,同时把Hadoop\bin添加到Path中.



## 运行

直接运行./bin/spark-shell

此时可能会出现一些问题,常见的是/tmp/hive目录没有权限.

1. 创建/tmp/hive目录 

    %HADOOP_HOME%\bin\hdfs dfs -mkdir -p /tmp/hive

2. 使用winutils.exe设置权限

    %HADOOP_HOME%\bin\winutils.exe chmod 777 \tmp\hive

执行上面的步骤后,解决了spark-shell启动的错误信息.



### 参考地址

* [官方的Windows编译安装说明](http://wiki.apache.org/hadoop/Hadoop2OnWindows)
* http://spark.apache.org/docs/latest/hadoop-provided.html
* [预编译好的Wndows版本的Hadoop](https://www.barik.net/archive/2015/01/19/172716/)
* [Spark Start Guide](http://spark.apache.org/docs/latest/quick-start.html)
