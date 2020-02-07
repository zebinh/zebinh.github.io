---
layout: post
title: Maven实战  - 许晓斌 - 机械工业出版社
date: 2020-02-07
tags: 读书
---
# Maven实战-许晓斌-机械工业出版社

## Maven简介

maven这个词可以翻译为“知识的积累”或者“专家”。

Maven能帮我们做的事：
+ 项目构建：Maven抽象了一个完整的“构建生命周期”模型，这个模型吸取了大量其他的构建脚本和构建工具的优点。
+ 依赖管理
+ 项目信息管理

构建工具对比：
+ make：由目标、依赖、命令构成，Makefile驱动。命令依赖于系统，无法跨平台。
+ Ant：使用Java编写，可以跨平台，和make类似由目标、依赖、任务构成，build.xml驱动。如目标为jar打包一个项目，依赖于compile编译一个项目，任务就是编译为后的打包操作。
+ Maven：以上两种最大的问题就是每个项目都要重新编写Makefile、build.xml驱动文件，不能管理jar包依赖（Ant可以集成Ivy进行依赖管理）。Maven很好的解决了这两个问题。

Maven核心概念：
+ 坐标和依赖
+ 仓库
+ 生命周期
+ 插件

## Maven的安装与配置

mvn是一个shell脚本，执行mvn命令时最终调用的是Java来解析。

用户目录下的.m2目录一般存放：repository参考，和用户设置文件settings.xml(可以从maven安装目录下的conf/settings.xml复制并修改)

不要使用IDE内嵌的Maven版本：
+ IDE内嵌的Maven一般比较新，可能不稳定
+ 如果使用IDE内嵌的Maven，其跟命令行下的mvn命令使用的就不是同一个maven，会为构建带来麻烦。

## Maven使用入门

就像Make的Makefile一样、Ant的build.xml一样，Maven项目的核心是pom.xml文件。创建Mavne工程的步骤为：
+ 创建pom.xml文件
+ 按maven规范创建src/main/java工程结构

Maven的pom.xml工程定义的坐标，最佳实践：
+ groupId最好填写公司项目名称，artifactId填写模块名称中间加短划线。这样同一个公司的项目mvn install会安装到同一个目录下
+ 而主包名最好加模块名不要短划线，如这里的主包名为io.zebinh.hellodemo.hellomaven
+ maven package如果需要生成可运行的jar包，是要加入maven-shade-plugin插件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>io.zebinh.hellodemo</groupId>
  <artifactId>hello-maven</artifactId>
  <version>1.0-SNAPSHOT</version>
  <name>hello maven demo</name>
