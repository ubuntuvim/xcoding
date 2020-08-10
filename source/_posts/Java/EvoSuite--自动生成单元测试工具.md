---
title: EvoSuite--自动生成单元测试工具
tag:
	- Java
---

[EvoSuite](https://www.evosuite.org/)是一个非常强大的自动化生成单元测试用例的工具，并且是免费的。提供了IDE插件，可以直接继承到Idea或者是Eclipse上。通过一个菜单就可以生成类对应的但愿测试用例。非常方便，可以极大提高开发的测试用例编写效率。



生成命令：
```language
/opt/apache-maven-3.6.1-mavenrep/bin/mvn  compile  evosuite:generate  -Dcores=1  -DmemoryInMB=1024  -DtimeInMinutesPerClass=2  -DspawnManagerPort=65021  -Dcuts=com.ubuntuvim.springcloud.controller.PaymentController  evosuite:export  -DtargetFolder=src/test/java  

```