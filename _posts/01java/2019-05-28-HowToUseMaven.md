---
layout: post
title: HowToUseMaven
category: Java
tags: Maven
keywords: Maven 使用 运作逻辑 概念 配置 
description: 
---
## 1.Maven的应用场景
Maven是一个Java平台下项目管理及自动构建工具,抽象出了Java平台下的软件工程构建的标准生命周期，并提供了提供了各个生命周期下的自动化工具。
## 2.Maven的核心概念
### 2.1 构建的生命周期（标准顺序流程）
Maven对生命周期的抽象是流程顺序化的，也就是说你要进行到某个周期的某个阶段，那么这个阶段之前的全部阶段都会被执行.
#### 2.1.1 clean
清理周期的目的在于去除掉上次构建生成的资源，比如的.class等等，保证构建最后使用的输出资源是当前工作空间内的处理输出资源；包含的主要流程：clean
#### 2.2.2 default
默认周期包含了的主要流程：校验（validate)，编译(compile)，测试(test)，打包(package)，验证（verify），安装到本地仓库（install），发布到远程仓库（deploy）。
#### 2.2.3 site
也叫站点生成周期，主要包括两个流程：第一生成工程的文档，第二部署到服务器上去；
### 2.2 插件
Maven规定了构建周期的阶段执行顺序及某个阶段应该干什么的标准，也就是完成了对构建流程的抽象并制定了流程的顺序标准，那么这些标准的实现者是谁呢？插件（plugin），插件具体的完成每个流程每个阶段的具体的工作。
#### 2.2.1 目标（goal）
一个插件可以有多种功能，每种功能称之为一个目标（goal）；比如compile插件可以支持编译功能代码（goal-compile）和 编译测试代码（goal-testCompile）;
#### 2.2.2 功能与生命周期的绑定
比如下面的编译插件提供了compile和testCompile两个功能，并分别绑定到了default声明周期的<phase>compile</phase>以及<phase>test-compile</phase>；

这就是告诉Maven工具在执行到<phase>compile</phase>和<phase>test-compile</phase>阶段时，要去调用maven-compiler-plugin的compile或者testCompile功能。
```xml
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>2.5.1</version>
    <executions>
        <execution>
	        <id>default-compile</id>
	        <phase>compile</phase>
	        <goals>
		        <goal>compile</goal>
	        </goals>
	        <configuration>
		        <source>1.8</source>
		        <target>1.8</target>
		        <encoding>UTF-8</encoding>
	        </configuration>
        </execution>
        <execution>
	        <id>default-testCompile</id>
	        <phase>test-compile</phase>
	        <goals>
		        <goal>testCompile</goal>
	        </goals>
	        <configuration>
		         <source>1.8</source>
		         <target>1.8</target>
		         <encoding>UTF-8</encoding>
	        </configuration>
        </execution>
    </executions>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <encoding>UTF-8</encoding>
    </configuration>
</plugin>
```
#### 2.2.3 mvn命令的基本格式
##### 2.2.3.1 指定声明周期形式
`mvn test`:意思是告诉mvn执行到<phasese>test</phasese>，那么在之前的所有阶段及每个阶段绑定的每一个goal都会被执行；
##### 2.2.3.2 具体执行插件的功能：
`mvn compiler:compile`:意思是告诉mvn 值执行这个compiler插件的compile功能，注意这样只会执行这个插件的这个功能，其他插件的功能不不会执行；
这里补充一些插件名解析的规则，对于一个插件的名字K，maven会把这个插件名解析成maven-K-plugin（maven自己提供的默认插件）及K-maven-plugin（第三方自定义插件）,并从当前工程有效pom.xml的去寻找artifactId（请参看下面的Jar的坐标）为maven-K-plugin或者K-maven-plugin的插件，K就是插件的别名，
#### 2.2.4 生命周期类型
Maven为不同的类型的工程的每个阶段提供了默认的插件来实现**约定**的功能。
（工程的类型由pom文件的<packaging></packaging> 指定，一般为jar、war、pom==多 module的情况下）
### 2.3 Jar的坐标
基本上在每一个工程中都不可避免的会加入多个jar作为lib，而这些lib的中jar的版本兼容性是一个令人相当头痛的问题：A.jar依赖C.jar,B.jar也依赖C.jar,但是依赖的分别是C的不同版本。

