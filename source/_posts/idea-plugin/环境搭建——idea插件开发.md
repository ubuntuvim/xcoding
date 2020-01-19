---
title: 环境搭建——idea插件开发
tag:
  - Java
  - idea-plugin
---

## 开发工具

idea的插件肯定就是用idea。
下载社区版或者专业版都可以，链接：[https://www.jetbrains.com/idea/download/](https://www.jetbrains.com/idea/download/)。

## 编译工具配置

### JDK

至少1.8或者以上的JDK版本。JDK的下载安装、配置自行处理。

### Gradle
项目编译使用gradle，请自行安装配置gradle。
gradle下载地址：[https://services.gradle.org/distributions/](https://services.gradle.org/distributions/)

### 启用idea的 Plugin Devkit

在插件里面确认plugin devkit已经安装成功，如果没有则安装。

![devkit](/Users/ubuntuvim/Pictures/idea/1-7.png)

## 创建第一个插件项目



打开idea，从左上角的File里面创建一个gradle项目。

>  File -> New -> Project…

![选中Java、Intellij Platfrorm Plugin](/Users/ubuntuvim/Pictures/idea/1-1.png)

**Java、Intellij Platform Plugin**这两个是必须的，JDK至少`1.8`或者以上版本。**Kotlin(Java)**可选。继续下一步。

![group](/Users/ubuntuvim/Pictures/idea/1-2.png)

输入GroupId，ArtifactId，继续下一步。

![gradle设置](/Users/ubuntuvim/Pictures/idea/1-3.png)

设置自己安装的gradle。选中`Use auto-import`，JDK默认即可。继续下一步。

![项目名称](/Users/ubuntuvim/Pictures/idea/1-4.png)

输入项目名称。其他默认即可。最后点击Finish，等待gradle自动下载项目依赖，如果下载很慢可以引入本地 Intellij SDK。

Intellij SDK下载：[https://d2cico3c979uwg.cloudfront.net/com/jetbrains/intellij/idea/ideaIC/2019.1.4/ideaIC-2019.1.4.zip](https://d2cico3c979uwg.cloudfront.net/com/jetbrains/intellij/idea/ideaIC/2019.1.4/ideaIC-2019.1.4.zip)

我下载的是和当前Idea同一个版本的SDK。然后修改`build.gradle`文件的里面的`repositories`标签内容，引入本地SDK。

```groovy
repositories {
    maven {
        url '/Users/ubuntuvim/software/sdk/ideaIC-2019.1.4.zip'
    }
    maven {
        url 'https://dl.bintray.com/jetbrains/intellij-plugin-service'
    }
    maven { url 'https://cache-redirector.jetbrains.com/jetbrains.bintray.com/intellij-plugin-service' }
    maven { url 'https://cache-redirector.jetbrains.com/repo1.maven.org/maven2' }
    maven { url 'https://cache-redirector.jetbrains.com/jcenter.bintray.com' }
    mavenCentral()
}

```

自定义的路径一定记得放在`mavenCentral()`的前面，下载的顺序是如果前面的URL已经能获取到SDK则后面的不再下载，由于SDK比较大，通过maven下载会很慢。自己另外下载之后直接引入即可。

等待gradle自动编译完之后可以看到gradle的命令。

![gradle命令](/Users/ubuntuvim/Pictures/idea/1-5.png)

都配置好之后，创建一个简单的hello world程序验证环境是否搭建成功。

如下所示，创建一个简单的`Action`类，创建一个Plugin的类和普通类有所差别，在包上`右键 -> New -> Plugin Devkit -> Action `。

![Devkit](/Users/ubuntuvim/Pictures/idea/1-8.png)

简单设置一下Action的信息。

![Action Class](/Users/ubuntuvim/Pictures/idea/1-9.png)

第一个红色框里面是自定义的Action类的名字；

第二个红色框设置的意思是新建的Action菜单放在Idea的**Help**菜单下；

第三个红色框意思是把新增的Action菜单放在**Help**菜单下的一个子菜单。

其他默认即可。

类的内容很简单：

```java
package com.ubuntuvim.idea.plugin;

import com.intellij.openapi.actionSystem.AnAction;
import com.intellij.openapi.actionSystem.AnActionEvent;

/**
 * @Description： 第一个测试Action
 * @Author: ubuntuvim
 * @HomePage： http://xcoding.tech
 * @Date: 2020-01-20 00:16
 */
public class AppPlugin extends AnAction {

    @Override
    public void actionPerformed(AnActionEvent e) {
        System.out.println("Hello my plugin...");
    }

}
```

Action创建好之后，`idea-test-plugin2/src/main/resources/META-INF/plugin.xml`项目下的这个文件中会自动增加以下配置信息：

```xml
<!-- Add your actions here -->
<action id="com.ubuntuvim.idea.plugin" class="com.ubuntuvim.idea.plugin.AppPlugin" text="my-plugin"
        description="我的第一个plugin">
   <add-to-group group-id="HelpMenu" anchor="first"/>
</action>
```

最后，我们尝试启动一个自定义的Action，看下会是一个什么样的效果，和期待有没有。

直接运行新建的Action，第一次运行会比较慢，等待gradle自动编译，可以看到以下运行日志。

![run log](/Users/ubuntuvim/Pictures/idea/1-11.png)

运行完毕之后，会从新打开一个Idea窗口。新打开的窗口里面会包含了你自定义的Action，其实说白了就是自己扩展里IDE的功能。

新打开的Idea创建有可能是一个空的，你也可以直接打开一个本地已经创建过的项目。

![open window](/Users/ubuntuvim/Pictures/idea/1-12.png)

可以看到在新打开的创建中，在**Help**菜单下多了一个自定义的子菜单，点击这个子菜单，此时不会有任何反应，因为我们并没有实现任何操作，只是打印了一个控制台日志。回到原来开发插件的窗口，可以在控制台上看到输出了`System.out`日志。

![out log](/Users/ubuntuvim/Pictures/idea/1-14.png)

虽然很简单，但是起码验证了开发插件的环境是搭建成功了。另外，新打开的创建也是一个正常可以用的Idea窗口，你可以在里面创建、开发其他的项目。与正常的Idea窗口无差别。