#Jenkins #GitParameter #Bug

# 简介
使用 Git Parameter 传递参数到 Branch to build 时：
1. 去除 “轻量级检出”（lightweight checkout）；
2. 否则可能会报错 “”
# 参考
1. [Jenkins Pipeline 动态参数传递 Git 分支 ](https://cloud.tencent.com/developer/article/1816877)
2. [[#JENKINS-28447] CpsScmFlowDefinition does not resolve variables - Jenkins Jira](https://issues.jenkins.io/plugins/servlet/mobile#issue/JENKINS-28447)