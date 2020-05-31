# Sping Cloud Alibaba

## Nacos

异步非阻塞通信

Netty -> NIO、AIO

HTTP -> 应用层，跨防火墙，在不同的局域网之间通信

RPC -> 远程过程调用，TCP，第四层，传输层。优点：速度快。缺点：不能跨防火墙，仅支持局域网通信

对内RPC，对外RESTful风格

Nacos下载地址

https://github.com/alibaba/nacos

mvn -Prelease-nacos clean install -U       --安装release-nacos profile版本，到本地库中

启动

D:\Environment\nacos\distribution\bin\startup.cmd -m standalone   ---单机版运行

启动后的访问地址：http://localhost:8848/nacos/

http://localhost:8081/actuator/nacos-discovery



