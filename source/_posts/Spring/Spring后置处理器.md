---
title: Spring后置处理器
tag:
	- Spring
	- Java
---

本篇来总结汇总Spring各种类型后置处理器的使用。

### bean定义后置处理器

bean定义后置处理器是Spring框架提供的第一个扩展点。其中有两个接口，一个是`BeanFactoryPostProcessor`接口，一个是`BeanDefinitionRegistryPostProcessor`接口。

#### BeanFactroyPostProcessor接口

`BeanFactoryPostProcessor`接口提供的扩展功能：


> **允许用户修改容器中的bean定义信息，调整bean定义属性值。容器会在所有bean定义信加载完毕之后回调此接口，用以修改容器中的bean定义信息**。但是不要在此接口直接通过`getBean`实例化bean，这样会导致bean过早实例化，违反容器规则导致不可预知的副作用。
>
> 如果要实现bean实例化请通过`BeanPostProcessor`接口实现。
>
> 如果有多个`BeanFactoryPostProcessor`接口并且需要执行它们的执行顺序可以同时实现`PriorityOrdered`接口或者`Ordered`接口。
>
> 简单讲就是，我们可以通过实现此接口获取到`BeanFactory`对象（就是参数），操作`BeanFactory`对象，修改里面的`BeanDefinition`。
> **但是不要去实例化bean**。
>
> 接口的一个典型应用就是`PropertySourcesPlaceholderConfigurer`。

##### 接口源码

Spring框架`BeanFactoryPostProcessor`接口源码如下：

```java
package org.springframework.beans.factory.config;

import org.springframework.beans.BeansException;

/**
 * 允许自定义修改容器中的bean定义信息，调整bean定义属性值。
 * 容器会在所有bean定义信息加载完毕之后回调此接口，用以修改容器中的bean定义信息。
 * 但是不要在此接口直接通过getBean实例化bean，这样会导致bean过早实例化，违反容器规则导致不可预知的副作用。
 * 如果要实现bean实例化请通过BeanPostProcessor接口。
 * 如果有多个BeanFactoryPostProcessor接口并且需要执行它们的执行顺序可以同时实现PriorityOrdered接口或者Ordered接口。
 *
 * 简单讲就是，我们可以通过实现此接口获取到BeanFactory对象（就是参数），操作BeanFactory对象，修改里面的BeanDefinition。
 * 但是不要去实例化bean。
 * 
 * 另外有一点需要注意的是此接口的实现类会忽略懒加载设置，即使你显式设置了实现类懒加载也是不生效的。
 * 因为Spring需要保证BeanFactoryPostProcessor实现类优先实例化，如果实现类都懒加载了，那么你又如何能修改容器的bean定义呢。。。
 *
 * @author Juergen Hoeller
 * @author Sam Brannen
 * @since 06.07.2003
 * @see BeanPostProcessor
 * @see PropertyResourceConfigurer
 */
@FunctionalInterface
public interface BeanFactoryPostProcessor {

	/**
	 * 在所有的bean定义被加载到容器中后，并且是在所有bean实例化之前就会回调这个接口，
	 * 这个接口可以修改容器中的所有bean定义信息，包括重写某些bean的定义属性信息。
	 * 比如修改MyServiceImpl为懒加载：beanFactory.getMergedBeanDefinition(MyServiceImpl.class.getName()).setLazyInit(true);
	 * 另外一个很典型的应用就是修改bean定义中属性的占位符（PropertySourcesPlaceholderConfigurer），比如读取配置文件把配置文件的配置值注入到类属性上
	 * 最常见的就是@Value("${xxxx}")
	 *
	 * 注意：不要在此接口中实例化bean（就是不要调getBean()方法），提前实例化bean会导致不可预知的结果，
	 * 因为目前还处在解析完所有bean定义阶段，bean的实例化（实例化就是根据bean的定义信息创建实例对象）还在后面的阶段。
	 */
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;

}
```

##### 使用案例

自定义一个实现类，验证。

1. 修改某个bean的定义信息
2. 接口实现来显式设置为懒加载，看是否有效果（正常情况下应该是无效果的，Spring需要保证实现类提前初始化，否则谈何能修改bean定义）。容器启动的过程中就会打印构造方法的日志

