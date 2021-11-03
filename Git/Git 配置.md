# config
Git 共有三个级别的 config 文件：
1. system
2. global。配置文件: $home\\.gitconfig
3. local。配置文件: 仓库目录下的 .git\\config。

这三个级别都分别配置了用户信息优先级从高到低：
local >> global >> system 
当 git commit 时，会依次从local、global、system 里读取用户信息。



# 记住密码
https方式每次都要输入密码，按照如下设置即可输入一次就不用再手输入密码

## 远程地址带有密码
增加远程地址的时候带上密码

如在 config 文件中
```shell
[remote "origin"]
	url = https://github.com/justin4v/ZettelNotes.git
	fetch = +refs/heads/*:refs/remotes/origin/*
```
修改 url 增加用户名和密码
http://yourname:password@git.oschina.net/name/project.git

```shell
[remote "origin"]
	url = https://justin4v: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@github.com/justin4v/ZettelNotes.git
	fetch = +refs/heads/*:refs/remotes/origin/*
```


# 报错
```
remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.
```

## 原因

> 原因：自2021年8月13日起，github不再支持使用密码push的方式
> 解决方案：
> 一、 使用SSH  
> 二、使用Personal access token


Personal Access Token 参考
[github docs](https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token) 

## Personal Access Token
2022年2月 失效
```
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
```