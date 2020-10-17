# MAVEN
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
内置插件
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
#### 模块管理
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
#### mvnw
可以针对特定的项目，安装指定版本的mvn。   
* 安装  
  * 在项目根目录(pom文件)，运行`mvn -N io.takari:maven:0.7.6:wrapper`,0.7.6e为Maven Wrapper版本，使用最新maven。https://github.com/takari/maven-wrapper 可以查看最新Maven Wrapper版本。  
  * `mvn -N io.takari:maven:0.7.6 :wrapper -Dmaven=3.3.3`安装 maven为3.3.3
  * ![](https://raw.githubusercontent.com/BlissSeven/image/master/java/2020/10/14/21-31-50-63c8292c964a052c7f2fbe63597f689e-20201014213149-06dc63.png)
* 使用
  * `mvnw clean package`
  * `./mvnw clean package` linux/macos 下
#### 发布artifact