```java
package com.ubuntuvim.spring.beanfactorypostprocessor;


import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Component;

/**
 * 使用Spring的一个扩展点，实现BeanFactoryPostProcess接口。
 * 1. 修改某个bean的定义信息
 * 2. 接口实现来显式设置为懒加载，看是否有效果（正常情况下应该是无效果的，Spring需要保证实现类提前初始化，否则谈何能修改bean定义）
 *		容器启动的过程中就会打印构造方法的日志
 *
 * 运行结果：
 * com.ubuntuvim.spring.processor.MyBeanFactoryPostProcessorImpl被加载了。。。
 * com.ubuntuvim.spring.bean.LazyLoadingBean被设置成懒加载了。
 *
 * 没有看到LazyLoadingBean被加载的日志，把beanFactory.getBeanDefinition(beanName).setLazyInit(true);改成false再运行：
 * com.ubuntuvim.spring.processor.MyBeanFactoryPostProcessorImpl被加载了。。。
 * com.ubuntuvim.spring.bean.LazyLoadingBean被设置成懒加载了。
 *
 * com.ubuntuvim.spring.bean.LazyLoadingBean被加载了。。。
 *
 * 可以看到LazyLoadingBean被加载了，完美的验证了前面的两点描述
 *
 * @Author: ubuntuvim
 * @Date: 2020/9/23 下午9:17
 */
@Component
@Lazy  // 显式指定为懒加载
public class MyBeanFactoryPostProcessorImpl implements BeanFactoryPostProcessor {
	public MyBeanFactoryPostProcessorImpl() {
		System.out.println("\n" + this.getClass().getName() + "被加载了。。。\n");
	}

	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
		// @Component注解没有指定名称，所有是默认首字母小写名字
		String beanName = "lazyLoadingBean";
		System.out.println("\n" + beanName + "被设置成懒加载了。\n");
		beanFactory.getBeanDefinition(beanName).setLazyInit(true);
	}
}
```

再定义一个普通bean。

```java
package com.ubuntuvim.spring.beanfactorypostprocessor;


import org.springframework.stereotype.Component;

/**
 * 这个bean在MyBeanFactoryPostProcessorImpl中被设置懒加载了，所以容器启动完毕也会不打印构造方法的日志
 * @Author: ubuntuvim
 * @Date: 2020/9/23 下午10:32
 */
@Component
public class LazyLoadingBean {
	public LazyLoadingBean() {
		System.out.println("\n" + this.getClass().getName() + "被加载了。。。\n");
	}
}
```

运行结果：

```shell
com.ubuntuvim.spring.beanfactorypostprocessor.MyBeanFactoryPostProcessorImpl被加载了。。。

lazyLoadingBean被设置成懒加载了。
```

`MyBeanFactoryPostProcessorImpl`被加载了，`LazyLoadingBean`没有被加载，把`beanFactory.getBeanDefinition(beanName).setLazyInit(true);`改成`false`再运行：

```shell
com.ubuntuvim.spring.beanfactorypostprocessor.MyBeanFactoryPostProcessorImpl被加载了。。。

lazyLoadingBean被设置成懒加载了。

com.ubuntuvim.spring.beanfactorypostprocessor.LazyLoadingBean被加载了。。。
```

可以看到`MyBeanFactoryPostProcessorImpl`被加载了，`LazyLoadingBean`也被加载了，完美符合预期。



#### 待跟进

学习`PropertyResourceConfigurer`是如何替换类中的占位符`@Value("${xxx}")`。



#### BeanDefinitionRegistryPostProcessor接口

`BeanDefinitionRegistryPostProcessor`接口的提供的扩展功能是：

> `BeanDefinitionRegistryPostProcessor`接口是`BeanFactoryPostProcessor`接口的子类，它在父类的基础上增加了`postProcessBeanDefinitionRegistry()`方法。允许用户获取`BeanDefinitionRegistry`对象，从而可以**通过编码方式动态修改、新增**`BeanDefinition`。
>
> 此接口一个非常重要的实现类就是`ConfigurationClassPostProcessor`，这个类用于解析`@Component`，`@Service`，`@ComponentScan`，`@Configuration`等注解，把注解对应的类转换成`BeanDefinition`然后注册到IoC容器中。

##### 接口源码

