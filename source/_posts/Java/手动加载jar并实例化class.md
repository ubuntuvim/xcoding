---
title: 手动加载jar并实例化class
tag:
	- Java
---



在Java项目中加载一个类的方式有很多，通常情况都是通过idea配置好的classPath加载到项目中。如果你想通过编码方式加载一个本地jar文件里面的class，要怎么做呢？

实现起来也很简单，Java提供了现成的API，直接通过`URLClassLoader`就可以加载本地硬盘任何路径下jar，并通过`newInstance()`方法就可以实例化一个类。请看下面的例子：

```java
package com.ubuntuvim.spring.reflection;

import org.junit.runner.JUnitCore;

import java.net.URL;
import java.net.URLClassLoader;

/**
 * 手动从硬盘加载class
 * 加载/Users/ubuntuvim/code/spring-framework/test-env/src/main/resources/lib/serverless-0.0.1-SNAPSHOT.jar
 * 这个jar里面的com.ubuntuvim.serverless.MyJarLoader这个类并且实例化
 */
public class LoaderFromDisk {

	public static void main(String[] args) throws Exception {
		// 编程new方式
//		JUnitCore jUnitCore = new JUnitCore();
//		System.out.println(jUnitCore.getVersion());

		// 手动加载类方式，这个类已经是在项目类路径下了
		String className = "org.junit.runner.JUnitCore";
		Class<?> clazz = LoaderFromDisk.class.getClassLoader().loadClass(className);
		JUnitCore o = (JUnitCore) clazz.newInstance();
		System.out.println(o.getVersion());

		// 手动加载不在项目类路径下的jar里面的类
		String jarPath = "file:///Users/ubuntuvim/code/spring-framework/test-env/src/main/resources/lib/serverless-0.0.1-SNAPSHOT.jar";
		String className1 = "com.ubuntuvim.serverless.MyJarLoader";
		// 对于不在项目类路径中的jar需要手动加载，否则找不到。报异常：java.lang.ClassNotFoundException
//		Object o1 = LoaderFromDisk.class.getClassLoader().loadClass(className1).newInstance();

		URL jarUrl = new URL(jarPath);
		URLClassLoader urlClassLoader = new URLClassLoader(new URL[]{ jarUrl });
		Object o1 = urlClassLoader.loadClass(className1).newInstance();
		System.out.println(o1);
	}
}
```

`serverless-0.0.1-SNAPSHOT.jar`这个jar包并不在项目的classPath中，并且没有配置到项目类路径下，而是单独放在硬盘的某个目录下。

**这种方式实现了类的热加载。**

另外一种更加扩展的使用方式是，定义一个接口（接口也是打包成一个公共jar在项目中引用），所有接口的实现类都可以通过插件的方式安装到项目。也就是说所有的实现类都可以在另外项目中实现接口，并且只要在需要的时候通过`URLClassLoader`加载进来就可以。如果不需要就不加载。实现即插即用。

