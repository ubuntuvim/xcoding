---
title: OOM问题排查
excerpt: 通过本文您将学习到如何通过VisualVM分析定位OOM问题。
tag:
	- OOM
	- Java
	- 内存溢出
---

https://www.bilibili.com/video/BV1yQ4y1y7CE/?spm_id_from=pageDriver&vd_source=9877623a0618b2480d87deedce70ed31



通过*VisualVM*打开dump文件，主要看如下几个数据项：



### 实例数量

![image-20230214022632396](https://oscimg.oschina.net/oscnet/up-48876a12c72262818c97e5daaa8b569888f.png)



### 实例内存占用大小

![image-20230214022719485](https://oscimg.oschina.net/oscnet/up-308922e2d23ec1a5245ff4fe0d3c97f332a.png)



![image-20230214022756553](https://oscimg.oschina.net/oscnet/up-57c1f10042466556cb538034cee1b134914.png)

在可疑问题类上面右键打开，选中之后可以查看GC Root，可以看到详细的调用链，定位到问题调用的上层位置。



![image-20230214022842191](https://oscimg.oschina.net/oscnet/up-e24dcc90a9e61e3c16e17e1a032d95cd538.png)

![image-20230214022944577](https://oscimg.oschina.net/oscnet/up-3b698c6ff28a7957d277aeb5adc5518ba1b.png)





![image-20230214023019606](https://oscimg.oschina.net/oscnet/up-fc93fdfb4b0925127bedce1ed66e0e2d071.png)

在问题类上面右键选择select in threads查看到详细的堆栈信息。

![image-20230214023106072](https://oscimg.oschina.net/oscnet/up-9fe48aa6b3147563f9dcb54fbdc5fa5b697.png)



通过堆栈详细信息可以定位到某个类的某一行代码，再分析代码是否存在资源未释放的问题、加载大对象、加载大文件的问题。



