---
title: Maven deploy 部署 jar 到 Nexus 私服
date: 2020-04-19
tags: [Maven,Nexus]
categories: Java
---

在SOA服务成为标准配置的今天，我们经常会遇到需要将jar上传到公司Nexus私服来满足其他服务调用的需求。

常用命令如下:
```shell
mvn deploy:deploy-file -DgroupId=<group-id> \
  -DartifactId=<artifact-id> \
  -Dversion=<version> \
  -Dpackaging=<一般是jar> \
  -Dfile=<相对路径和绝对路径都可> \
  -Durl=<公司仓库地址> \
  -DrepositoryId=<一般是snapshots或者releases，根据.m2/settings.xml文件servers配置来> \
  -DpomFile=<pom.xml> \
  -Dsources=<源码file地址，可不填>
```

上面这个命令会生成jar并且上传到Nexus 私服中。

<!-- more -->


## -DpomFile=pom.xml

这个参数指定pom文件为我们自己的pom文件，方便当其他人引入我们的jar的时候把我们的jar包的依赖都一起引入。但是如果我们不指定的话，默认会生成一个空pom.xml，没有依赖关系，这个时候如果别人引用了我们的 jar 包，就会抛出 NoClassDefFoundError 错误，因为编译时没有问题，但运行时却找不到 class 文件。

## maven deploy plugin pom 配置

下面提供了maven deploy plugin的xml 配置，方面在snapshots阶段快速部署。

```xml
<plugin>
    <artifactId>maven-deploy-plugin</artifactId>
    <version>2.8.2</version>
    <executions>
        <execution>
            <id>deploy-file</id>
            <phase>deploy</phase>
            <goals>
                <goal>deploy-file</goal>
            </goals>
            <configuration>
                <groupId>com.example</groupId>
                <artifactId>demo</artifactId>
                <version>1.0.0-SNAPSHOT</version>
                <packaging>jar</packaging>
                <file>target/demo-api-1.0.0-SNAPSHOT.jar</file>
                <url>http://nexus.公司.com/nexus/content/repositories/snapshots/</url>
                <repositoryId>snapshots</repositoryId>
                <pomFile>pom.xml</pomFile>
            </configuration>
        </execution>
    </executions>
</plugin>
```

## maven deploy source jar(maven上传源代码到Nexus私服)

因为maven部署插件的`sources`配置是文件，因此我们需要在部署jar之前将 `source.jar`打包出来，所以将jar-no-fork绑定在verify流程节点上。

```xml
<build>
    <plugins>
        <plugin>
            <artifactId>maven-source-plugin</artifactId>
            <executions>
                <execution>
                    <id>attach-sources</id>
                    <phase>verify</phase>
                    <goals>
                        <goal>jar-no-fork</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <artifactId>maven-deploy-plugin</artifactId>
            <version>2.8.2</version>
            <executions>
                <execution>
                    <id>deploy-file</id>
                    <phase>deploy</phase>
                    <goals>
                        <goal>deploy-file</goal>
                    </goals>
                    <configuration>
                        <groupId>com.example</groupId>
                        <artifactId>demo</artifactId>
                        <version>1.0.0-SNAPSHOT</version>
                        <packaging>jar</packaging>
                        <file>target/demo-1.0.0-SNAPSHOT.jar</file>
                        <url>http://nexus.公司.com/nexus/content/repositories/snapshots/</url>
                        <repositoryId>snapshots</repositoryId>
                        <pomFile>pom.xml</pomFile>
                        <sources>target/demo-1.0.0-SNAPSHOT-sources.jar</sources>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

**参考文档**:
1.  [deploy-file 插件 官方文档](https://maven.apache.org/plugins/maven-deploy-plugin/deploy-file-mojo.html#packaging)
