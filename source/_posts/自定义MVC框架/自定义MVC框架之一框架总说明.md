---
title: 自定义MVC框架之一框架总说明
tags: 
    - MVC
    - Java
    - Spring
---

本系列文章将为你介绍一个简单的自定义的MVC框架，主要是用于学习，框架模拟struts实现。

**项目结构如下：**

![项目结构](http://7xnrhh.com1.z0.glb.clouddn.com/QQ%E5%9B%BE%E7%89%8720160303141839.png)

1. LoginAction.java 测试，模拟登陆处理
2. Action.java 框架Action接口
3. ActionManager.java 根据配置的Action类名反射得到实例
4. ActionMapping.java 根据Action配置定义的javabean类，用于保存Action配置信息
5. ActionMappingManager.java 读取、解析Action配置并把配置转换成对应的ActionMapping对象
6. CharactorFilter.java 编码过滤器
7. ActionServlet.java 框架拦截器，根据web.xml的配置拦截请求
8. snails-actions-validate.xsd Action配置的校验文件，此文件限定了Action配置的格式
9. snails-actions.xml Action配置，类似于struts，配置了Action的名称、类、结果页面
10. 依赖jar commons-lang3-3.1.jar dom4j-1.6.1.jar
11. web.xml项目总配置文件
12. fail.jsp 登录失败页面后跳转页面
13. index.jsp登录页面
14. success.jsp 登录成功后跳转页面

### 框架执行流程

![执行流程](http://7xnrhh.com1.z0.glb.clouddn.com/%E8%87%AA%E5%AE%9A%E4%B9%89MVC%E6%A1%86%E6%9E%B6%EF%BC%8C%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png)

重点在框架拦截器`ActionServlet`，此拦截器负责初始化配置、根据`Action`实现类处理结果跳转不同的试图页面。
另一个需要注意的类是`ActionMappingManager`此类负责配置文件的解析。

框架的介绍从`Action`的配置开始，因为本框架很大程度上都是围绕着这个配置文件开展的。

1. [自定义MVC框架之二action配置文件定义](https://github.com/ubuntuvim/study-note/blob/master/%E8%87%AA%E5%AE%9A%E4%B9%89MVC%E6%A1%86%E6%9E%B6/%E8%87%AA%E5%AE%9A%E4%B9%89MVC%E6%A1%86%E6%9E%B6%E4%B9%8B%E4%BA%8Caction%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E5%AE%9A%E4%B9%89.md)
2. [自定义MVC框架之三Action接口定义](https://github.com/ubuntuvim/study-note/blob/master/%E8%87%AA%E5%AE%9A%E4%B9%89MVC%E6%A1%86%E6%9E%B6/%E8%87%AA%E5%AE%9A%E4%B9%89MVC%E6%A1%86%E6%9E%B6%E4%B9%8B%E4%B8%89Action%E6%8E%A5%E5%8F%A3%E5%AE%9A%E4%B9%89.md)
3. [自定义MVC框架之四ActionMapping定义](https://github.com/ubuntuvim/study-note/blob/master/%E8%87%AA%E5%AE%9A%E4%B9%89MVC%E6%A1%86%E6%9E%B6/%E8%87%AA%E5%AE%9A%E4%B9%89MVC%E6%A1%86%E6%9E%B6%E4%B9%8B%E5%9B%9BActionMapping%E5%AE%9A%E4%B9%89.md)
4. [自定义MVC框架之五配置文件解析器ActionMappingManager定义](https://github.com/ubuntuvim/study-note/blob/master/%E8%87%AA%E5%AE%9A%E4%B9%89MVC%E6%A1%86%E6%9E%B6/%E8%87%AA%E5%AE%9A%E4%B9%89MVC%E6%A1%86%E6%9E%B6%E4%B9%8B%E4%BA%94%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E8%A7%A3%E6%9E%90%E5%99%A8ActionMappingManager%E5%AE%9A%E4%B9%89.md)
5. [自定义MVC框架之六ActionManager定义](https://github.com/ubuntuvim/study-note/blob/master/%E8%87%AA%E5%AE%9A%E4%B9%89MVC%E6%A1%86%E6%9E%B6/%E8%87%AA%E5%AE%9A%E4%B9%89MVC%E6%A1%86%E6%9E%B6%E4%B9%8B%E5%85%ADActionManager%E5%AE%9A%E4%B9%89.md)
6. [自定义MVC框架之七框架拦截器ActionServlet定义、配置](https://github.com/ubuntuvim/study-note/blob/master/%E8%87%AA%E5%AE%9A%E4%B9%89MVC%E6%A1%86%E6%9E%B6/%E8%87%AA%E5%AE%9A%E4%B9%89MVC%E6%A1%86%E6%9E%B6%E4%B9%8B%E4%B8%83%E6%A1%86%E6%9E%B6%E6%8B%A6%E6%88%AA%E5%99%A8ActionServlet%E5%AE%9A%E4%B9%89%E3%80%81%E9%85%8D%E7%BD%AE.md)
7. [自定义MVC框架之八使用框架模拟登陆](https://github.com/ubuntuvim/study-note/blob/master/%E8%87%AA%E5%AE%9A%E4%B9%89MVC%E6%A1%86%E6%9E%B6/%E8%87%AA%E5%AE%9A%E4%B9%89MVC%E6%A1%86%E6%9E%B6%E4%B9%8B%E5%85%AB%E4%BD%BF%E7%94%A8%E6%A1%86%E6%9E%B6%E6%A8%A1%E6%8B%9F%E7%99%BB%E9%99%86.md)


希望通过上述的文章能给你一点收获，如果你能理解其中的思想对于你学习Java三大框架SSH是非常有帮助的。


项目完整代码请看[MyMVC](https://github.com/ubuntuvim/myMVC)，欢迎fork学习，如果你觉得对你有帮助给我点个赞吧，当然也欢迎给我提意见（email:1527254027@qq.com，chendequanroob@gmail.com）。
