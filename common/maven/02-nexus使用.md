# 02-nexus3使用

下载地址：https://help.sonatype.com/repomanager3/download

本版本使用的是：3.30.1

官方文档地址：https://help.sonatype.com/repomanager3

## 1. 基础配置

解压文档后有两个目录：nexus-3.30.1-01、sonatype-work

默认配置在 nexus-3.30.1-01/etc/nexus-default.properties

可以修改相关配置

### 1. 基本命令

基础命令：nexus.exe/run  strop  restart force-reload status

构建服务：nexus.exe /install <optional-service-name>

```
nexus.exe /start <optional-service-name>
nexus.exe /stop <optional-service-name>
nexus.exe /uninstall <optional-service-name>
```

建议不自定义名称，不然后面命令都需要自定义名称

> ​	初次启动的时候会运行是否可以匿名获取jar

### 2. 基础配置

```properties
## DO NOT EDIT - CUSTOMIZATIONS BELONG IN $data-dir/etc/nexus.properties
##
# Jetty section
application-port=8081 
application-host=0.0.0.0
nexus-args=${jetty.etc}/jetty.xml,${jetty.etc}/jetty-http.xml,${jetty.etc}/jetty-requestlog.xml
nexus-context-path=/

# Nexus section
nexus-edition=nexus-pro-edition
nexus-features=\
 nexus-pro-feature

nexus.hazelcast.discovery.isEnabled=true

```



### 3. repository添加

地址：http://url:port/#admin/repository/repositories

添加maven2(proxy)

> ​	proxy代理、hosted 本地  group 分组几个repo合并

常用maven地址：

spring:https://repo.spring.io/libs-milestone

aliyun(central仓和jcenter仓的聚合仓): https://maven.aliyun.com/repository/public

### 4.增加可以deploy的角色

| 权限                             | 说明     |
| -------------------------------- | -------- |
| nx-repository-view-maven2-*-edit | 必须     |
| nx-repository-view-maven2-*-add  | 可以添加 |
| 匿名的权限                       |          |



## 2.使用

### 1. setting修改

```xml
 <mirror>
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <name>nexus</name>
      <url>http://url:port/repository/maven-public</url><!--maven-public 需要自己配置repo-->
    </mirror>
<!-- 如果需要deploy -->
<server>
		  <id>nexus</id>
		  <username>username</username>
		  <password>pwd</password>
		</server>
```

### 2. pom修改

```xml
<!-- 远程地址 -->
<distributionManagement>
        <repository>
            <id>nexus</id> <!-- setting serverID -->
            <name>Internal Releases</name>
            <url>http://url:port/repository/maven-releases</url>
        </repository>
        <snapshotRepository>
            <id>nexus</id>
            <name>Internal Releases</name>
            <url>http://url:port/repository/maven-snapshots</url>
        </snapshotRepository>
    </distributionManagement>

		<!-- 项目资源文件并拷贝到输出目录 -->
<!-- 编译 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
            </plugin>
<!-- 上传 deploy -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-deploy-plugin</artifactId>
                <version>2.8.2</version>
            </plugin>
<!-- deploy源码 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <version>3.2.1</version>
                <configuration>
                    <attach>true</attach>
                </configuration>
                <executions>
                    <execution>
                        <phase>compile</phase>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```

多profiles使用使用 大概这个模式

```xml
 <profiles>
        <profile>
            <id>dev</id>
            <properties>
                <!-- 环境标识，需要与配置文件的名称相对应 -->
                <profiles.active>dev</profiles.active>
            </properties>
            <activation>
                <!-- 默认环境 -->
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
        <profile>
            <id>test</id>
            <properties>
                <profiles.active>test</profiles.active>
            </properties>
        </profile>
        <profile>
            <id>prod</id>
            <properties>
                <profiles.active>prod</profiles.active>
            </properties>
        </profile>
    </profiles>
```

加上plugins

