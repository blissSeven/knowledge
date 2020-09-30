# Thrif- [Thrift](#thrift)
  - [install](#install)
    - [requirements](#requirements)
    - [build](#build)  
http://thrift.apache.org/
## install
### requirements
* Java 1.7+
* Apache Ant
  * https://ant.apache.org/manual/index.html
###  build 
* 步骤http://thrift.apache.org/docs/BuildingFromSource
  * 确保有权限 sudo chown -R bliss ./thrift-0.13.0
  * sudo 下找不到java 可用sudo -E 代替sudo
 ```bash
  ./configure
        Building Java Library ........ : yes
            Java Library:
        Using gradlew ............. : lib/java/gradlew
        Using java ................ : java
        Using javac ............... : javac
        Using Gradle version ...... : Gradle 5.6.2
        Using java version ........ : java version "1.8.0_261"
 ```