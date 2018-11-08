# Maven 中如何管理多模块项目的依赖关系

> 平时在查看项目的过程中，发现包的依赖关系及其随便，在各个子模块中都各自引入相应的依赖包，有些时候重复导入了也不会发觉。
> 在参考alibaba dubbo的源码之后，做出如下总结（别人家的代码 ==！）

```
groupId：组织标识，一般为：公司网址的反写+项目名
artifactId：项目名称，一般为：项目名-模块名
```

## 1.首先建一个空的maven项目，项目只需要一个`pom.xml`文件

> 交代一下我的demo

项目（root）的`groupId`是`com.nezha.learn.demo`

项目名称 `artifactId` 是 `demo`

该项目有三个子模块：

```xml
<!--使用Jedis依赖-->
<module>demo-redis</module>
<!--主要用于管理依赖关系-->
<module>demo-parent</module>
<!--使用spring boot的依赖-->
<module>demo-juc</module>
```

## 2.通过`IDEA`中`New` > `Module`来构建多个模块

其中的`pom.xml`文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.nezha.learn.demo</groupId>
    <artifactId>demo</artifactId>
    <packaging>pom</packaging>
    <version>1.1-SNAPSHOT</version>
    <!--这里是我们的子模块列表-->
    <modules>
        <!--使用Jedis依赖-->
        <module>demo-redis</module>
        <!--主要用于管理依赖关系-->
        <module>demo-parent</module>
        <!--使用spring boot的依赖-->
        <module>demo-juc</module>
    </modules>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-release-plugin</artifactId>
                <version>2.4.1</version>
                <configuration>
                    <autoVersionSubmodules>true</autoVersionSubmodules>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## 3.创建一个专门管理依赖包的模块

> 同样，该模块只有一个`pom.xml`文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>demo</artifactId>
        <groupId>com.nezha.learn.demo</groupId>
        <version>1.1-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <packaging>pom</packaging>
    <artifactId>demo-parent</artifactId>

    <!--集中配置版本关系-->
    <properties>
        <jedis.version>2.9.0</jedis.version>
        <spring-boot.version>1.5.2.RELEASE</spring-boot.version>
    </properties>

    <!--此处 dependencyManagement 并不会直接引入依赖，是一种懒加载的方式-->
    <dependencyManagement>
        <dependencies>
            <!--引入Redis的客户端-->
            <dependency>
                <groupId>redis.clients</groupId>
                <artifactId>jedis</artifactId>
                <version>${jedis.version}</version>
            </dependency>
            <!--引入spring boot的依赖-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

注意此处的`artifactId`是项目名称，因为所有子模块的依赖都在项目`root`的`pom.xml`中定义的。


## 4.在子模块中引入`demo-paren`的依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>demo-parent</artifactId>
        <groupId>com.nezha.learn.demo</groupId>
        <version>1.1-SNAPSHOT</version>
        <relativePath>../demo-parent/pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>demo-juc</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>${project.groupId}</groupId>
            <artifactId>demo-redis</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
```

> 注意此处的`artifactId`是`demo-parent`
> 此时引入的spring依赖是不需要加上版本号的
> 然后`${project.groupId}`和`${project.version`用于指代项目和版本。

## 如何更新所有子模块的版本号

> 这个也好解决，Maven有插件鸭！

在父级的`pom.xml`中加个插件就OK了，`plugins` > `release` > `update-versions` 

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-release-plugin</artifactId>
                <version>2.4.1</version>
                <configuration>
                    <autoVersionSubmodules>true</autoVersionSubmodules>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

## maven的pom文件报错： must be "pom" but is "jar"

> parent工程的pom.xml文件的project节点下加入如下节点：

```xml
<packaging>pom</packaging>
```

## 参考文献

1. @Maven POM 详解：<https://www.jianshu.com/p/8417a94c4d94>
2. 使用maven-release-plugin自动化版本发布<https://blog.csdn.net/shenchaohao12321/article/details/79302791>
3. spring dubbo的源码：<https://github.com/apache/incubator-dubbo-spring-boot-project>