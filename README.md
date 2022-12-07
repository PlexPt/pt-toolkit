# 开发常用一揽子工具包

  <p align="center" >      <a href="https://mvnrepository.com/artifact/com.github.plexpt/toolkit">
            <img src="https://maven-badges.herokuapp.com/maven-central/com.github.plexpt/toolkit/badge.svg" >
        </a>
 </p>


## 使用
```
<!-- https://mvnrepository.com/artifact/com.github.plexpt/toolkit -->
<dependency>
    <groupId>com.github.plexpt</groupId>
    <artifactId>toolkit</artifactId>
    <version>2022.6.0</version>
</dependency>
```

## 发布步骤

1. 发布release
2. 等待action自动构建完成
3. 去 https://oss.sonatype.org/#stagingRepositories 点击release



## 自动发布参考
自动1
https://segmentfault.com/a/1190000039716048

自动2
https://monkeywie.cn/2020/11/06/maven-deploy-to-nexus-by-github-action/

手动
https://blog.fundebug.com/2019/01/14/how-to-deploy-jar-to-maven-central-repository/
https://www.jianshu.com/p/8c3d7fb09bce
https://zhuanlan.zhihu.com/p/141676033

发布
https://oss.sonatype.org/#stagingRepositories

### 命令

gpg --gen-key

gpg --list-keys

gpg --keyserver https://keys.openpgp.org/ --send-keys D16494BFCDC100DF5F051D4335BEE09515BF6AB1
上传完应有邮件提示，或者换服务器
 - http://pgp.mit.edu:11371/
 - https://keys.openpgp.org/
 - http://keyserver.ubuntu.com:11371/


gpg -a --export-secret-keys D16494BFCDC100DF5F051D4335BEE09515BF6AB1
