# MAVEN
## 依赖冲突
`mvn dependency:tree`
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