## Source
**KEYS**
- internal command；
- 将目标文件当做 shell script 并依次执行其中的命令；
- 常用于Source修改后的配置文件，使其立即生效。


**DES**

source命令的关键在于将文件当做脚本执行，source 配置文件，相当于在当前 context 中导入了一系列配置变量。
例如有配置文件 nacos.config
``` properties
server_ip=10.5.69.9
port=30122
```
执行 `source nacos.config`之后，当前 context 中就注入了 `server_ip`和`port`变量。
执行 `echo $`