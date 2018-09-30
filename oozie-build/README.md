## oozie编译
### 一、版本
 - oozie版本：**4.3.1**
 - JDK：**1.8**
 - HADOOP：**2.7.7**
 - hive：**1.2.2**
 - pig：**0.16.0**
 - sqoop：**1.4.7**
 - spark：**2.3.0**
 - hbase：**0.94.27**
 - tomcat：**8.0.53**
 
### 二、编译前需要修改的地方
 - root pom.xml
    1. jdk版本修改
        ```xml
           <properties>
               <targetJavaVersion>1.8</targetJavaVersion>
               <sourceJavaVersion>1.8</sourceJavaVersion>
               <minJavaVersion>1.7</minJavaVersion>
           </properties>
        ```
    2. hadoop版本修改
        ```xml
           <properties>
               <hadoop.version>2.7.7</hadoop.version>
           </properties>
        ```
    3. hbase版本成新的,可能会在maven中心仓库中找不到相应的jar包导致报错,可不修改.
    4. hive版本修改:
        ```xml
           <hive.version>1.2.2</hive.version>
        ```
    5. sqoop 版本修改
        ```xml
           <sqoop.version>1.4.7</sqoop.version>
           <!--
               将sqoop版本修改为1.4.7时,还需要将<id>hadoop-2</id>下面对应的sqoop.classifier值改成hadoop260
               <sqoop.classifier>hadoop260</sqoop.classifier>
               <profile>
                    <id>hadoop-2</id>
                    <activation>
                        <activeByDefault>true</activeByDefault>
                    </activation>
                    <properties>
                        <hadoop.version>2.4.0</hadoop.version>
                        <hadoop.majorversion>2</hadoop.majorversion>
                        <pig.classifier>h2</pig.classifier>
                        <sqoop.classifier>hadoop260</sqoop.classifier>
                        <jackson.version>1.9.13</jackson.version>
                    </properties>
               </profile>
           -->
        ```
    6. 修改spark版本以及对应的scala版本
       ```xml
           <properties>
                <spark.version>2.3.0</spark.version>
                <spark.streaming.kafka.version>1.6.3</spark.streaming.kafka.version>
                <spark.bagel.version>1.6.3</spark.bagel.version>
                <spark.guava.version>14.0.1</spark.guava.version>
                <spark.scala.binary.version>2.11</spark.scala.binary.version>
           </properties>
       ```
    7. 修改tomcat版本(oozie默认使用的是tomcat6的)
        ```xml
            <tomcat.version>8.0.53</tomcat.version>
        ```
        同时还要修改子项目distro中的pom文件,修改的地方如下:
        将tomcat-6改成tomcat-8
        ```
           ...
           <plugin>
              ...   
             <mkdir dir="downloads"/>
             <get src="http://archive.apache.org/dist/tomcat/tomcat-8/v${tomcat.version}/bin/apache-tomcat-${tomcat.version}.tar.gz"
                  dest="downloads/tomcat-${tomcat.version}.tar.gz" verbose="true" skipexisting="true"/>
             ...
           </plugin>
      
        ```
### 三、编译环境准备
   - 安装maven(可以使用最新版本)<br/>
     直接去maven官网下载linux版本的maven安装包,解压后配置好环境变量.
     同时需要在conf目录下配置好其下载资源的localRepository(需要特别说明:该目录对于下面用于编译oozie的用户需要有读写权限,否则会报oozie的parent project不能构建),同时,如果嫌弃maven中心库下载东西慢的话,可以使用阿里云的mirror,但是不全.
     ```
     <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
       <!-- localRepository
        | The path to the local repository maven will use to store artifacts.
        |
        | Default: ${user.home}/.m2/repository
       <localRepository>/path/to/local/repo</localRepository>
       -->
     <!--配置目录-->
     <localRepository>/usr/repo</localRepository>
     ...
     ```
   - git安装,可以直接使用`yum -y install git` 命令进行安装.
   - svn安装,可以直接使用`yum -y install svn` 命令进行安装.
   
    
