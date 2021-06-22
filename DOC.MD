# [使用CI/CD工具Github Action发布jar到Maven中央仓库](https://segmentfault.com/a/1190000039716048)

[![](assets/1624344755-6dcf08306d3bfb2275d64dbbf7b86881.webp)**码农小胖哥**](https://segmentfault.com/u/10000000)发布于 3 月 26 日

之前发布开源项目[Payment Spring Boot](https://github.com/NotFound403/payment-spring-boot)到Maven中央仓库我都是手动执行`mvn deploy`，在**CI/CD**大行其道的今天使用这种方式有点“原始”。于是我一直在寻求一种能够支持流水线作业的发布工具，能让我在进行合并代码时自动触发构建发布。有一款免费的产品能做到这一点，它就是**Github Action**。

## Github Action

**Github Action**是由Github创建的**CI/CD**服务。 它的目的是使所有软件开发工作流程的自动化变得容易。 直接从GitHub构建，测试和部署代码。CI（持续集成）由很多操作组成，比如代码合并、运行测试、登录远程服务器，发布到第三方服务等等。

今天我就尝试用**Github Action**来将[Payment Spring Boot](https://github.com/NotFound403/payment-spring-boot)发布到Maven中央仓库。

## 期望效果

当代码库发布**Release(发行版)**的时候触发一个将**Release**所包含的分支发布到Maven中央仓库的效果。

![Github中的Release发行版](assets/1624344755-3cb371466178e2823a301a922bf91cce.webp "Github中的Release发行版")

> 拓展阅读：
> 
> `Release（发行版）`是具有 `Changelogs`（变更日志）和二进制文件的一级对象，可以代表超出 Git 架构本身的一个特定时间点之前的所有项目历史。

## 前提条件

关于项目如何发布到Maven中央仓库及其一些必要的条件这里不再讨论，网上有很多教程，有兴趣的可以去搜索一下。也可以参考[Payment Spring Boot](https://github.com/NotFound403/payment-spring-boot)的`pom.xml`。这里只说一些关键的点，您需要：

*   OSSRH账号。
*   GPG密钥信息。

> 💡注意：这两个都是敏感数据不要泄露给其他人，否则你的项目将可能被其他人掌控。

## Github Action Secrets

为了从**Github Action**发布，我们需要让**Github Action**可以使用我们的GPG私钥和OSSRH用户信息。为了保证这些敏感信息的安全性，我们可以使用[Github Action Secrets](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets)来存储它们。

### GPG的细节补充

这里的 `GPG_PASSWORD`为**GPG**的 `Passphrase`,网上Maven中央仓库教程肯定会提这个，这里不再细说。需要注意的是公钥一定要上传公钥服务器。

`GPG_SECRET` 获取步骤如下：

*   确定你有GPG环境，并按照其它教程配置好了GPG密钥对。
*   执行 `gpg --list-secret-keys` 查看Key列表并复制**你需要用的ID**

```shell
[root@192 ~]# gpg --list-secret-keys
/root/.gnupg/pubring.kbx
------------------------
sec   rsa2048 2020-07-27 [SC]
      8AC0AB86C34ADC6ED110A5A9E6730F4374866065
uid           felord (felord) <dax@felord.cn>
```

*   执行`gpg -a --export-secret-keys KEY_ID`(`KEY_ID`为上图中以`8AC0AB`开头的字符串)导出私钥，这里需要输入保护私钥的密码（`GPG_PASSWORD`）。然后会出现以下的密文：

```plain
-----BEGIN PGP PRIVATE KEY BLOCK----
............密文区域.............
-----END PGP PRIVATE KEY BLOCK-----
```

这就是`` `GPG_SECRET`` \`。

## 修改项目的POM

然后修改项目的`pom.xml`文件，模板我已经提出来了，不能修改的地方我已经写了注释：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <groupId>cn.felord</groupId>
    <artifactId>payment-spring-boot</artifactId>
    <version>1.0.9.RELEASE</version>
    <packaging>pom</packaging>
    <modelVersion>4.0.0</modelVersion>

    <name>payment-spring-boot</name>
    <description>wechat-pay and alipay sdk</description>
    <url>https://github.com/NotFound403/payment-spring-boot</url>

    <licenses>
        <license>
            <name>Apache License, Version 2.0</name>
            <url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
            <distribution>repo</distribution>
            <comments>A business-friendly OSS license</comments>
        </license>
    </licenses>

    <developers>
        <developer>
            <name>felord</name>
            <email>felord@qq.com</email>
            <organization>felord.cn</organization>
        </developer>
    </developers>

    <scm>
        <tag>payment-spring-boot-1.0.9.RELEASE</tag>
        <url>https://github.com/NotFound403/payment-spring-boot</url>
        <connection>scm:git:https://github.com/NotFound403/payment-spring-boot.git</connection>
        <developerConnection>scm:git:https://github.com/NotFound403/payment-spring-boot.git</developerConnection>
    </scm>

    <profiles>
        <!-- Deployment profile (required so these plugins are only used when deploying) -->
         <!-- 下面这个标签里的不能改 -->
        <profile>
            <id>deploy</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-source-plugin</artifactId>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-javadoc-plugin</artifactId>
                    </plugin>
                    <!-- GPG plugin -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-gpg-plugin</artifactId>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>

    <modules>
        <module>payment-spring-boot-autoconfigure</module>
        <module>payment-spring-boot-starter</module>
    </modules>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-boot.version>2.4.2</spring-boot.version>
        <aliy-pay-sdk.version>4.10.167.ALL</aliy-pay-sdk.version>
        <oss-starter.version>1.0.0.RELEASE</oss-starter.version>
        <lombok.verison>1.18.12</lombok.verison>
        <jackson.version>2.9.10</jackson.version>
        <bcprov.version>1.66</bcprov.version>
        <jackson.version>2.11.4</jackson.version>
        <httpclient.version>4.5.13</httpclient.version>
    </properties>

    <!-- 下面这个标签里的不能改 -->
    <distributionManagement>
        <repository>
            <id>ossrh</id>
            <name>Nexus Release Repository</name>
            <url>https://oss.sonatype.org/service/local/staging/deploy/maven2</url>
        </repository>
        <snapshotRepository>
            <id>sonatype-nexus-snapshots</id>
            <name>Nexus Snapshot Repository</name>
            <url>https://oss.sonatype.org/content/repositories/snapshots</url>
        </snapshotRepository>
    </distributionManagement>

    <dependencyManagement>
        <dependencies>
            <!-- 你项目的依赖写这里-->
        </dependencies>
    </dependencyManagement>
         <!-- 下面这个标签里的不能改 -->
    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-source-plugin</artifactId>
                    <version>3.1.0</version>
                    <executions>
                        <execution>
                            <phase>package</phase>
                            <goals>
                                <goal>jar-no-fork</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-javadoc-plugin</artifactId>
                    <version>3.2.0</version>
                    <configuration>
                        <show>private</show>
                        <nohelp>true</nohelp>
                        <charset>UTF-8</charset>
                        <encoding>UTF-8</encoding>
                        <docencoding>UTF-8</docencoding>
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
                            <configuration>
                                <!-- Prevent `gpg` from using pinentry programs -->
                                <gpgArguments>
                                    <arg>--pinentry-mode</arg>
                                    <arg>loopback</arg>
                                </gpgArguments>
                            </configuration>
                        </execution>
                    </executions>
                </plugin>
                <plugin>
                    <groupId>org.sonatype.plugins</groupId>
                    <artifactId>nexus-staging-maven-plugin</artifactId>
                    <version>1.6.8</version>
                    <extensions>true</extensions>
                    <configuration>
                        <serverId>ossrh</serverId>
                        <nexusUrl>https://oss.sonatype.org/</nexusUrl>
                        <autoReleaseAfterClose>false</autoReleaseAfterClose>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.sonatype.plugins</groupId>
                <artifactId>nexus-staging-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

> 结合你自己的项目进行必要的填充。

## 编写Github Action 脚本

`Github Action`脚本保存在项目根目录下的`.github/workflows`路径中。我们只需要编写一个`yaml`来声明执行的步骤即可，具体的语法可以去看相关的中文文档，这里只列出发布到**Maven**中央仓库的action脚本：

```yaml
# 相当于脚本用途的一个声明
name: Maven Central Repo Deployment
# 触发脚本的事件  这里为发布release之后触发
on:
  release:
    types: [released]
# 定义一个发行任务
jobs:
  publish:
# 任务运行的环境
    runs-on: ubuntu-latest
# 任务的步骤
    steps:
# 1. 声明 checkout 仓库代码到工作区
      - name: Checkout Git Repo
        uses: actions/checkout@v2
# 2. 安装Java 环境 这里会用到的参数就是 Git Action secrets中配置的，
#    取值要在key前面加  secrets.
      - name: Set up Maven Central Repo
        uses: actions/setup-java@v1
        with:
          java-version: 1.8
          server-id: sonatype-nexus-staging
          server-username: ${{ secrets.OSSRH_USER }}
          server-password: ${{ secrets.OSSRH_PASSWORD }}
          gpg-passphrase:  ${{ secrets.GPG_PASSWORD }}
# 3. 发布到Maven中央仓库
      - name: Publish to Maven Central Repo
# 这里用到了其他人写的action脚本，详细可以去看他的文档。
        uses: samuelmeuli/action-maven-publish@v1
        with:
          gpg_private_key: ${{ secrets.GPG_SECRET }}
          gpg_passphrase: ${{ secrets.GPG_PASSWORD }}
          nexus_username: ${{ secrets.OSSRH_USER }}
          nexus_password: ${{ secrets.OSSRH_PASSWORD }}
```

## 触发Action

都准备完毕后，action脚本要提交到**Github**，当你使用`release`功能后会自动在`action`一栏中执行整个发布流程：

![自动发布到Maven中央仓库](assets/1624344755-e66879dd8fcb3b9757f684e3209a7493.webp "自动发布到Maven中央仓库")

这种方式一次配置，到处发布。我们不需要再关心怎么发布了，只需要关心在什么时候发布。

> 可以参考 **Payment Spring Boot**项目。

## 总结

今天通过对**Github Action**的简单使用来介绍了**CI/CD**的作用，这个技术体系是项目集成交付的趋势，也是面试中的一个亮点技能。 而且这种方式可以实现“一次配置，随时随地集成部署”。

`关注公众号：Felordcn 获取更多资讯`

[个人博客：https://felord.cn](https://felord.cn/)

[java](https://segmentfault.com/t/java)

阅读 286发布于 3 月 26 日

赞收藏

[分享](###)

本作品系原创，[采用《署名-非商业性使用-禁止演绎 4.0 国际》许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/)

