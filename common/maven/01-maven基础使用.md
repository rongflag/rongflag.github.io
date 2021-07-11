# 01-Maven基础使用

下载：https://maven.apache.org/download.cgi

本讲解使用的版本是3.8.1

## 1.基础命令

| 基础命令    | 说明                      |
| ----------- | ------------------------- |
| mvn clean   | 删除target/               |
| mvn package | 打包                      |
| mvn test    | test case junit/testNG    |
| mvn install | 把项目install到local repo |
| mvn deploy  | 发本地jar发布到remote     |
| mvn compile |                           |

参数

| 命令                   | 功能                                         |
| ---------------------- | -------------------------------------------- |
| -Dmaven.test.skip=true | 不执行测试用例，也不编译测试用例类。         |
| -Dmaven.test.skip      | 不但跳过单元测试的运行，也跳过测试代码的编译 |
|                        |                                              |

## 2. dependency详解

### 2.1 type

type的值一般有jar、war、pom等，声明引入的依赖的类型。

### 2.2 classifier

lassifier它表示在相同版本下针对不同的环境或者jdk使用的jar,如果配置了这个元素，则会将这个元素名在加在最后来查找相应的jar，例如：

```xml
<code><code><code>给相同的version，构建不同的环境使用依赖
<classifier>jdk17</classifier>
<classifier>jdk18</classifier></code></code></code>
```

### 2.3 optional

当project-A 依赖project-B, project-B 依赖project-D时

```xml
<code><code><code><dependency>
  <groupid>sample.ProjectD</groupid>
  <artifactid>ProjectD</artifactid>
  <version>1.0-SNAPSHOT</version>
  <optional>true</optional>
</dependency></code></code></code>
```

所以当project-B的true时, project-A中如果没有显式的引入project-D, 则project-A不依赖project-D, 即project-A可以自己选择是否依赖project-D
默认的值为false, 及子项目必须依赖。

### 2.4 scope

**scope的分类**

**compile**(默认)

> ​	默认就是compile，什么都不配置也就是意味着compile。compile表示被依赖项目需要参与当前项目的编译，当然后续的测试，运行周期也参与其中，是一个比较强的依赖。打包的时候通常需要包含进去。

**test**

> ​	scope为test表示依赖项目仅仅参与测试相关的工作，包括测试代码的编译，执行。比较典型的如junit。

**runntime**

