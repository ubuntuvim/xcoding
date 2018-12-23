---
title: spring-gradle-代码覆盖测试
tag:
	- Spring
	- Gradle
	- Java
---

**单元测试、代码覆盖率**

- [junit-jacoco代码覆盖测试](http://xcoding.tech/2018/12/06/junit-jacoco/junit-jacoco%E4%BB%A3%E7%A0%81%E8%A6%86%E7%9B%96%E6%B5%8B%E8%AF%95/)
- [spring-junit-jacoco代码覆盖测试](http://xcoding.tech/2018/12/09/junit-jacoco/spring-junit-jacoco%E4%BB%A3%E7%A0%81%E8%A6%86%E7%9B%96%E6%B5%8B%E8%AF%95/)


接着上一篇《spring-junit-jacoco代码覆盖测试》，本篇介绍一下如何gradle单元测试、代码覆盖率。

### 准备

* gradle。请自行安装好gradle

### 使用maven转为gradle项目

在项目根目录下执行如下命令把maven项目转为gradle项目。

```shell
gradle init --type pom
```

命令执行完毕之后刷新项目可以看到项目目录下已经有gradle的配置`build.gradle`、`settings.gradle`等配置。

修改`settings.gradle`里面的项目名称，改为`coverage-spring-gradle`。如果不改为和项目同名字使用gradle编译的时候会报错。

```shell
org.eclipse.buildship.core.UnsupportedConfigurationException: Project at '/Users/ubuntuvim/codes/java_keeping/coverage-spring-gradle' can't be named 'coverage-spring-gradle33' because it's located directly under the workspace root. If such a project is renamed, Eclipse would move the container directory. To resolve this problem, move the project out of the workspace root or configure it to have the name 'coverage-spring-gradle'.
	at org.eclipse.buildship.core.workspace.internal.DefaultWorkspaceOperations.validateProjectName(DefaultWorkspaceOperations.java:183)
	at org.eclipse.buildship.core.workspace.internal.ProjectNameUpdater.checkProjectName(ProjectNameUpdater.java:107)
	at org.eclipse.buildship.core.workspace.internal.ProjectNameUpdater.updateProjectName(ProjectNameUpdater.java:44)
	at org.eclipse.buildship.core.workspace.internal.SynchronizeGradleBuildOperation.synchronizeOpenWorkspaceProject(SynchronizeGradleBuildOperation.java:208)
	at org.eclipse.buildship.core.workspace.internal.SynchronizeGradleBuildOperation.synchronizeWorkspaceProject(SynchronizeGradleBuildOperation.java:186)
	at org.eclipse.buildship.core.workspace.internal.SynchronizeGradleBuildOperation.synchronizeGradleProjectWithWorkspaceProject(SynchronizeGradleBuildOperation.java:176)
	at org.eclipse.buildship.core.workspace.internal.SynchronizeGradleBuildOperation.access$000(SynchronizeGradleBuildOperation.java:99)
	at org.eclip
```

#### 删除maven依赖文件

使用gradle之后就不需要maven的文件了，删除`master-lib`,`coverage.xml`,`compile.xml`,`jacocolib`。


项目右键 --> gradle --> refresh gradle project。之后项目结构如下图：

![报告](/image/blog-image/java/junit-coverage/9.png)

### 运行生成单元测试报告

打开gradle视图

![报告](/image/blog-image/java/junit-coverage/10.png)

可以看到gradle已经提供了很多任务，直接运行即可，我们不需要像使用Ant一样自己编写一大堆的脚本。

![build任务](/image/blog-image/java/junit-coverage/11.png)

直接运行`build` -> `build`这个任务。稍等执行完毕之后，刷新项目，打开项目目录下`build`目录，展开之后可以看到一个index.html。这个文件就是单元测试报告。

![build任务](/image/blog-image/java/junit-coverage/12.png)

生成一个单元测试报告就是这么简单，什么脚本都不需要写了，gradle已经帮你做好。

### 生成覆盖率报告

要构建一个覆盖率报告也是非常简单的。仍然是使用gradle提供好的插件，修改`build.gradle`，在文件中增加覆盖率插件。

#### 增加覆盖率插件

首先引入jacoco插件。在`build.gradle`第三行增加：

```groovy
apply plugin: 'jacoco'
```

在这个文件末尾增加如下配置：

```groovy
// jacoco任务配置
jacoco {
	// 设置覆盖率报告目录
	reportsDir = file("$buildDir/coverage-report")
}
jacocoTestReport {
   reports {
      xml.enabled = true
      html.enabled = true
   }
}
```

#### 生成覆盖率报告

进入项目根目录，直接如下命令即可生成覆盖率报告。

```shell
gradle clean test jacocoTestReport
```

执行过程需要下载一下依赖jar，稍等几分钟之后刷新项目，一步步展开`build`目录。

![报告文件](/image/blog-image/java/junit-coverage/13.png)

打开`index.html`可以看到详细报告内容。

![覆盖率报告](/image/blog-image/java/junit-coverage/14.png)

使用gradle是在简单很多，根本不需要你编写一大堆的脚本。非常爽。

### 项目源码

[https://github.com/ubuntuvim/coverage/tree/coverage-spring-gradle](https://github.com/ubuntuvim/coverage/tree/coverage-spring-gradle)
