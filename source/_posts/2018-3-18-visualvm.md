---
layout: post
title: VisualVm monitor remote jvm
date: 2018-03-8 00:00
tags: [java, jvm, JVisualVM]
---

### 使用 VisualVM 监控 JVM 的 内存使用率 GC情况，开始时使用的如下代码：

```
java command
	-Djava.rmi.server.hostname=192.168.2.234 
	-Dcom.sun.management.jmxremote 
	-Dcom.sun.management.jmxremote.port=1099 
	-Dcom.sun.management.jmxremote.authenticate=false 
	-Dcom.sun.management.jmxremote.ssl=false 
 ```

### 外网访问可能存在问题

在局域网内是完全没有问题，局域网不会牵涉端口的问题防火墙的问题，但是一旦要监控远程云主机的应用的时候问题就来了，写了之后怎么都连不上，经过查看，JMXServer对外暴露的接口是1099没问题，有问题的是VisualVm连接到JMX是通过RMI Server连接的，假如不配置 RMI的端口，会自动生成端口，导致这个端口不可控，在云主机中肯定要有防火墙的，那怎么配置呢，如下配置：

```
java command
			-Djava.rmi.server.hostname=192.168.2.234 
			-Dcom.sun.management.jmxremote 
			-Dcom.sun.management.jmxremote.port=1099 
			// if not set, RMI will choose random port 
			-Dcom.sun.management.jmxremote.rmi.port=1099
			-Dcom.sun.management.jmxremote.authenticate=false 
			-Dcom.sun.management.jmxremote.ssl=false 
			-Dcom.sun.management.jmxremote.local.only=false 
 RMI Connector 和 JMX Server 的端口一直，VisualVM就能监控进程了
```
 

参考：
  https://stackoverflow.com/questions/20884353/why-java-opens-3-ports-when-jmx-is-configured
  https://docs.oracle.com/javase/6/docs/api/javax/management/remote/rmi/package-summary.html
  http://blog.51cto.com/lizhenliang/1608005