> ​	runntime表示被依赖项目无需参与项目的编译，不过后期的测试和运行周期需要其参与。与compile相比，跳过编译而已，说实话在终端的项目（非开源，企业内部[系统](https://www.2cto.com/os/)）中，和compile区别不是很大。比较常见的如JSR×××的实现，对应的API jar是compile的，具体实现是runtime的，compile只需要知道接口就足够了。oracle jdbc驱动架包就是一个很好的例子，一般scope为runntime。另外runntime的依赖通常和optional搭配使用，optional为true。我可以用A实现，也可以用B实现。

**provided**

> ​	provided意味着打包的时候可以不用包进去，别的设施(Web Container)会提供。事实上该依赖理论上可以参与编译，测试，运行等周期。相当于compile，但是在打包阶段做了exclude的动作。

**system**

> ​	从参与度来说，也provided相同，不过被依赖项不会从maven仓库抓，而是从本地文件系统拿，一定需要配合systemPath属性使用。

**import**(only available in Maven 2.0.9 or later)

> ​	这个是maven2.0.9版本后出的属性，import只能在dependencyManagement的中使用，能解决maven单继承问题，import依赖关系实际上并不参与限制依赖关系的传递性。

使用import scope解决maven继承（单）问题

**scope的依赖传递**

> A–>B–>C。当前项目为A，A依赖于B，B依赖于C。知道B在A项目中的scope，那么怎么知道C在A中的scope呢？答案是：
>
> 当C是test或者provided时，C直接被丢弃，A不依赖C；
> 否则A依赖C，C的scope继承于B的scope。

### 2.5 systemPath

当maven依赖本地而非repository中的jar包，sytemPath指明本地jar包路径,例如：

```xml
<code><code><code><code><code><code><dependency>
    <groupid>org.hamcrest</groupid>
    hamcrest-core</artifactid>
    <version>1.5</version>
    <scope>system</scope>
    <systempath>${basedir}/WebContent/WEB-INF/lib/hamcrest-core-1.3.jar</systempath>
</dependency></code></code></code></code></code></code>
```



### 2.6 exclusions

依赖排除，就是有时候引入某一个依赖的时候，该依赖下有jar包冲突，可以排除掉，不引用该jar，例如：

```xml
<code><code><code><code><code><code>        <dependency>
            <groupid>test</groupid>
            test</artifactid>
            <version>1.0.2-SNAPSHOT</version>
            <exclusions>
                <exclusion>
                    <groupid>org.springframework</groupid>
                    spring</artifactid>
                </exclusion>
                <exclusion>
                    slf4j-log4j12</artifactid>
                    <groupid>org.slf4j</groupid>
                </exclusion>
            </exclusions>
        </dependency></code></code></code></code></code></code>
```

## 3. 冲突解决

### 1.查找依赖冲突

1. 依赖冲突常见的表现有NoSuchMethodError，ClassNotFoundException 等。
2. 在出现报错信息之后，我们可以通过出错信息快速定位到有问题的jar包。比如通过上面的报错信息可以知道出问题的是 org.springframework.core包。然后我们可以通过分析 pom文件尝试查找冲突的原因。
3. 如果pom文件不能轻易的看出来，还有几种方式来查看maven依赖冲突

1. 可以通过IDE自带的maven依赖管理工具进行查看。

IDEA -Maven Helper

2. 或者可以通过maven 命令 **dependency:tree -Dverbose**

如果在执行结果中出现 **omitted for conflict with** 这样的字样，就表示项目中存在依赖冲突

### 2.解决冲突

1. 发现了冲突的包之后，剩下的就是选择一个合适版本的包留下，如果是传递依赖的包正确，那么把显示依赖的包去掉。
2. 如果是某一个传递依赖的包有问题，使用**exclusions**那么我们需要手动把这个传递依赖去掉1

## 4.setting

### 1.整个setting结构

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
 | 官方文档: https://maven.apache.org/settings.html
 |
 | Maven 提供以下两种 level 的配置:
 |
 |  1. User Level.      当前用户独享的配置, 通常在 ${user.home}/.m2/settings.xml 目录下。 
 |                      可在 CLI 命令行中通过以下参数设置:  -s /path/to/user/settings.xml
 |
 |  2. Global Level.    同一台计算机上的所有 Maven 用户共享的全局配置。 通常在${maven.home}/conf/settings.xml目录下。
 |                      可在 CLI 命令行中通过以下参数设置:  -gs /path/to/global/settings.xml
 |
 |  备注:  User Level 优先级 > Global Level
 |-->

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <!--
     | Maven 依赖搜索顺序, 当我们执行 Maven 命令时, Maven 开始按照以下顺序查找依赖的库: 
     |
     | 步骤 1 － 在本地仓库中搜索, 如果找不到, 执行步骤 2, 如果找到了则执行其他操作。
     | 步骤 2 － 在中央仓库中搜索, 如果找不到, 并且有一个或多个远程仓库已经设置, 则执行步骤 4, 如果找到了则下载到本地仓库中已被将来引用。
     | 步骤 3 － 如果远程仓库没有被设置, Maven 将简单的停滞处理并抛出错误（无法找到依赖的文件）。
     | 步骤 4 － 在一个或多个远程仓库中搜索依赖的文件, 如果找到则下载到本地仓库已被将来引用, 否则 Maven 将停止处理并抛出错误（无法找到依赖的文件）。
     |-->

    <!-- 地仓库路径, 默认值: ${user.home}/.m2/repository -->
    <localRepository>${user.home}/workspace/env/maven/repository</localRepository>

    <!-- 当 maven 需要输入值的时候, 是否交由用户输入, 默认为true；false 情况下 maven 将根据使用配置信息进行填充 -->
    <interactiveMode>true</interactiveMode>

    <!-- 是否支持联网进行 artifact 下载、 部署等操作, 默认: false -->
    <offline>false</offline>

    <!-- 
     | 搜索插件时, 如果 groupId 没有显式提供时, 则以此处配置的 groupId 为默认值, 可以简单理解为默认导入这些 groupId 下的所有 artifact（需要时才下载）
     | 默认情况下该列表包含了 org.apache.maven.plugins和org.codehaus.mojo
     |
     | 查看插件信息: 
     |    mvn help:describe -Dplugin=org.apache.maven.plugins:maven-compiler-plugin:3.5.1 -Ddetail
     |-->
    <pluginGroups>

        <!-- plugin 的 groupId -->
        <!--
        <pluginGroup>com.your.plugins</pluginGroup>
        -->

    </pluginGroups>

    <!-- 进行远程服务器访问时所需的授权配置信息。通过系统唯一的 server-id 进行唯一关联 -->
    <servers>
        <server>
            <!-- 这是 server 的 id, 该 id 与 distributionManagement 中 repository 元素的id 相匹配 -->
            <id>server_id</id>

            <!-- 鉴权用户名 -->
            <username>auth_username</username>

            <!-- 鉴权密码 -->
            <password>auth_pwd</password>

            <!-- 鉴权时使用的私钥位置。和前两个元素类似, 私钥位置和私钥密码指定了一个私钥的路径（默认是/home/hudson/.ssh/id_dsa）以及如果需要的话, 一个密钥 -->
            <privateKey>path/to/private_key</privateKey>

            <!-- 鉴权时使用的私钥密码, 非必要, 非必要时留空 -->
            <passphrase>some_passphrase</passphrase>

            <!-- 
             | 文件被创建时的权限。如果在部署的时候会创建一个仓库文件或者目录, 这时候就可以使用权限（permission）
             | 这两个元素合法的值是一个三位数字, 其对应了unix文件系统的权限, 如664, 或者775 
             |-->
            <filePermissions>664</filePermissions>

            <!-- 目录被创建时的权限 -->
            <directoryPermissions>775</directoryPermissions>

            <!-- 传输层额外的配置项 -->
            <configuration></configuration>

        </server>
    </servers>

    <!-- 
   | 从远程仓库才下载 artifacts 时, 用于替代指定远程仓库的镜像服务器配置；
   | 
   | 例如当您无法连接上国外的仓库是, 可以指定连接到国内的镜像服务器；
   |
   | pom.xml 和 setting.xml 中配置的仓库和镜像优先级关系（mirror 优先级高于 repository）: 
   |
   |    repository（setting.xml） < repository（pom.xml） < mirror（setting.xml）
   |
   |    例如, 如果配置了 mirrorOf = *,  则 不管项目的 pom.xml 配置了什么仓库, 最终都会被镜像到 镜像仓库
   |
   |  私服的配置推荐用profile配置而不是mirror
   |-->
    <mirrors>

        <!-- 
         | 【mirro 匹配顺序】: 
         | 多个 mirror 优先级 按照 id字母顺序进行排列（即与编写的顺序无关）
         | 在第一个 mirror 找不到 artifact, 不会继续超找下一个镜像。
         | 只有当 mirror 无法链接的时候, 才会尝试链接下一个镜像, 类似容灾备份。
         |-->

        <!-- 上海交通大学反向代理 --> 
        <mirror>

            <!-- 该镜像的唯一标识符, id用来区分不同的 mirror 元素, 同时会套用使用 server 中 id 相同授权配置链接到镜像 -->
            <id>sjtugmaven</id>

            <!-- 镜像名称, 无特殊作用, 可视为简述 -->
            <name>sjtug maven proxy</name>

            <!-- 镜像地址 -->
            <url>https://mirrors.sjtug.sjtu.edu.cn/maven-central/</url>
            <!-- 被镜像的服务器的id, 必须与 repository 节点设置的 ID 一致。但是 This must not match the mirror id
             | mirrorOf 的配置语法: 
             | *           = 匹配所有远程仓库。 这样所有 pom 中定义的仓库都不生效
             | external:*  = 匹配除 localhost、使用 file:// 协议外的所有远程仓库
             | repo1,repo2 = 匹配仓库 repo1 和 repo2
             | *,!repo1    = 匹配所有远程仓库, repo1 除外
             |-->
            <mirrorOf>central</mirrorOf>
        </mirror>

    </mirrors>

    <!-- 用来配置不同的代理, 多代理 profiles 可以应对笔记本或移动设备的工作环境: 通过简单的设置 profile id 就可以很容易的更换整个代理配置 -->
    <proxies>

        <!-- 代理元素包含配置代理时需要的信息 -->
        <proxy>

            <!-- 代理的唯一定义符, 用来区分不同的代理元素 -->
            <id>example_proxy</id>

            <!-- 该代理是否是激活的那个。true则激活代理。当我们声明了一组代理, 而某个时候只需要激活一个代理的时候, 该元素就可以派上用处 -->
            <active>false</active>

            <!-- 代理的协议 -->
            <protocol>https</protocol>

            <!-- 代理的主机名 -->
            <host>proxy.molo.com</host>

            <!-- 代理的端口 -->
            <port>443</port>

            <!-- 代理服务器认证的登录名 -->
            <username>proxy_user</username>

            <!-- 代理服务器认证登录密码 -->
            <password>proxy_pwd</password>

            <!-- 不该被代理的主机名列表。该列表的分隔符由代理服务器指定；例子中使用了竖线分隔符, 使用逗号分隔也很常见 -->
            <nonProxyHosts>*.google.com|ibiblio.org</nonProxyHosts>

        </proxy>

    </proxies>

    <!--
     | 构建方法的配置清单, maven 将根据不同环境参数来使用这些构建配置。
     | settings.xml 中的 profile 元素是 pom.xml 中 profile 元素的裁剪版本。
     | settings.xml 负责的是整体的构建过程, pom.xml 负责单独的项目对象构建过程。
     | settings.xml 只包含了id, activation, repositories, pluginRepositories 和 properties 元素。
     | 
     | 如果 settings 中的 profile 被激活, 它的值会覆盖任何其它定义在 pom.xml 中或 profile.xml 中的相同 id 的 profile。
     |
     | 查看当前激活的 profile:
     |   mvn help:active-profiles
     |-->
    <profiles>

        <profile>

            <!-- 该配置的唯一标识符 -->
            <id>profile_id</id>

            <!--
             | profile 的激活条件配置；
             | 其他激活方式: 
             | 1. 通过 settings.xml 文件中的 activeProfile 元素进行指定激活。
             | 2. 在命令行, 使用-P标记和逗号分隔的列表来显式的激活, 如: mvn clean package -P myProfile）。 
             |-->
            <activation>

                <!-- 是否默认激活 -->
                <activeByDefault>false</activeByDefault>

                <!--  内建的 java 版本检测, 匹配规则: https://maven.apache.org/enforcer/enforcer-rules/versionRanges.html -->
                <jdk>9.9</jdk>

                <!-- 内建操作系统属性检测, 配置规则: https://maven.apache.org/enforcer/enforcer-rules/requireOS.html -->
                <os>

                    <!-- 操作系统 -->
                    <name>Windows XP</name>

                    <!-- 操作系统家族 -->
                    <family>Windows</family>

                    <!-- 操作系统 -->
                    <arch>x86</arch>

                    <!-- 操作系统版本 -->
                    <version>5.1.2600</version>

                </os>

                <!--
                 | 如果Maven检测到某一个属性（其值可以在POM中通过${名称}引用）, 并且其拥有对应的名称和值, Profile就会被激活。
                 | 如果值字段是空的, 那么存在属性名称字段就会激活profile, 否则按区分大小写方式匹配属性值字段
                 |-->
                <property>

                    <!-- 属性名 -->
                    <name>mavenVersion</name>

                    <!-- 属性值 -->
                    <value>2.0.3</value>

                </property>
                
                <!-- 根据文件存在/不存在激活profile -->
                <file>

                    <!-- 如果指定的文件存在, 则激活profile -->
                    <exists>/path/to/active_on_exists</exists>

                    <!-- 如果指定的文件不存在, 则激活profile -->
                    <missing>/path/to/active_on_missing</missing>

                </file>

            </activation>
            <!-- 扩展属性设置。扩展属性可以在 POM 中的任何地方通过 ${扩展属性名} 进行引用
             |
             | 属性引用方式（包括扩展属性, 共 5 种属性可以引用）: 
             |
             | env.x                  : 引用 shell 环境变量, 例如, "env.PATH"指代了 $path 环境变量（在 Linux / Windows 上是 %PATH% ）.
             | project.x              : 引用 pom.xml（根元素就是 project） 中 xml 元素内容.例如 ${project.artifactId} 可以获取 pom.xml 中设置的 <artifactId /> 元素的内容
             | settings.x             : 引用 setting.xml（根元素就是 setting） 中 xml 元素内容, 例如 ${settings.offline}
             | Java System Properties : 所有可通过 java.lang.System.getProperties() 访问的属性都能在通过 ${property_name} 访问, 例如 ${java.home}
             | x                      : 在 <properties/> 或者 外部文件 中设置的属性, 都可以 ${someVar} 的形式使用
             | 
             |-->
            <properties>

                <!-- 在当前 profile 被激活时,  ${profile.property} 就可以被访问到了 -->
                <profile.property>this.property.is.accessible.when.current.profile.actived</profile.property>

            </properties>

            <!-- 远程仓库列表 -->
            <repositories>
                <!-- 
                 | releases vs snapshots
                 | maven 针对 releases、snapshots 有不同的处理策略, POM 就可以在每个单独的仓库中, 为每种类型的 artifact 采取不同的策略
                 | 例如: 
                 |     开发环境 使用 snapshots 模式实时获取最新的快照版本进行构建
                 |     生成环境 使用 releases 模式获取稳定版本进行构建
                 | 参见repositories/repository/releases元素 
                 |-->

                <!--
                 | 依赖包不更新问题:                
                 | 1. Maven 在下载依赖失败后会生成一个.lastUpdated 为后缀的文件。如果这个文件存在, 那么即使换一个有资源的仓库后, Maven依然不会去下载新资源。
                 |    可以通过 -U 参数进行强制更新、手动删除 .lastUpdated 文件：
                 |      find . -type f -name "*.lastUpdated" -exec echo {}" found and deleted" \; -exec rm -f {} \;
                 |
                 | 2. updatePolicy 设置更新频率不对, 导致没有触发 maven 检查本地 artifact 与远程 artifact 是否一致
                 |-->
                <repository>

                    <!-- 远程仓库唯一标识 -->
                    <id>maven_repository_id</id>

                    <!-- 远程仓库名称 -->
                    <name>maven_repository_name</name>

                    <!-- 远程仓库URL, 按protocol://hostname/path形式 -->
                    <url>http://host/maven</url>

                    <!-- 
                    | 用于定位和排序 artifact 的仓库布局类型-可以是 default（默认）或者 legacy（遗留）
                    | Maven 2为其仓库提供了一个默认的布局；然而, Maven 1.x有一种不同的布局。我们可以使用该元素指定布局是default（默认）还是legacy（遗留）
                    | -->
                    <layout>default</layout>

                    <!-- 如何处理远程仓库里发布版本的下载 -->
                    <releases>

                        <!-- 是否允许该仓库为 artifact 提供 发布版 / 快照版 下载功能 -->
                        <enabled>false</enabled>

                        <!-- 
                         | 每次执行构建命令时, Maven 会比较本地 POM 和远程 POM 的时间戳, 该元素指定比较的频率。
                         | 有效选项是: 
                         |     always（每次构建都检查）, daily（默认, 距上次构建检查时间超过一天）, interval: x（距上次构建检查超过 x 分钟）、 never（从不）
                         |
                         | 重要: 
                         |     设置为 daily, 如果 artifact 一天更新了几次, 在一天之内进行构建, 也不会从仓库中重新获取最新版本
                         |-->
                        <updatePolicy>always</updatePolicy>

                        <!-- 当 Maven 验证 artifact 校验文件失败时该怎么做: ignore（忽略）, fail（失败）, 或者warn（警告） -->
                        <checksumPolicy>warn</checksumPolicy>

                    </releases>

                    <!-- 如何处理远程仓库里快照版本的下载 -->
                    <snapshots>
                        <enabled />
                        <updatePolicy />
                        <checksumPolicy />
                    </snapshots>

                </repository>

                <!-- 
                    国内可用的 maven 仓库地址（updated @ 2019-02-08）：

                    http://maven.aliyun.com/nexus/content/groups/public
                    http://maven.wso2.org/nexus/content/groups/public/
                    http://jcenter.bintray.com/
                    http://maven.springframework.org/release/
                    http://repository.jboss.com/maven2/
                    http://uk.maven.org/maven2/
                    http://repo1.maven.org/maven2/
                    http://maven.springframework.org/milestone
                    http://maven.jeecg.org/nexus/content/repositories/
                    http://repo.maven.apache.org/maven2
                    http://repo.spring.io/release/
                    http://repo.spring.io/snapshot/
                    http://mavensync.zkoss.org/maven2/
                    https://repository.apache.org/content/groups/public/
                    https://repository.jboss.org/nexus/content/repositories/releases/   
                -->

            </repositories>

            <!-- 
             | maven 插件的远程仓库配置。maven 插件实际上是一种特殊类型的 artifact。
             | 插件仓库独立于 artifact 仓库。pluginRepositories 元素的结构和 repositories 元素的结构类似。
             |-->
            <!--
            <pluginRepositories>
                <pluginRepository>
                    <releases>
                        <enabled />
                        <updatePolicy />
                        <checksumPolicy />
                    </releases>
                    <snapshots>
                        <enabled />
                        <updatePolicy />
                        <checksumPolicy />
                    </snapshots>
                    <id />
                    <name />
                    <url />
                    <layout />
                </pluginRepository>
            </pluginRepositories>
            -->

        </profile>

    </profiles>

    <!--
     | 手动激活 profiles 的列表, 按照 profile 被应用的顺序定义 activeProfile
     | 任何 activeProfile, 不论环境设置如何, 其对应的 profile 都会被激活, maven 会忽略无效（找不到）的 profile
     |-->
    <!--
    <activeProfiles>
        <activeProfile>not-exits-profile</activeProfile>
    </activeProfiles>
    -->

</settings>
```

### 2.最简配置

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
            http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <localRepository/>
    <interactiveMode/>
    <usePluginRegistry/>
    <offline/>
    <pluginGroups/>
    <servers/>
    <mirrors/>
    <proxies/>
    <profiles/>
    <activeProfiles/></settings>
</settings>
```



## 5. 常用插件

官方推荐：https://maven.apache.org/plugins/index.html

### 1.maven-compiler-plugin



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



### 2.maven-war-plugin

### 3. maven-surefire-plugin

http://maven.apache.org/surefire/maven-surefire-plugin/

> ​	maven里执行测试用例的插件，不显示配置就会用默认配置。这个插件的`surefire:test`命令会默认绑定maven执行的`test`阶段。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.19</version>
    <dependencies>
        <dependency>
            <groupId>org.apache.maven.surefire</groupId>
            <artifactId>surefire-junit47</artifactId>
            <version>2.19</version>
        </dependency>
    </dependencies>
    <configuration>
        <skipTests>true</skipTests>
    </configuration>
</plugin>
```

在properties配置中声明跳过测试用例

```xml
<properties>
    <maven.test.skip>true</maven.test.skip>
</properties>
```

或

```xml
<properties>
    <skipTests>true</skipTests>
</properties>
```

### 4. maven-resources-plugin

### 5. maven-enforcer-plugin

### 6.jacoco-maven-plugin

### 7.maven-source-plugin

### 8.git-commit-id-plugin

### 9.build-helper-maven-plugin





