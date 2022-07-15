#Https #Springboot #Keytool 

# 获取证书
## jdk keytool 生成自定义证书
```shell
keytool -genkey -alias httpsTest -dname "CN=localhost,OU=UPlus,O=UIH,L=WH,ST=HB,C=CN" -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore keystore.p12 -validity 365
```
- 在当前目录生成证书文件 `keystore.p12`；
- 这里的 `CN (Common Name)` 为 `localhost` ，相当于给 

### 命令含义
- `alias` 别名  
- `keypass` 指定生成密钥的密码  
- `keyalg` 指定密钥使用的加密算法（如 RSA）  
- `keysize` 密钥大小  
- `validity` 过期时间，单位天  
- `keystore` 指定存储密钥的密钥库的生成路径、名称  
- `storepass` 指定访问密钥库的密码

keytool 中 `dname` 字段含义：
1. `C` ：Country，单位所在国家，为两位数的国家缩写，如：CN 表示中国；
2. `ST` ：State/Province，单位所在州或省；
3. `L` ：Locality，单位所在城市/或县区；
4. `O` ：Organization，此网站的单位名称；
5. `OU` ：Organization Unit，下属部门名称，也常常用于显示其他证书相关信息，如证书类型，证书产品名称或身份验证类型或验证内容等；
6. `CN` ：Common Name，网站的域名；



# Springboot 配置


# 参考
