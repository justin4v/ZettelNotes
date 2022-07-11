#Docker #Command-param 

# 镜像列表

```bash
docker images
```

# 删除镜像
强制删除 workflow 的镜像

```shell
docker rmi -f $(docker images | grep workflow |awk '{print $3}')
```
- rmi 删除镜像  
- -f 强制删除  
- $( ) 执行命令  
- | 管道  
- awk 文本处理