```xml
<profiles>
        <profile>
            <id>gen-src+doc</id>
            <activation>
                <property>
                    <name>performRelease</name>
                    <value>true</value>
                </property>
            </activation>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-source-plugin</artifactId>
                        <version>3.2.1</version>
                        <executions>
                            <execution>
                                <id>attach-sources</id>
                                <goals>
                                    <goal>jar</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-javadoc-plugin</artifactId>
                        <version>3.2.0</version>
                        <executions>
                            <execution>
                                <id>attach-javadoc</id>
                                <goals>
                                    <goal>jar</goal>
                                </goals>
                            </execution>
                        </executions>
                        <configuration>
                            <source>8</source>
                            <show>protected</show>
                            <charset>UTF-8</charset>
                            <encoding>UTF-8</encoding>
                            <docencoding>UTF-8</docencoding>
                            <additionalJOptions>
                                <additionalJOption>--no-module-directories</additionalJOption>
                                <additionalJOption>-quiet</additionalJOption>
                                <additionalJOption>-J-Duser.language=en</additionalJOption>
                                <additionalJOption>-J-Duser.country=US</additionalJOption>
                                <additionalJOption>-Xdoclint:none</additionalJOption>
                            </additionalJOptions>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        </profile>
        <profile>
            <id>gen-code-cov</id>
            <activation>
                <property>
                    <name>env.TRAVIS</name>
                    <value>true</value>
                </property>
            </activation>
            <build>
                <plugins>
                    <plugin>
                        <!-- for codecov.io -->
                        <!-- config example: https://github.com/codecov/example-java -->
                        <!-- plugin docs: http://eclemma.org/jacoco/trunk/doc/ -->
                        <groupId>org.jacoco</groupId>
                        <artifactId>jacoco-maven-plugin</artifactId>
                        <version>0.8.6</version>
                        <executions>
                            <execution>
                                <goals>
                                    <goal>prepare-agent</goal>
                                </goals>
                            </execution>
                            <execution>
                                <id>report</id>
                                <phase>test</phase>
                                <goals>
                                    <goal>report</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
        <profile>
            <id>gen-sign</id>
            <activation>
                <property>
                    <name>performRelease</name>
                    <value>true</value>
                </property>
            </activation>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-gpg-plugin</artifactId>
                        <version>1.6</version>
                        <executions>
                            <execution>
                                <id>sign-artifacts</id>
                                <phase>verify</phase>
                                <goals>
                                    <goal>sign</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
        <profile>
            <id>gen-git-properties</id>
            <activation>
                <property>
                    <name>performRelease</name>
                    <value>true</value>
                </property>
            </activation>
            <build>
                <plugins>
                    <!--
                        Maven plugin which includes build-time git repository information into an POJO / *.properties).
                        Make your apps tell you which version exactly they were built from! Priceless in large distributed deployments.
                            https://github.com/ktoso/maven-git-commit-id-plugin
                    -->
                    <plugin>
                        <groupId>pl.project13.maven</groupId>
                        <artifactId>git-commit-id-plugin</artifactId>
                        <version>4.0.3</version>
                        <executions>
                            <execution>
                                <id>get-the-git-infos</id>
                                <goals>
                                    <goal>revision</goal>
                                </goals>
                            </execution>
                            <execution>
                                <id>validate-the-git-infos</id>
                                <goals>
                                    <goal>validateRevision</goal>
                                </goals>
                            </execution>
                        </executions>
                        <configuration>
                            <validationProperties>
                                <!-- verify that the current repository is not dirty -->
                                <validationProperty>
                                    <name>validating git dirty</name>
                                    <value>${git.dirty}</value>
                                    <shouldMatchTo>false</shouldMatchTo>
                                </validationProperty>
                            </validationProperties>
                            <generateGitPropertiesFile>true</generateGitPropertiesFile>
                            <generateGitPropertiesFilename>${project.build.outputDirectory}/META-INF/scm/${project.groupId}/${project.artifactId}/git.properties</generateGitPropertiesFilename>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        </profile>
        <profile>
            <id>force-jdk11-when-release</id>
            <activation>
                <property>
                    <name>performRelease</name>
                    <value>true</value>
                </property>
            </activation>
            <build>
                <plugins>
                    <!--
                        add maven-enforce-plugin to make sure the right jdk is used
                        https://stackoverflow.com/a/18420462/922688
                    -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-enforcer-plugin</artifactId>
                        <version>3.0.0-M3</version>
                        <executions>
                            <execution>
                                <id>enforce-jdk-versions</id>
                                <goals>
                                    <goal>enforce</goal>
                                </goals>
                                <configuration>
                                    <rules>
                                        <requireJavaVersion>
                                            <version>11</version>
                                        </requireJavaVersion>
                                    </rules>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
```