</project>
```

## 背景案例

设计一个注册登录系统

## 坐标和依赖

maven的坐标包括：groupId, artifactId, version, packaging, classifier
+ groupId：groupId不应该只定义到公司级别，而应该定义到项目级别。
+ artifactId：模块名
+ version：nexus定义了一套版本管理，遵循该版本管理即可
+ packaging：打包方式，如jar、war
+ classifier：定义附属构建，如javadoc和source等，一般用于指定该jar包是基于哪个jdk版本构建的，classifier最终会加到jar包名字中，如<classifier>jdk15</classifier>，则最终jar包名会为artifact-version-jdk15.jar

引入依赖时，除了一般的groupId, artifactId, version外，可以有以下一些额外的配置。
+ scope：依赖的范围
+ optional：标记依赖是否可选，可选依赖被引入时无法传递
+ exclusions：用来排除传递性依赖

依赖范围：引入依赖时，可以使用```<scope></scope>```指定依赖范围。Maven在编译、测试、运行时都会指定不同的classpath。如下：
+ compile：编译依赖范围，默认的范围。此范围对编译、测试、运行时都有效。
+ test：测试依赖范围，使用此依赖范围的maven依赖，只对测试classpath有效。在编译主代码(非src/test/java)和运行项目时不会引入此依赖。如Junit。如果在src/main/java中使用了junit的类，则报无法找到该类。
+ provided：已提供依赖范围，对于编译和测试classpath有效，对运行时classpath无效。如servlet-api这些tomcat等容器已提供，不需要maven再引入了。
+ runtime：运行时依赖范围，对于测试和运行时有效。如Jdbc驱动，项目编译时使用的是接口，不需具体实现。
+ system：与provided类似，但system不是引用的远程仓库，而是引用本地路径，使用system时需要配置一个本地要导入的jar包路径。

依赖传递：maven引入依赖时，会自动引入该依赖对应的依赖。如果b，c依赖都引入了d依赖的不同版本，则优先引入依赖链较短的d依赖版本。若依赖链等长，则优先引入pom.xml中排在前面的b依赖对应的d依赖版本

依赖归类：正如代码中相同的字面量要提取出一个常量名一般，maven中使用属性值来统一管理版本号

## 仓库

私服：私服是一种特殊的远程仓库，当maven需要下载构件时，它请求私服，如果私服上不存在该构件，则从外部的远程仓库下载，缓存在私服之上，再为maven的下载请求提供服务。此外，一些无法从外部仓库下载到的构件，也能上传到私服供局域网内的用户使用。


远程仓库的配置和认证：可以在pom文件中配置<repositories>结点配置远程仓库，但如果远程仓库需要用户名和密码认证，则同时需要在settings.xml文件中配置<server>结点配置用户名和密码。settings.xml中<server>下的id必须与<repositories>下的id一致。

部署到私服：可以使用mvn命令部署构件到私服，需在pom文件中配置<distributionManagement>结点，该结点配置了发布版和快照版两个仓库。快照版本的构件会自动部署到快照仓库。

快照版本：为什么要有快照版本的构件？协同开发时，如果使用快照版本，maven会自动为快照版本的构件添加时间戳，如artifactId-version-20200101-221414-13.jar表示2020年1月1日22点14分14秒的第13次快照。根据maven的更新策略<updatePolicy>默认每天自动更新快照版本到本地仓库，如果使用稳定版，则如果本地仓库中存在该版本的构件，是不会自动更新构件的。

maven从仓库解析依赖：maven所有的仓库下的构件目录，都包含了表示当前构件信息的maven-metadata.xml文件。当使用快照版本时，maven会对比所有仓库下的maven-metadata.xml信息，选出最新的快照下载到本地使用。

镜像：在settings.xml配置<mirror>结点，其中<mirrorOf>指向国外网速较慢的仓库id，如中央仓库central，所有对于central的访问都重定向到了这个mirror。

## 生命周期与插件

maven的生命周期是抽象的，其实际行为都用插件来完成。

生命周期：在maven出现之前，项目构件的生命周期就已经存在，软件开发人员每天对项目进行清理、编译、测试和部署。maven的生命周期抽象了构建的各个步骤，定义了它们的次序，但没有提供具体实现，而是由插件来实现。

三套生命周期：
+ clean：清理项目
+ default：构建项目
+ site：建立项目站点

插件目标：一个插件能执行很多同类的功能，如maven-dependency-plugin，它能够分析项目依赖、列出项目依赖树等等。所有的这些功能，表示为一个插件目标，maven-dependency-plugin有10几个目标，如dependency:analyze等。

插件绑定：maven的生命周期的阶段是和插件目标相互绑定的。default生命周期的很多阶段是默认和插件绑定的，有些阶段没有绑定任何插件，因此没有任何实际行动。

自定义绑定：用户可以在<build>结点下的<plugin>下执行指定某插件绑定到具体的生命周期。

插件配置：有两种方式
+ 命令行：执行maven命令时还可以带参数，-D加参数即可。如执行mvn package时maven会执行单元测试，maven-surefire-plugin提供了maven.test.skip参数可以跳过单元测试，如```mvn package -Dmaven.test.skip=true```。-D是java命令自带的。
+ pom配置：在<build><plugins><plugin><configuration>结点下进行配置。

从命令行调用插件：插件是生命周期的具体实现，当然可以直接使用插件来进行项目构建了。如mvn dependency:tree调用maven-dependency-plugin的tree目标，其实该命令的全写法为：mvn org.apache.maven.plugins:maven-dependency-plugin:2.1:tree，为了使得命令更简介，maven引入了目标前缀的概念，dependency是maven-dependency-plugin的前缀。

## 插件解析机制

插件和依赖一样，也有自己的远程仓库，默认是中央仓库。当插件时官方插件时，即groupId为org.apache.maven.plugins时，则可以省略groupId。当使用插件前缀时，maven会搜索仓库元数据下的配置groupId/maven-metadata.xml文件，解析得到插件前缀对于的全名。如下图所示：clean前缀对应的插件就是maven-clean-plugin

![1ZQxT1.png](https://s2.ax1x.com/2020/01/24/1ZQxT1.png)

## 聚合(多模块)与继承

聚合(多模块)：maven可以允许构建一个pom工程，管理其下的多个模块，对主工程的命令会跑到其他的各个子模块中运行。

继承：类似聚合模块需要构建一个pom工程，继承也需要一个pom工程来使其他子模块继承。

父工程依赖管理：根据父工程元素可以继承到子项目的特点，我们可以将子项目中相同的依赖提取到父工程中，但会带来一个问题，即以后新加入的工程都会引入这些依赖。因此maven提供了<dependencyManagement>依赖管理来解决这个问题。

父工程插件管理： 同上，父工程依赖管理。

聚合和继承的区别：都是pom工程，除了pom.xml没有其他内容。聚合工程知道其下的所有子项目，子项目不知道聚合工程的存在。而继承工程不知道其被哪个子项目继承了，子项目却知道其继承自哪个父项目。

一般将父工程设置为聚合和继承项目，融合聚合和继承。

约定优于配置：maven约定了目录的结构，如：
+ 源码目录：src/main/java
+ 编译输出目录：target/classes

这样的好处是减少了配置，不用去通过配置告诉maven我的源码目录在什么地方。那么，在maven中约定是怎么体现的呢？他就是超级pom，所有的maven项目都继承自超级pom，故超级pom就可以认为是maven的约定。

反应堆(Reactor)：在多模块的项目中，反应堆指的是所有模块组成的构建结构。因为多模块间存在着继承和聚合关系。maven提供了命令行指令进行反应堆裁剪，即只构建某个模块，mvn compile -pl <模块名>

## 使用Nexus创建私服

无

## 使用Maven进行测试

maven使用maven-surefire-plugin插件运行测试用例，src/test/java下的以Test、TestCase结尾的类，maven会自动运行它们。

可以使用maven package -D skipTests命令跳过单元测试。使用maven package -D maven.test.skip=true连测试代码的编译也跳过了。

测试报告：surefire插件默认在target/surefire-report目录中输出文本型和xml型的测试报告。前者是给人看，后者给工具解析，如eclipse、jenkins可以解析xml格式的文件。

测试覆盖率报告：cobertura

## 使用hudson进行集成测试

建议阅读jenkins相关的书

## 使用Maven构建web应用

本章节主要讲web应用的war包部署，目前流行的springboot编写web应用都是jar包格式了，本章较落后。本章可以跳过不看。

## 版本管理

无

## 灵活的构建

Maven属性：
+ 自定义属性，如<properties>标签
+ 内置属性，如${basedir}代表pom.xml所在根目录
+ pom属性：如${project.artifacted}代表pom.xml下的<project><artifacted>标签的值
+ settings属性，应用settings.xml文件下的值，如${settings.localRepository}
+ Java系统属性，如${user.home}，可以使用mvn help:system查看该类属性
+ 环境变量属性：如${env.JAVA_HOME}

maven profile：跟springboot的多环境配置profile类似，一般开发都是使用springboot的profile功能，maven这点功能可以不看。

## 生成项目站点

无

## m2eclipse

本章是eclipse插件的使用，现在基本都是用idea了，可以跳过不看。

## 编写Maven插件

插件项目也是maven工程，打包类型<packaging>不是jar而是maven-plugin。需要继承AbstractMojo类并实现其execute方法。

## Archetype

Maven Archetype可以快速生成项目骨架，可以将其理解为Maven项目的模板。例如maven-archetype-quickstart就是最简单的Maven项目模板，其只提供基本的元素（如groupId, artifactId, version等）。很多著名的项目都提供了Archetype方便用户创建项目。

maven-archetype-plugin：主要的ide都集成了该插件，mvn命令为mvn archetype:generate

常用的archetype：
+ maven-archetype-quickstart：默认的，构建了一个基本的maven目录骨架
+ maven-archetype-webapp：包含src/main/webapp目录

编写archetype：略