```java
package org.springframework.beans.factory.support;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;

/**
 * Extension to the standard {@link BeanFactoryPostProcessor} SPI, allowing for
 * the registration of further bean definitions <i>before</i> regular
 * BeanFactoryPostProcessor detection kicks in. In particular,
 * BeanDefinitionRegistryPostProcessor may register further bean definitions
 * which in turn define BeanFactoryPostProcessor instances.
 *
 * 扩展标准的BeanFactoryPostProcessor SPI，
 * 允许在常规BeanFactoryPostProcessor检测开始之前注册其他的bean定义，特别是，
 * BeanDefinitionRegistryPostProcessor可以注册其他的bean定义，
 * 这些定义反过来可以用于定义BeanFactoryPostProcessor实例。
 * （也就是说可以借此方法往容器中注入bean定义）一个典型的使用就是ConfigurationClassPostProcessor，
 * 这个类用于解析@Component，@Services，@ComponentScan，@Configuration等注解，把注解对应的类转换成BeanDefinition然后注册到IoC容器中。
 *
 * @author Juergen Hoeller
 * @since 3.0.1
 * @see org.springframework.context.annotation.ConfigurationClassPostProcessor
 */
public interface BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor {

	/**
	 * Modify the application context's internal bean definition registry after its
	 * standard initialization. All regular bean definitions will have been loaded,
	 * but no beans will have been instantiated yet. This allows for adding further
	 * bean definitions before the next post-processing phase kicks in.
	 * 
	 * 在标准bean初始化前修改、新增bean定义。
	 * @param registry the bean definition registry used by the application context
	 * @throws org.springframework.beans.BeansException in case of errors
	 */
	void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException;

}
```

##### 使用案例

自定义一个实现类，通过编码的方式往容器注入一个bean定义。

```java
package com.ubuntuvim.spring.beanfactorypostprocessor;


import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor;
import org.springframework.beans.factory.support.RootBeanDefinition;
import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Component;

/**
 * 通过编程方式注册InjectBeanFromPostProcessor
 * 同样的本类设置成懒加载也是无效的
 * @Author: ubuntuvim
 * @Date: 2020/7/17 20:43
 */
@Component
@Lazy
public class MyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {
	public MyBeanDefinitionRegistryPostProcessor() {
		System.out.println("\n" + this.getClass().getName() + "被加载了。。。\n");
	}
	@Override
	public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
		//编程方式注入一个bean定义
		registry.registerBeanDefinition(InjectBeanFromPostProcessor.class.getName(),
				new RootBeanDefinition(InjectBeanFromPostProcessor.class));
	}

	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
    // 本方法的功能和BeanFactoryPostProcessor一样。本来就是从BeanFactoryPostProcessor继承过来的。
	}
}
```

`InjectBeanFromPostProcessor`并没有使用任何注解，也没有通过其他方式导入。

```java
package com.ubuntuvim.spring.beanfactorypostprocessor;


/**
 * 通过BeanDefinitionRegistryPostProcessor接口注入本类到容器中。
 * 并没有在类上使用任何注解，也没有通过其他方式导入容器，期望效果是容器启动完毕之后会打印构造方法的日志
 * @Author: ubuntuvim
 * @Date: 2020/9/23 下午11:00
 */
public class InjectBeanFromPostProcessor {
	public InjectBeanFromPostProcessor() {
		System.out.println("\n" + this.getClass().getName() + "被加载了。。。\n");
	}
}
```

运行结果：

```java
com.ubuntuvim.spring.beanfactorypostprocessor.MyBeanDefinitionRegistryPostProcessor被加载了。。。

com.ubuntuvim.spring.beanfactorypostprocessor.InjectBeanFromPostProcessor被加载了。。。

```

结果符合预期，`InjectBeanFromPostProcessor`成功注册到IoC容器中，并且可以被IoC容器实例化。

以上两个接口就是Spring框架提供的第一个扩展点，用于修改为实例化之前的bean定义信息。


#### ConfigurationClassPostProcessor接口

##### 接口源码

##### 使用案例



####  SmartInitializingSingleton接口

**这个接口Spring4.1之后才有**

