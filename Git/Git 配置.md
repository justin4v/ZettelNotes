# config
Git 共有三个级别的 config 文件：
1. system
2. global。配置文件: $home\\.gitconfig
3. local。配置文件: 仓库目录下的 .git\\config。

这三个级别都分别配置了用户信息优先级从高到低：
local >> global >> system 
当 git commit 时，会依次从local、global、system 里读取用户信息。



# 
