# linux命令记录



## 1. 设置自启动脚本

``` bash
$runlevel #获取当前运行级别

$cd /etc/init.d
$touch runlwc.sh 
$update-rc.d run.sh start 98 5 .
```

run.sh内容

``` bash 
#!/bin/sh
cconnect -d -w /home/root/.cconnect/
```



## 2. 软链接

a. 创建 `ln -s [目标目录]  [软链接地址]`
b. 删除 `rm -rf [软链接地址]`
c. 修改 `ln -snf [新目标目录]  [软链接地址]`

## 3. tar

a. 解压缩 `tar -xzvf xxxx.tar.gz`
b. 压缩 `tar -czvf xxxx.tar.gz xxxx/`