> `SmartInitializingSingleton`是spring 4.1中引入的新特效，与`InitializingBean`的功能类似，都是**bean实例化后执行自定义初始化**，都是属于[spring bean生命周期](https://blog.csdn.net/alex_xfboy/article/details/51211054)的增强。但是，`SmartInitializingSingleton`的**定义及触发方式方式上有些区别**，它的定义不在当前的bean中（a bean's local construction phase），它是回调接口（针对**非lazy单例Bean**），回调的操作是由spring事件`ContextRefreshedEvent`触发。



##### 接口源码

```java
package org.springframework.beans.factory;

/**
 * 实现该接口后，当所有单例 bean 都初始化完成以后， 容器会回调该接口的方法 afterSingletonsInstantiated。
 * 主要应用场合就是在所有单例 bean 创建完成之后，可以在该回调中做一些事情。
 * @PostConstruct是最先被执行的，然后是InitializingBean，最后是SmartInitializingSingleton
 *
 * 为什么是在当所有单例 bean 都初始化完成以后才执行这个接口的原因直接看源码就知道了：
 * AbstractApplicationContext.refresh() -> finishBeanFactoryInitialization() -> ConfigurableListableBeanFactory.preInstantiateSingletons()
 *
 * 但是需要注意：不要再次接口中提前使用容器管理的bean对象，
 * 因为此时直接通过getBean()方法获取到的实例还没通过IoC容器的其他初始化后置处理的增强。
 *
 * @since 4.1
 */
public interface SmartInitializingSingleton {

	/**
	 * 所有单例对象都是实例化完成之后就会回调这个接口实现类的此方法。
	 */
	void afterSingletonsInstantiated();

}
```

`SmartInitializingSingleton`接口的实现主要是Spring框架内部使用，目前Spring框架内部已经有差不多30个实现类。

![SmartInitializingSingleton接口实现类](https://oscimg.oschina.net/oscnet/up-677ccab99e962d5562335695f80b97a8340.png)

一个很典型的应用是`EventListenerMethodProcessor`类，这个类的作用的是用来对 `@EventListener` 提供支持.

主要是标注了`@EventListener` 的方法进行解析, 然后转换为一个 `ApplicationListener`。解析的方法就是实现了`SmartInitializingSingleton`接口的`afterSingletonsInstantiated()`方法，在这个方法中处理。

##### 使用案例

定义一个实现类，同时实现了`SmartInitializingSingleton`接口和`InitializingBean`接口，并且在类中使用`@PostConstruct`注解。验证这几种方式的初始化执行顺序。

```java
package com.ubuntuvim.spring.beanpostprocess;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.SmartInitializingSingleton;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;

/**
 * 在所有bean实例化之后（初始化前）回调这个接口afterSingletonsInstantiated
 * 初始化操作执行顺序：@PostConstruct是最先被执行的，然后是InitializingBean，最后是SmartInitializingSingleton
 */
@Component
public class MySmartInitializingSingletonImpl implements SmartInitializingSingleton, ApplicationContextAware, InitializingBean {

	ApplicationContext applicationContext;

	@PostConstruct
	public void invokePostConstruct() {
		System.out.println("1. @PostConstruct注释方法被执行");
	}

	@Override
	public void afterSingletonsInstantiated() {
		System.out.println("3. SmartInitializingSingleton接口的afterSingletonsInstantiated()方法被执行了");
		InitBean initBean = applicationContext.getBean(InitBean.class);
		initBean.f();
	}

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		this.applicationContext = applicationContext;
	}

	@Override
	public void afterPropertiesSet() throws Exception {
		System.out.println("2. InitializingBean接口的afterPropertiesSet()方法被执行了");
	}
}
```

执行结果：

```shell
1. @PostConstruct注释方法被执行
2. InitializingBean接口的afterPropertiesSet()方法被执行了
3. SmartInitializingSingleton接口的afterSingletonsInstantiated()方法被执行了
com.ubuntuvim.spring.beanpostprocess.InitBean的方法f()被调用
```



#### InstantiationAwareBeanPostProcessor接口或者InstantiationAwareBeanPostProcessorAdapter

##### 接口源码

##### 使用案例


#### MergedBeanDefinitionPostProcessor接口

##### 接口源码

##### 使用案例



#### SmartInstantiationAwareBeanPostProcessor接口

##### 接口源码

##### 使用案例




#### BeanFactoryAware/ApplicationContextAware/BeanNameAware接口

##### 接口源码

##### 使用案例

#### CommonAnnotationBeanPostProcessor

##### 接口源码

##### 使用案例

#### AutowiredAnnotationBeanPostProcessor

##### 接口源码

##### 使用案例


#### InitDestroyAnnotationBeanPostProcessor接口

##### 接口源码

##### 使用案例





#### 参考资料

https://fangjian0423.github.io/2017/06/20/spring-bean-post-processor/

