# idea启动java服务报错OutOfMemoryError: GC overhead limit exceeded解决方法

在用idea开发java项目时，启动报内存溢出错误，致服务启动失败：

> Error:java: java.lang.OutOfMemoryError: GC overhead limit exceeded

报此错说明启动期间内存不够用了，把idea的启动进程堆内存值设大点就行了。

设置窗口：Settings --->Build,Execution,Deployment---> Complier

把 build process heap size 值改大点即可

![img](https://cdn.nlark.com/yuque/0/2023/png/22461487/1692602584333-5be6d7be-2191-4a47-82c6-0c396dd817ab.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

把此值改为1000，重新启动服务正常。