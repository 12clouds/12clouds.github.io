---
title: WSL2 网络代理设置
tags: wsl, linux
---

# WSL2 网络代理设置

有两个关键步骤： 1. WSL2 中配置的代理要指向 Windows 的 IP； 2. Windows 上的代理客户端需要允许来自本地局域网的请求；

7890 是 Windows 上运行的代理客户端的端口，记得要在 Windows 代理客户端上配置允许本地局域网请求。

一键配置脚本

``` bash
# @jayce： 直接将这段追加到 ~/.bashrc， 这样每次打开终端就自动设定了
# 获取wsl虚拟机的ip， 并代理至windows 7890 端口，让wsl可以访问外网
echo "设定wsl网络代理到7890外网访问端口......"
 
host_ip=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
export ALL_PROXY="http://$host_ip:7890"
# curl 命令检查，并仅输出状态码
echo "尝试通过curl命令检查 google 是否可以访问......返回状态码为："
curl -s -o /dev/null -w "%{http_code}\n" https://www.google.com
```
