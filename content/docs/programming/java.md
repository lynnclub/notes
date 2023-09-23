---
title: "Java入门"
weight: 8
---

官方网站  
[https://www.java.com/](https://www.java.com/)

基础语法  
[https://www.runoob.com/java/java-tutorial.html](https://www.runoob.com/java/java-tutorial.html)

## OpenJDK

OpenJDK 完全开源免费，不允许使用Java商标，由 Oracle、Redhat 等开源组织维护。OracleJDK 只是 OpenJDK 的发行版，就像原生安卓系统和手机厂商定制操作系统的关系。

其它 OpenJDK 发行版，比如 AWS 的 Amazon Corretto，阿里巴巴的 Alibaba Dragonwell，华为的 毕昇SDK，腾讯的 Kona，微软的 Microsoft Build of OpenJDK 等等。

Redhat 接替 Oracle 维护 JDK 的长期支持版，比如 OpenJDK 8 和 OpenJDK 11。OpenJDK 和 OracleJDK 大致相同，但是可能有细节不一致，可能导致相同代码运行出的结果不一致，比如加解密算法。

### JDK 16 贡献排名

![JDK 16 贡献排名](/notes/images/computer/java1.png)

参与贡献的厂商，几乎都有自己的OpenJDK发行版，并且在各自领域推行自家JDK，Java阵营四分五裂。

### JDK版本使用率

![JDK 16 贡献排名](/notes/images/computer/java2.png)

### 版本选择

OracleJDK 收费、OpenJDK 阉割，最新版本Java19  
Java1.8 = Java8  
Java8 2030年停止更新  
Java8、Java11最流行，都是LTS长期支持版本

Java8还没有拆分出OracleJDK，可以免费使用，可以不使用OpenJDK避免兼容性问题，而且支持到2030年，目前（2023年）还是最佳选择。

Java8

![Java8](/notes/images/computer/java3.png)

Java8 openjdk
![Java8 openjdk](/notes/images/computer/java4.png)

JRE下载地址 [https://www.java.com/zh-CN/download/](https://www.java.com/zh-CN/download/)  
JDK下载地址 [https://www.oracle.com/java/technologies/downloads/#java8-mac](https://www.oracle.com/java/technologies/downloads/#java8-mac)

## 数据结构

基本数据类型

1. byte：等同于int8
2. short：int16
3. int：int32
4. long：int64
5. float：float32
6. double：float64
7. boolean
8. char：单一的16位Unicode字符
9. 数组：用来存储固定大小的同类型元素

高级数据结构

字符串：基于String类创建，属于对象  
枚举 Enumeration  
位集合 BitSet  
向量 Vector  
栈 Stack  
字典 Dictionary  
哈希表 Hashtable  
属性 Properties  
ArrayList：可以动态修改的数组  
链表 LinkedList  
HashMap：键值散列表  
HashSet：基于HashMap实现，不允许有重复元素的集合  
迭代器 Iterator：用于访问集合的方法，可用于迭代 ArrayList 和 HashSet 等集合。  
泛型：多种类型组合。

Java集合框架图

![Java集合框架图](/notes/images/computer/java5.png)

## 历代框架

SSH（14年前）：Struts(MVC)、Spring和Hibernate(DB)  
SSM（14年后）：SpringMVC、Spring和MyBatis(DB)  
SpringBoot：基于Spring4.0设计，SSM脚手架，功能完整、开箱即用

《一步一步教你用IntelliJ IDEA 搭建SSM框架》  
[http://t.zoukankan.com/greenteaone-p-11078108.html](http://t.zoukankan.com/greenteaone-p-11078108.html)

《Spring MVC 4.2.4.RELEASE 中文文档》  
[https://www.w3cschool.cn/spring_mvc_documentation_linesh_translation/](https://www.w3cschool.cn/spring_mvc_documentation_linesh_translation/)

## Spring架构图

![Spring架构图](/notes/images/computer/java6.png)

标注：org.springframework 是新版组织名称，springframework 是16年前的历史版本。

## Spring容器与Bean

IOC（控制反转）容器是个实例池，工厂模式，通过反射等方式自动实例化。JavaBean 是一种可重用组件，为了方便容器将其自动实例化，JavaBean 需要定义无参数构造函数。

![Spring容器与Bean](/notes/images/computer/java7.png)

## Maven

maven 是一个项目管理工具，可以对 Java 项目进行构建、依赖管理。maven用哪个版本的JDK运行，取决于环境变量JAVA_HOME指向哪个版本。maven处理不了多层依赖，很多复杂项目使用gradle。

教程  
[https://www.runoob.com/maven/maven-tutorial.html](https://www.runoob.com/maven/maven-tutorial.html)

### 基本使用

创建maven项目

```shell
mvn archetype:generate -DgroupId=com.lynn -DartifactId=command -Dpackaging=com.lynn.command -Dversion=1.0-SNAPSHOT
```

进入项目包目录，打包或编译

```shell
mvn clean package
mvn clean compile
```

进入编译后的target/classes目录，执行main方法

```shell
java com.lynn.RSA
java -jar command-1.0-SNAPSHOT.jar
```

### 仓库

开源包可以从maven 中央仓库 自动拉取，还支持私有部署的 远程仓库，但是私有的本地包需要手动导入，最终都是使用本地仓库。 本地仓库在用户目录下的 .m2/repository/ 目录。

![repository](/notes/images/computer/java8.png)

#### 查询中央仓库

[https://search.maven.org/#browse](https://search.maven.org/#browse) 或者 [https://central.sonatype.com/](https://central.sonatype.com/)

标注：必须根据组织名称、包名查，完整类名几乎查不到

#### 安装本地包

```shell
mvn install:install-file -DgroupId=com.mg.master -DartifactId=spring -Dversion=1.0-SNAPSHOT -Dpackaging=jar -Dfile=lib/spring-1.0-SNAPSHOT.jar
mvn install:install-file -DgroupId=com.mg.master -DartifactId=utils-core -Dversion=1.0-SNAPSHOT -Dpackaging=jar -Dfile=lib/utils-core-1.0-SNAPSHOT.jar

```

#### 移除包

```shell
mvn dependency:purge-local-repository -DmanualInclude="com.mg.master:spring"
mvn dependency:purge-local-repository -DmanualInclude="com.mg.master:utils-core"
```

#### IDEA项目安装依赖包

```shell
mvn install
mvn idea:idea
```

标注：maven依赖太难用了，根本不知道依赖类在哪个包，就算知道了也不知道应该用哪个版本，需要试。

![maven依赖问题](/notes/images/computer/java9.png)

## 区分环境

运行时VM参数 -Dspring.profiles.active=dev  
打包时指定环境 mvn clean package -P test -Dmaven.test.skip=true

## IDEA

File--Project Structure 设置JDK版本，不建议手动设置Maven包（本地包可以用命令安装），建议添加pom的依赖。

## Tomcat

Java8 需要搭配 Tomcat 9

## 定时任务

原始方式：Thread线程等待，创建线程，sleep循环执行；  
古老方式：JDK的timer包，单线程，执行时间长或报错会影响其它任务，不稳定；  
ScheduledExecutorService：Java1.5新增，通过线程池解决了timer包的问题；  
Spring Scheduled注解：轻量，易用；  
Quartz：开源框架，很强大；
