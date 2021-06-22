# 开发常用一揽子工具包





## 参考
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

gpg --keyserver hkp://keyserver.ubuntu.com:11371 --send-keys D16494BFCDC100DF5F051D4335BEE09515BF6AB1

gpg -a --export-secret-keys D16494BFCDC100DF5F051D4335BEE09515BF6AB1