### 四、编译
   - 进入到oozie的安装目录下,执行以下命令:<br/>
   `bin/mkdistro.sh -DskipTests -Phadoop-2 -Dhadoop.auth.version=2.7.7 -Ddistcp.version=2.7.7`<br/>
   编译过程中如果发现某些jar包下载失败,需要自己下载相应的jar包放到maven的localRepository对应的目录下.<br/>
   同时,编译过程中,可能会报如下错误:<br/>
        ```
         [ERROR] Java heap space -> [Help 1]
         [ERROR] 
         [ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
         [ERROR] Re-run Maven using the -X switch to enable full debug logging.
         [ERROR] 
         [ERROR] For more information about the errors and possible solutions, please read the following articles:
         [ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/OutOfMemoryError

        ```
     解决方案:在命令行输入`export MAVEN_OPTS='-Xmx512m -XX:MaxPermSize=128m'`后再重新执行编译命令.
     
### 五、编译结果.
   - 待编译很久后,如果编译成功,显示如下:
        ```
            [INFO] ------------------------------------------------------------------------
            [INFO] Reactor Summary:
            [INFO] 
            [INFO] Apache Oozie Main 4.3.1 ............................ SUCCESS [  2.022 s]
            [INFO] Apache Oozie Hadoop Utils hadoop-2-4.3.1 hadoop-2-4.3.1 SUCCESS [  3.970 s]
            [INFO] Apache Oozie Hadoop Distcp hadoop-2-4.3.1 hadoop-2-4.3.1 SUCCESS [  0.231 s]
            [INFO] Apache Oozie Hadoop Auth hadoop-2-4.3.1 Test hadoop-2-4.3.1 SUCCESS [  0.611 s]
            [INFO] Apache Oozie Hadoop Libs ........................... SUCCESS [  0.054 s]
            [INFO] Apache Oozie Client ................................ SUCCESS [ 22.406 s]
            [INFO] Apache Oozie Share Lib Oozie ....................... SUCCESS [  5.780 s]
            [INFO] Apache Oozie Share Lib HCatalog .................... SUCCESS [  4.434 s]
            [INFO] Apache Oozie Share Lib Distcp ...................... SUCCESS [  1.246 s]
            [INFO] Apache Oozie Core .................................. SUCCESS [01:25 min]
            [INFO] Apache Oozie Share Lib Streaming ................... SUCCESS [  8.800 s]
            [INFO] Apache Oozie Share Lib Pig ......................... SUCCESS [  7.937 s]
            [INFO] Apache Oozie Share Lib Hive ........................ SUCCESS [ 13.658 s]
            [INFO] Apache Oozie Share Lib Hive 2 ...................... SUCCESS [ 13.582 s]
            [INFO] Apache Oozie Share Lib Sqoop ....................... SUCCESS [  7.564 s]
            [INFO] Apache Oozie Examples .............................. SUCCESS [ 14.970 s]
            [INFO] Apache Oozie Share Lib Spark ....................... SUCCESS [ 31.352 s]
            [INFO] Apache Oozie Share Lib ............................. SUCCESS [ 17.920 s]
            [INFO] Apache Oozie Docs .................................. SUCCESS [ 12.683 s]
            [INFO] Apache Oozie WebApp ................................ SUCCESS [01:24 min]
            [INFO] Apache Oozie Tools ................................. SUCCESS [01:24 min]
            [INFO] Apache Oozie MiniOozie ............................. SUCCESS [01:24 min]
            [INFO] Apache Oozie Distro ................................ SUCCESS [01:24 min]
            [INFO] Apache Oozie ZooKeeper Security Tests 4.3.1 ........ SUCCESS [01:24 min]
            [INFO] ------------------------------------------------------------------------
            [INFO] BUILD SUCCESS
            [INFO] ------------------------------------------------------------------------
            [INFO] Total time: 05:41 min
            [INFO] Finished at: 2018-09-14T23:56:36+08:00
            [INFO] ------------------------------------------------------------------------
        ```
        编译成功后的安装包在子项目distro的target目录下的oozie-4.3.1-distro.tar.gz即是我们需要的oozie安装包.
        到此,oozie源码编译完成.