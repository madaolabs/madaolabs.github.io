---
title: ipv6
date: 2023-08-09 08:17:12
tags:
---

ipv6的在国内普及度越来越高。家庭的宽带也可以使用ipv6，并且也可以从外网访问，还有重要的一点是免费。

家庭接入宽带的路径是：广域网 -> 光猫(调制解调器) -> 路由器 -> 电脑等设备。

下面是我的开通流程。我家里是接的是电信宽带。

1. 打电话给电信客服(或者工程师)，要求开通ipv6
2. 开通路由器的ipv6功能。
3. 此时，家庭的设备就具备的ipv6地址。
4. 如果你想要从外网访问家庭设备。需要关闭光猫的防火墙(需要超级管理员，找电信工程师要就可以), 在光猫显示已经关闭的情况下还是不行。使用 http://192.168.1.1:8080/enableTelnet.html 开启 telnet(事后记得关闭telnet)。

telnet登录命令：
   ```shell
    telnet: telnet 192.168.1.1
    User:
    Password:
   ``` 

    执行命令:
   ```shell
    ip6tables -F
    ip6tables -P INPUT ACCEPT
    ip6tables -P FORWARD ACCEPT
    ip6tables -P OUTPUT ACCEPT
   ```

启动一个web服务，端口7001，做访问测试，成功。