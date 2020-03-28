---
title: Ubuntu udp replay pcap package
date: 2020-03-28 20:10:46
tags: Ubuntu,udp,pcap
---
在日常工作中，需要对抓包后的数据在调试时重放。由于UDP头限定等问题，会造成传统UDP包发送无法被程序接收。此处列出一种可以重放保存的UDP pcap包的方法。
1. 安装`bittwist`, [此工具](http://bittwist.sourceforge.net/doc.html)能够修改pcap包，其他修改pcap包的工具（如NetDude）也能用，但是此处我们使用`bittwist`处理
2. 获取需要接收UDP的机器的IP地址，此处我们用本机地址。输入`ip address`命令, 我们需要IP地址和mac地址，此处为`192.168.1.106`和`9c:b6:d0:bf:1f:8b`

    ```
    2: wlp59s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
      link/ether 9c:b6:d0:bf:1f:8b brd ff:ff:ff:ff:ff:ff
      inet 192.168.1.106/24 brd 192.168.1.255 scope global dynamic noprefixroute wlp59s0
        valid_lft 4881sec preferred_lft 4881sec
      inet6 fe80::bd4a:cb93:1b2e:2ca0/64 scope link noprefixroute
        valid_lft forever preferred_lft forever
    ```

3. 使用`bittwiste`修改pcap包， 如果不修改，导致验证不一致，UDP包会被丢弃

    ```
    # 修改为接收放端口
    bittwiste -I 0316.pcap -O 0316-port.pcap -T udp -d 3002
    # 修改发送方和接收方IP
    bittwiste -I 0316-port.pcap -O 0316-port-ip.pcap -T ip -s 192.168.1.106 -d 192.168.1.106
    # 修改发送方和接收方MAC地址
    bittwiste -I 0316-port-ip.pcap -O 0316-port-ip-mac.pcap -T eth -s 9c:b6:d0:bf:1f:8b -d 9c:b6:d0:bf:1f:8b
    ```

4. 发送UDP包，此处我们使用[github](https://github.com/rigtorp/udpreplay)上的一个开源工具`udpreplay`
    ```
    udpreplay -i wlp59s0 0316-new-ip2-mac.pcap
    ```

至此UDP包能够在接收方Linux程序收到消息了。