一个两个还好可以上三者官网上看一下解决下冲突，如果很多这样的情况，是不是很想死？甚至如果你不是很熟悉每个Jar的话，有冲突你甚至都发现不了。

最好出现一个约定Jar版本管理系统，自动的解决或者提示冲突在哪里。Maven提供这样一个系统（包含一个内置的中央仓库和Jar的坐标系）来解决这个问题：

每个Jar必须声明自己的x（groupId:组织Id）y（artifactId:工程Id）z(version:工程版本)三个坐标，使用的依赖也必须明确的声明其xyz坐标，这样Maven就可以汇总查看整个系统具体使用哪个Jar的哪个版本，如果出现了同Jar不同版本（同XY不同Z）的情况就报告冲突。
#### 2.3.1 冲突的解决机制
这个以后再补充,我一般在冲突的dependency里面直接exclusion掉，然后重新指定一个较高级的版本。
### 2.4 仓库
#### 2.4.1 远程仓库
也就是保存全部的Jar的服务器，maven自己提供了一个中央仓库，但一般企业开发中都会自己搭建nexus私服。
#### 2.4.2 本地仓库
也就是下面setting.xml里面指定的本地路径。
## 3.Setting的解释及解析
```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
   <!--指定你本地仓库的位置-->
   <localRepository>E:\JarLib</localRepository>
   <!--是否启用交互模式:默认true;交互模式是指当maven需要你的输入时会读取你在console的输入的数据-->
   <!--
   <interactiveMode>true</interactiveMode>
   -->
   <!-- offline是否在离线模式运行？默认false-->
   <offline>false</offline>
   <!--插件组的Id：plugin 的groupId,内置了 "org.apache.maven.plugins" and "org.codehaus.mojo" ,不需要使用自定插件的人员无需修改-->
   <pluginGroups>
    <!-- 
    <pluginGroup>com.your.plugins</pluginGroup>
    -->
   </pluginGroups>
   <!-- 网络代理：默认使用第一个，实际上基本不需要；比如你要去某个特定的仓库下载Jar而这个仓库又不在公网需要通过代理去下载，这里配置代理的安全密码验证的东西-->
   <proxies>
    <!--  
    <proxy>
      <id>optional</id>
      <active>true</active>
      <protocol>http</protocol>
      <username>proxyuser</username>
      <password>proxypass</password>
      <host>proxy.host.net</host>
      <port>80</port>
      <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
    </proxy>
    -->
   </proxies>
   <!--访问某些服务器的凭据账户密码或者sshkey等，注意这些Id应该和仓库的Id一样,实际上Maven也只会访问这些服务器-->
   <servers>
      <server>
         <id>nexus-private-repo</id>
         <username>admin</username>
         <password>adminpass</password>
      </server>
   </servers>
   <!--远程仓库的镜像,如果从某个仓库下载Jar下不下来的时候，比如网络太垃圾，如果存在这个仓库的镜像maven则会转到这个镜像来下载。-->
   <mirrors>
    <!--注意mirrodId 必须unique-->
	<!--
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
	-->
   </mirrors>
   <!-- 配置项组,在满足某些条件时激活,配置组中定义的配置会增加到Pom.xml中,这些配置项会被Pom中定义的各个插件使用。一般的都会配置各个工程通用的项目配置及保密的配置等等-->
   <profiles>
      <profile>
         <id>nexus-private-repo</id>
         <repositories>
            <!--比如这个仓库的位置-->
            <repository>
               <id>nexus-private-repo</id>
               <name>nexus-private-repo</name>
               <url>http://127.0.0.1:8090/repository/maven-snapshots</url>
               <releases>
                  <enabled>false</enabled>
               </releases>
               <snapshots>
                  <enabled>true</enabled>
               </snapshots>
            </repository>
         </repositories>
      </profile>
   </profiles>
   <!-- 激活的profile,会被所有的工程继承使用-->
   <activeProfiles>
      <activeProfile>nexus-private-repo</activeProfile>
   </activeProfiles>
</settings>
```
## 4.pom.xml的一些基本的解释
```xml
<?xml version="1.0" encoding="utf-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
  <!--必须声这一项-->  
  <modelVersion>4.0.0</modelVersion>  
  <!--多module开发时指定的具体的父POM,父POM的配置大部分都会被继承-->  
  <parent> 
    <groupId>com.aruforce</groupId>  
    <artifactId>mvn-test</artifactId>  
    <version>1.0.0-RELEASE</version> 
  </parent>  
  <!--下面设定自己的坐标系-->  
  <groupId>com.aruforce</groupId>  
  <artifactId>common</artifactId>  
  <version>1.0.1-SNAPSHOT</version>  
  <!--设定自己的工程类型-->  
  <packaging>Jar</packaging>  
  <!--一些配置项目，可以使用${tag}获取值，注意父POM的中的定义项目不会被继承下来-->  
  <properties> 
    <jdk.version>1.8</jdk.version>  
    <maven.compiler.encoding>UTF-8</maven.compiler.encoding>  
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding> 
  </properties>  
  <!--jar deploy的远程仓库位置-->  
  <distributionManagement> 
    <repository>
      <!--RELEASE包发布地址-->  
      <id>nexus-private-repo</id>  
      <name>Fruit Releases</name>  
      <url>http://127.0.0.1:8090/repository/maven-releases</url> 
    </repository>  
    <snapshotRepository>
      <!--snapshot包发布地址-->  
      <id>nexus-private-repo</id>  
      <name>Fruit Snapshots</name>  
      <url>http://127.0.0.1:8090/repository/maven-snapshots</url> 
    </snapshotRepository> 
  </distributionManagement>  
  <!--共用的依赖管理,如果在下面的以来配置中修改其中的配置的话(主要是版本),就是管理中指定的版本-->  
  <dependencyManagement> 
    <dependencies> 
      <dependency> 
        <groupId>junit</groupId>  
        <artifactId>junit</artifactId>  
        <version>4.12</version>  
        <scope>test</scope>  
        <!--
		插件的有效级别：System，provide，test，compile，runtime等等，涉及到插件的依赖传递，默认值是compile
		compile:编译范围依赖在所有的classpath中可用，同时它们也会被打包;
		provided:provided 依赖只有在当JDK 或者一个容器已提供该依赖之后才使用.已提供范围的依赖在编译classpath（不是运行时）可用。它们不是传递性的，也不会被打包。
		runtime：runtime 依赖在运行和测试系统的时候需要，但在编译的时候不需要。比如，你可能在编译的时候只需要JDBC API JAR，而只有在运行的时候才需要JDBC驱动实现。
		test:test范围依赖 在一般的编译和运行时都不需要，它们只有在测试编译和测试运行阶段可用。
		system:system范围依赖与provided 类似，但是你必须显式的提供一个对于本地系统中JAR 文件的路径。不要使用它。
		optional：可选的,classpath有没有都行。
		--> 
      </dependency>  
      <dependency> 
        <groupId>com.aruforce</groupId>  
        <artifactId>dependA</artifactId>  
        <version>1.0.0-RELEASE</version>  
        <scope>compile</scope>
        <!--覆盖上面的配置-->  
        <exclusions> 
          <exclusion> 
            <groupId>com.aruforce</groupId>  
            <artifactId>ABdependC</artifactId>
            <!--依赖1.2.0版本--> 
          </exclusion> 
        </exclusions> 
      </dependency>  
      <dependency> 
        <groupId>com.aruforce</groupId>  
        <artifactId>dependB</artifactId>  
        <version>1.0.0-RELEASE</version>  
        <scope>compile</scope>
        <!--覆盖上面的配置-->  
        <exclusions> 
          <exclusion> 
            <groupId>com.aruforce</groupId>  
            <artifactId>ABdependC</artifactId>
            <!--依赖1.0.0版本--> 
          </exclusion> 
        </exclusions> 
      </dependency>  
      <dependency> 
        <groupId>com.aruforce</groupId>  
        <artifactId>ABdependC</artifactId>  
        <version>1.2.0-RELEASE</version>
        <!---暂且先试下1.2.0版本如果版本不行就换其他的，都不行git fork modify deploy到nexus 重新引入-->  
        <scope>compile</scope> 
      </dependency> 
    </dependencies> 
  </dependencyManagement>  
  <dependencies> 
    <dependency> 
      <groupId>junit</groupId>  
      <artifactId>junit</artifactId>  
      <version>4.12</version>  
      <scope>compile</scope>
      <!--覆盖上面的配置--> 
    </dependency> 
  </dependencies>  
  <!--声明周期的相关配置-->  
  <build> 
    <!--构建过程的通用配置项目-->  
    <resources> 
      <resource> 
        <filtering>${pfile}</filtering>  
        <directory>src/main/resources</directory> 
      </resource> 
    </resources>  
    <testResources> 
      <testResource> 
        <filtering>false</filtering>  
        <directory>src/test/resources</directory> 
      </testResource> 
    </testResources>  
    <finalName>${artifactId}-${version}</finalName>  
    <filters> 
      <filter>${profile}</filter> 
    </filters>  
    <plugins> 
      <plugin> 
        <!--关于插件的配置项，可以去插件的官方网站查看使用说明，实际上很少几乎没有，长配置的只有编译插件，及reources插件（主要是根据filrters里面的配置在打包前处理源配置文件生成不同环境需要的配置）-->  
        <artifactId>maven-compiler-plugin</artifactId>  
        <version>2.5.1</version>  
        <configuration> 
          <source>1.8</source>  
          <target>1.8</target>  
          <encoding>UTF-8</encoding> 
        </configuration> 
      </plugin>  
      <plugin> 
        <artifactId>maven-jar-plugin</artifactId>  
        <version>2.3</version> 
      </plugin> 
    </plugins> 
  </build>  
  <!--不同配置项 mvn excutable -P profileId 这个profile里面的配置会生效-->  
  <profiles> 
    <profile> 
      <id>dev</id>  
      <activation> 
        <activeByDefault>true</activeByDefault> 
      </activation>  
      <properties> 
        <pfile>dev.properties</pfile> 
      </properties> 
    </profile>  
    <profile> 
      <id>test</id>  
      <activation> 
        <activeByDefault>false</activeByDefault> 
      </activation>  
      <properties> 
        <pfile>test.properties</pfile> 
      </properties> 
    </profile> 
  </profiles> 
</project>
```
## 5. 总结

1. 使用是够了,但是高级点的需要自己写插件的部分还没遇到；
2. 插件的配置相关内容不太应该写在这里而且也说明怎么去查找依赖的版本版本冲突排解方法也够简单粗暴。
3. 至于使用说明，读完也基本上有感觉，自己试着摆弄几个类型的工程吧 jar->war->多module;基本上也就这个学习使用流程吧
4. 剩下的私服搭建 一般就是nexus了,这个后期补充;

## 6. 感受
1. 开始学习使用maven的时候弄不清楚概念确实很难摸清楚怎么使用;
2. 学完后发现maven这些也是解决特定问题,知道Maven要解决什么问题,知道运作逻辑之后也就简单了