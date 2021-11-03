# config
Git 共有三个级别的 config 文件：
system、global和local。global的在$home\.gitconfig，local的在仓库目录下的.git\config。这三个级别都分别配置了用户信息，当git commit时，会依次从local、global、system里读取用户信息。
