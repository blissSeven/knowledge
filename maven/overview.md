# MAVEN
- [MAVEN](#maven)
  - [依赖冲突](#依赖冲突)
  - [命令](#命令)
  - [mvn 参数](#mvn-参数)
  - [maven介绍](#maven介绍)
    - [依赖关系](#依赖关系)
      - [mvn 镜像](#mvn-镜像)
    - [构件流程](#构件流程)
      - [phase](#phase)
      - [goal](#goal)
    - [插件](#插件)
        - [内置插件](#内置插件)
      - [maven-compiler-plugin](#maven-compiler-plugin)
      - [maven-dependency-plugin](#maven-dependency-plugin)
      - [maven-jar-plugin](#maven-jar-plugin)
      - [maven-assembly-plugin](#maven-assembly-plugin)
        - [assembly 的goal](#assembly-的goal)
        - [assembly descriptor](#assembly-descriptor)
    - [模块管理](#模块管理)
    - [mvnw](#mvnw)
    - [发布artifact](#发布artifact)
    - [不同环境打包不同配置](#不同环境打包不同配置)
      - [不同的属性文件夹](#不同的属性文件夹)
      - [不同的属性](#不同的属性)
## 依赖冲突
`mvn dependency:tree`
## 命令
```shell
mvn -V, –show-version 显示版本信息后继续执行Maven其他目标;

mvn -h, –help 显示帮助信息; mvn -e, –errors 控制Maven的日志级别,产生执行错误相关消息;

mvn -X, –debug 控制Maven的日志级别,产生执行调试信息;

mvn -q, –quiet 控制Maven的日志级别,仅仅显示错误;

mvn -Pxxx 激活 id 为 xxx的profile (如有多个，用逗号隔开);

mvn -Dxxx=yyy 指定Java全局属性;

mvn -o , –offline 运行offline模式,不联网更新依赖;

mvn -N, –non-recursive 仅在当前项目模块执行命令,不构建子模块;

mvn -pl, –module_name 在指定模块上执行命令;

mvn -ff, –fail-fast 遇到构建失败就直接退出;

mvn -fn, –fail-never 无论项目结果如何,构建从不失败;

mvn -fae, –fail-at-end 仅影响构建结果,允许不受影响的构建继续;

mvn -C, –strict-checksums 如果校验码不匹配的话,构建失败;

mvn -c, –lax-checksums 如果校验码不匹配的话,产生告警;

mvn -U 强制更新snapshot类型的插件或依赖库(否则maven一天只会更新一次snapshot依赖);

mvn -npu, –no-plugin-updates 对任何相关的注册插件,不进行最新检查(使用该选项使Maven表现出稳定行为，该稳定行为基于本地仓库当前可用的所有插件版本);

mvn -cpu, –check-plugin-updates 对任何相关的注册插件,强制进行最新检查(即使项目POM里明确规定了Maven插件版本,还是会强制更新);

mvn -up, –update-plugins [mvn -cpu]的同义词;

mvn -B, –batch-mode 在非交互（批处理）模式下运行(该模式下,当Mven需要输入时,它不会停下来接受用户的输入,而是使用合理的默认值);

mvn -f, –file 强制使用备用的POM文件; mvn -s, –settings 用户配置文件的备用路径;

mvn -gs, –global-settings 全局配置文件的备用路径;

mvn -emp, –encrypt-master-password 加密主安全密码,存储到Maven settings文件里;

mvn -ep, –encrypt-password 加密服务器密码,存储到Maven settings文件里;

mvn -npr, –no-plugin-registry 对插件版本不使用~/.m2/plugin-registry.xml(插件注册表)里的配置
```
```java
 <dependency>
            <groupId>org.apache.maven.surefire</groupId>
            <artifactId>maven-surefire-common</artifactId>
            <version>2.21.0</version>
            <exclusions>
                <!-- Exclude Commons Logging in favor of SLF4j -->
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-nop</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```
自动打包插件
```java
           <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <descriptors>
                        <descriptor>${project.basedir}/src/main/assembly/hadoop-job.xml</descriptor>
                    </descriptors>
                    <archive>
                        <manifest>
                            <mainClass>com.xiaomi.mico.stats.jobs.actDevice.ActDeviceEmailV2Job</mainClass>
                        </manifest>
                    </archive>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly-job</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```
## mvn 参数
```shell
mvn source:jar #生成source:jar
```
## maven介绍
正常情况下，需要一个包，需要把该包放到classpath目录下。  


maven 项目结构   
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2020/10/14/19-32-49-cef5566c3e5075a7a9de43bcce59df0c-20201014193249-7a6e3c.png)
```java
<project ...>
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.itranswarp.learnjava</groupId>
	<artifactId>hello</artifactId>
	<version>1.0</version>
	<packaging>jar</packaging>
	<properties>
        ...
	</properties>
	<dependencies>
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>
	</dependencies>
</project>
```
groupId类似java的包名，通常公司或组织名称。artifactld为java类名，通常项目名称+version。
### 依赖关系
|scope|desc|example|
|:-:|:-:|:-:|
|compile|编译时需要(default)|commons-logging|
|test|编译Test时需要|junit|
|runtime|编译时不需要，运行时需要|mysql|
|provided|编译时用到,运行时有JDK或服务器提供|servlet-api|
```java
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.48</version>
    <scope>runtime</scope>
</dependency>
```
mvn维护了一个中央仓库(repo1.maven.org)，对某个依赖，通过groupId，artifactld,version确定。  
#### mvn 镜像
.m2文件下settings.xml配置文件 ，通过https://search.maven.org/ 查找jar
```java
<settings>
    <mirrors>
        <mirror>
            <id>aliyun</id>
            <name>aliyun</name>
            <mirrorOf>central</mirrorOf>
            <!-- 国内推荐阿里云的Maven镜像 -->
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        </mirror>
    </mirrors>
</settings>
```
### 构件流程
#### phase
mvn有三套独立的生命周期Lifecycle   
* clean 清理 
* default 构件核心部分、编译、测试、打包、部署
* site 生成项目报告、站点、发布站点 

mvn的生命周期由一系列phase组成以内置的生命周期default为例，包含下列phrase
* validate
* initialize
* generate-sources
* process-sources
* generate-resources
* process-resources # 复制并处理资源文件，至目标目录，准备打包
* compile # 编译源代码
* process-classes
* generate-test-sources
* process-test-sources # 复制并处理资源文件，至目标测试目录；
* generate-test-resources
* process-test-resources
* test-compile
* process-test-classes
* test
* prepare-package
* package #接受编译好的代码，打包成可发布的格式，如 JAR ；
* pre-integration-test
* integration-test
* post-integration-test
* verify
* install #将包安装至本地仓库，以让其它项目依赖；
* deploy #将最终的包复制到远程的仓库，以让其它开发人员与项目共享；   

mvn package会执行default的生命周期，从头开始至package 这个phase为止  
clean 生命周期
* pre-clean
* clean  //phase
* post-clean 
```bash
mvn clean package 
# mvn后紧跟phase
```
#### goal
执行一个phase会触发一个或多个goal,goal命名以abc:xyz形式  
|phase|对应goal|
|:-:|:-:|
|complie|compiler:compiler|
|test|compiler:testCompile, surefire:test|
* lifestyle类似package，包含多个phase   
* phase 类似class，包含多个goal
* goal类似method，真正起作用

大都只要指定phase，就默认执行phase默认绑定的goal，少数情况可以指定goal。`mvn tomcat:run`
### 插件
执行每个phase，都是通过某个插件（plugin）来执行的，Maven本身其实并不知道如何执行compile，它只是负责找到对应的compiler插件，然后执行默认的compiler:compile这个goal来完成编译   
##### 内置插件
|插件|对应phase|
|:-:|:-:|
|clean|clean|
|compiler|compile|
|surefire|test|
|jar|package|
```xml 
<project>
  [...]
  <build>
    [...]
    <plugins>
      <plugin>
        <artifactId>maven-assembly-plugin</artifactId>
        <version>3.3.0</version>
        <configuration>
            <archive>
                <manifest>
                    <mainClass>
                        com.xiaomi.mico.stats.jobs.actDevice.ActDeviceEmailV2Job
                    </mainClass>
                </manifest>
            </archive>
          <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
          </descriptorRefs>
        </configuration>
        <executions>
          <execution>
            <id>make-assembly</id> <!-- this is used for inheritance merges -->
            <phase>package</phase> <!-- bind to the packaging phase -->
            <goals>
              <goal>single</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
      [...]
</project>
```
#### maven-compiler-plugin
指定项目源码的jdk版本，编译后的版本，以及编码   
常用配置
```xml
<build>
    <plugins>
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.0</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <encoding>UTF-8</encoding>
            </configuration>
        </plugin>
    </plugins>
</build>
```
详细配置    
```xml
<plugin>
    <!-- 指定maven编译的jdk版本,如果不指定,maven3默认用jdk 1.5 maven2默认用jdk1.3 -->
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.1</version>
    <configuration>
        <!-- 一般而言，target与source是保持一致的，但是，有时候为了让程序能在其他版本的jdk中运行(对于低版本目标jdk，源代码中不能使用低版本jdk中不支持的语法)，会存在target不同于source的情况 -->                    
        <source>1.8</source> <!-- 源代码使用的JDK版本 -->
        <target>1.8</target> <!-- 需要生成的目标class文件的编译版本 -->
        <encoding>UTF-8</encoding><!-- 字符集编码 -->
        <skipTests>true</skipTests><!-- 跳过测试 -->
        <verbose>true</verbose>
        <showWarnings>true</showWarnings>
        <fork>true</fork><!-- 要使compilerVersion标签生效，还需要将fork设为true，用于明确表示编译版本配置的可用 -->
        <executable><!-- path-to-javac --></executable><!-- 使用指定的javac命令，例如：<executable>${JAVA_1_4_HOME}/bin/javac</executable> -->
        <compilerVersion>1.3</compilerVersion><!-- 指定插件将使用的编译器的版本 -->
        <meminitial>128m</meminitial><!-- 编译器使用的初始内存 -->
        <maxmem>512m</maxmem><!-- 编译器使用的最大内存 -->
        <compilerArgument>-verbose -bootclasspath ${java.home}\lib\rt.jar</compilerArgument><!-- 这个选项用来传递编译器自身不包含但是却支持的参数选项 -->
    </configuration>
</plugin>
```
#### maven-dependency-plugin
复制项目依赖到指定文件夹  
* goal
  * copy 将配置在插件中的jar复制到指定位置
  * copy-dependencies 将项目pom指定的所有依赖及其传递依赖复制到指定位置
  * unpack 类似copy 但会将jar包解压缩
  * unpack-dependencies 类似copy-dependencies，但是会解压缩

copy指定依赖---将指定的junit复制到/libs
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>2.8</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>copy</goal>
            </goals>
            <configuration>
                <artifactItems>
                    <artifactItem>
                        <groupId>junit</groupId>
                        <artifactId>junit</artifactId>
                        <version>4.11</version>
                        <outputDirectory>${project.build.directory}/libs</outputDirectory>
                    </artifactItem>
                </artifactItems>
            </configuration>
        </execution>
    </executions>
</plugin>
```
unpack指定依赖---将slf4j复制到/lib文件夹，将junit复制到/libs文件夹，子内配置可以覆盖父类的配置
```xml
   <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <version>2.8</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>unpack</goal>
                    </goals>
                    <configuration>
                        <artifactItems>
                            <artifactItem>
                                <groupId>org.slf4j</groupId>
                                <artifactId>slf4j-log4j12</artifactId>
                                <version>1.7.7</version>
                            </artifactItem>
                            <artifactItem>
                                <groupId>junit</groupId>
                                <artifactId>junit</artifactId>
                                <version>4.11</version>
                                <outputDirectory>${project.build.directory}/libs</outputDirectory>
                            </artifactItem>
                        </artifactItems>
                        <outputDirectory>lib</outputDirectory>
                    </configuration>
                </execution>
            </executions>
        </plugin>
```
copy-dependencies---复制所有依赖项
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>3.1.1</version>
    <executions>
        <execution>
            <id>copy-dependencies</id>
            <phase>package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <!-- jar包存放位置 -->
                <outputDirectory>${project.build.directory}/alternateLocation</outputDirectory>
                <overWriteReleases>false</overWriteReleases>
                <overWriteSnapshots>false</overWriteSnapshots>
                <overWriteIfNewer>true</overWriteIfNewer>
            </configuration>
        </execution>
    </executions>
</plugin>
```
#### maven-jar-plugin
将项目打包为jar，设定 MAINFEST .MF文件的参数，比如指定运行的Main class、将依赖的jar包加入classpath中等等
```xml
<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.0.2</version>
    <configuration>  
        <archive>  
            <addMavenDescriptor>false</addMavenDescriptor>  
            <manifest>  
                <addClasspath>true</addClasspath>  
                <classpathPrefix>lib/</classpathPrefix>  
                <mainClass>com.meix.boot.Application</mainClass>  
            </manifest>
            <manifestEntries>  
                <Class-Path>./</Class-Path>  <!--将ojdbc8-1.0.jar写进MANIFEST.MF文件中的Class-Path-->
                <Class-Path>lib/ojdbc8-1.0.jar</Class-Path>
            </manifestEntries> 
        </archive>  
         <!-- 过滤掉不希望包含在jar中的文件  -->  
        <excludes>  
            <exclude>*.xml</exclude>  
            <exclude>spring/**</exclude>  
            <exclude>config/**</exclude>  
        </excludes> 
        <!-- 这里不做举例了 -->
        <includes>
            <include></include>
        </includes>         
    </configuration>  
</plugin>
```
#### maven-assembly-plugin
* maven-jar-plugin，默认的打包插件，用来打普通的project JAR包；
* maven-shade-plugin，用来打可执行JAR包，也就是所谓的fat JAR包；
* maven-assembly-plugin，支持自定义的打包结构，也可以定制依赖项等

##### assembly 的goal
* single 
* help
##### assembly descriptor 
* 内置
  * bin 类似默认打包，将bin目录下文件打包
  * jar-with-dependencies 将所有依赖解压到包中
  * src 只将src目录下文件打包
  * project将整个project资源打包

内置描述文件
```xml
 <plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <configuration>
        <archive>
            <manifest>
                <mainClass>DateTest.CalendarTest</mainClass>
            </manifest>
        </archive>
        <!-- 内置描述文件 -->
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
    </configuration>
    <executions>
        <execution>
            <id>make-assembly</id> <!-- this is used for inheritance merges -->
            <phase>package</phase> <!-- bind to the packaging phase -->
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```   
自定义描述文件---jar-with-dependencies
```xml
<assembly xmlns="http://maven.apache.org/ASSEMBLY/2.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/ASSEMBLY/2.0.0 http://maven.apache.org/xsd/assembly-2.0.0.xsd">
    <!-- TODO: a jarjar format would be better -->
    <id>jar-with-dependencies</id>
    <formats>
        <format>jar</format>
    </formats>
    <includeBaseDirectory>false</includeBaseDirectory>
    <dependencySets>
        <dependencySet>
            <outputDirectory>/</outputDirectory>
            <useProjectArtifact>true</useProjectArtifact>
            <!-- 将所有jar包解压 -->
            <unpack>true</unpack>
            <scope>runtime</scope>
        </dependencySet>
    </dependencySets>
</assembly>
```
* fileSets 指定要包含的文件集
* dependencySet 指定要包含的依赖
```xml
<assembly xmlns="http://maven.apache.org/ASSEMBLY/2.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/ASSEMBLY/2.0.0 http://maven.apache.org/xsd/assembly-2.0.0.xsd">
    <!-- TODO: a jar format would be better -->
    <id>own-assembly</id>
    <formats>
        <format>jar</format>
        <format>tar.gz</format>
        <format>zip</format>
    </formats>
<!--    是否生成和finalName文件夹，后续所有子目录基于此根目录-->
    <includeBaseDirectory>true</includeBaseDirectory>
    <fileSets>
<!--        将/target/classes 下文件 拷贝到/output下，基于上处basedirectory-->
        <fileSet>
            <directory>${project.build.directory}/classes</directory>
            <outputDirectory>/output</outputDirectory>
        </fileSet>
    </fileSets>
    <dependencySets>
        <dependencySet>
            <outputDirectory>/lib</outputDirectory>
            <useProjectArtifact>true</useProjectArtifact>
            <unpack>false</unpack>
            <!--将scope为runtime的依赖包打包-->
            <scope>runtime</scope>
        </dependencySet>
    </dependencySets>
</assembly>
```
### 模块管理
* 项目模块化划分  
![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2020/10/14/21-14-56-d6a082329a77f8c11d54c44be01c9bb7-20201014211456-eb2fee.png)    

子模块可以继承父的pol元素，parent的packing为pom而不是jar，parent不包含java代码，只是为了减少重复配置。  
* parent XML配置
```xml 
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itranswarp.learnjava</groupId>
    <artifactId>parent</artifactId>
    <version>1.0</version>
    <!--!!!!   parent的packing为pom而不是jar  -->
    <packaging>pom</packaging> 

    <name>parent</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <java.version>11</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.28</version>
        </dependency>
    </dependencies>
</project>
```
* 子项目 XML配置
```xml 
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.itranswarp.learnjava</groupId>
        <artifactId>parent</artifactId>
        <version>1.0</version>
        <!-- 引用parent pom文件 -->
        <relativePath>../parent/pom.xml</relativePath>
    </parent>

    <artifactId>module-a</artifactId>
    <packaging>jar</packaging>
    <name>module-a</name>
</project>
```
* 根目录XML配置
在根目录执行`mvn clean package`时，mvn根据pom找到包括parent的module，一次全部编译。   
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.itranswarp.learnjava</groupId>
    <artifactId>build</artifactId>
    <version>1.0</version>
    <!-- packaging 为 pom -->
    <packaging>pom</packaging>
    <name>build</name>

    <modules>
        <module>parent</module>
        <module>module-a</module>
        <module>module-b</module>
        <module>module-c</module>
    </modules>
</project>
```
### mvnw
可以针对特定的项目，安装指定版本的mvn。   
* 安装  
  * 在项目根目录(pom文件)，运行`mvn -N io.takari:maven:0.7.6:wrapper`,0.7.6e为Maven Wrapper版本，使用最新maven。https://github.com/takari/maven-wrapper 可以查看最新Maven Wrapper版本。  
  * `mvn -N io.takari:maven:0.7.6 :wrapper -Dmaven=3.3.3`安装 maven为3.3.3
  * ![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2020/10/14/21-31-50-63c8292c964a052c7f2fbe63597f689e-20201014213149-06dc63.png)
* 使用
  * `mvnw clean package`
  * `./mvnw clean package` linux/macos 下
### 发布artifact

### 不同环境打包不同配置
`mvn package -Pdev`
#### 不同的属性文件夹
**在src/main/resources目录下，新建dev、test、pro三个子目录，公共属性放在resources目录下**
```xml
<profiles>
    <profile>
        <!-- 本地开发环境 -->
        <id>dev</id>
        <properties>
            <profiles.active>dev</profiles.active>
        </properties>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <profile>
        <!-- 测试环境 -->
        <id>test</id>
        <properties>
            <profiles.active>test</profiles.active>
        </properties>
    </profile>
    <profile>
        <!-- 生产环境 -->
        <id>pro</id>
        <properties>
            <profiles.active>pro</profiles.active>
        </properties>
    </profile>
</profiles>
```
在build脚本下，配置资源文件的目录，首先指定通用的配置文件的目录，其次指定某个环境下的目录。在激活指定的profile时，会加载指定目录下的配置文件，如当前激活的是pro profile，那么这个资源目录就是src/main/resources/pro
```xml
<build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <!-- 资源根目录排除各环境的配置，防止在生成目录中多余其它目录 -->
                <excludes>
                    <exclude>test/*</exclude>
                    <exclude>pro/*</exclude>
                    <exclude>dev/*</exclude>
                </excludes>
            </resource>
            <resource>
                <directory>src/main/resources/${profiles.active}</directory>
            </resource>
        </resources>
    </build>
```
#### 不同的属性
配置不同环境下的属性
```xml
 <profile>
            <id>preview</id>
            <activation>
                <property>
                    <name>preview</name>
                    <value>true</value>
                </property>
            </activation>
            <properties>
                <configitem.path>internal/device_display_config/preview</configitem.path>
            </properties>
        </profile>
        <profile>
            <id>production</id>
            <activation>
                <property>
                    <name>production</name>
                    <value>true</value>
                </property>
            </activation>
            <properties>
                <configitem.path>internal/device_display_config</configitem.path>
            </properties>
        </profile>
```
build节点下设置资源路径
```xml
  <build>
        <resources>
            <resource>
                <directory>src/main/resources/</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
```
通过properties文件下的通配符实现不同环境下加载不同配置
```java
configitem.path = ${configitem.path}
name=${configitem.path}
```





