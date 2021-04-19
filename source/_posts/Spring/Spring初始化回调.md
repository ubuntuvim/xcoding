---
title: Spring初始化回调
tag:
	- Spring
---

Spring框架提供了多种方式控制bean初始化的过程，开发者可以自定义初始化的逻辑。

有如下几种自定义bean初始化逻辑的方式：

1. 使用`@PostConstruct`注解
2. 自定义初始化方法，比如`@Bean(initMethod = "method1")`
3. 实现`InitializingBean`接口的`afterPropertiesSet()`方法
4. 实现`SmartInitializingSingleton`接口的`afterSingletonsInstantiated()`方法



这四种方式都可以用于自定义bean的初始化。但是每种方法执行的时机不太一样，通过Spring源码来给大家逐一介绍。

首先自定义一个bean，分别实现上述4中方式。

```java
package com.ubuntuvim.spring.createbean;

import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.SmartInitializingSingleton;
import org.springframework.beans.factory.annotation.InitDestroyAnnotationBeanPostProcessor;
import org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory;
import org.springframework.beans.factory.support.RootBeanDefinition;

import javax.annotation.PostConstruct;

/**
 * 验证不同方式初始化回调的执行顺序：
 * 0. 执行初始化回调BeanPostProcessor.postProcessBeforeInitialization()方法
 * 1. 执行初始化回调@PostConstruct注解定义的方法
 * 2. 执行初始化回调InitializingBean.afterPropertiesSet()方法
 * 3. 执行初始化回调@Bean(initMethod = "beanInit")定义的方法
 * 4. 执行初始化回调BeanPostProcessor.postProcessAfterInitialization()方法
 * 5. 执行初始化回调SmartInitializingSingleton.afterSingletonsInstantiated()方法，
 * 		此回调是在bean实例化和初始化完成之后执行的
 *
 * @Author: ubuntuvim
 * @Date: 2020/10/18 上午1:16
 */
public class InitCallbackBean implements InitializingBean, SmartInitializingSingleton {

	/**
	 * 如果一个类中多个方法都使用@PostConstruct注解声明，则会根据方法名按照字母升序顺序执行。
	 * 比如init()和afterPropertiesSet()都加了注解，那么先执行afterPropertiesSet()方法
	 */
	@PostConstruct
	public void init() {
		System.out.println("执行初始化回调@PostConstruct注解定义的方法");
	}

	@PostConstruct
	public void init2() {
		System.out.println("执行初始化回调@PostConstruct注解定义的方法init2()");
	}

	/**
	 * 如果用户在afterPropertiesSet()方法上也使用了@PostConstruct注解，此方法在后置处理器中会先被执行
	 * @see InitDestroyAnnotationBeanPostProcessor#postProcessBeforeInitialization(java.lang.Object, java.lang.String)
	 * 然后在执行初始化回调接口InitializingBean时不会再次执行。
	 * @see AbstractAutowireCapableBeanFactory#invokeInitMethods(String, Object, RootBeanDefinition)
	 * 也就是说同时在回调接口InitializingBean上同时使用@PostConstruct注解只会执行一次，但是可以改变方法的执行时机，
	 * 比如本类中init()和afterPropertiesSet()都加了@PostConstruct注解，那么先执行afterPropertiesSet()方法，
	 * 如果afterPropertiesSet()方法不加@PostConstruct注解，那么会先执行init()方法，再执行afterProperties()方法。
	 * 因为@PostConstruct注解优先于回调接口InitializingBean执行
	 *
	 * 如果@Bean注解中自定义的方法也使用@PostConstruct注解声明，那么这个方法会重复执行。
	 * 1. 后置处理器处理@PostConstruct注解方法的时候执行一次，
	 * 2. 执行自定义的方法的时候也执行一次
	 *
	 * @Bean(initMethod = "afterPropertiesSet")
	 * 如果自定义的方法和InitializingBean的回调方法一致。
	 * afterPropertiesSet()方法会执行一次，只执行InitializingBean的回调，不会在执行自定义方法的回调。
	 * 原因可以看Spring源码，位置：
	 * @see AbstractAutowireCapableBeanFactory#invokeInitMethods(String, Object, RootBeanDefinition)
	 *
	 * @Bean(initMethod = "initBean")
	 * 如果自定义的方法也使用了@PostConstruct声明，那么initBean()会执行两次。
	 * 1. 后置处理器处理@PostConstruct注解方法的时候执行一次，
	 * 2. 执行自定义的方法的时候也执行一次
	 *
	 * 总结一句话，InitializingBean.afterPropertiesSet()方法只会执行一次。
	 * 			@PostConstruct 声明的方法都会执行，不管有几个，执行顺序按照方法名升序执行
	 */
//	@PostConstruct
	@Override
	public void afterPropertiesSet() throws Exception {
		System.out.println("执行初始化回调InitializingBean.afterPropertiesSet()方法");
	}

	@Override
	public void afterSingletonsInstantiated() {
		System.out.println("执行初始化回调SmartInitializingSingleton.afterSingletonsInstantiated()方法");
	}

	@PostConstruct
	private void beanInit() {
		System.out.println("执行初始化回调@Bean(initMethod = \"beanInit\")定义的方法");
	}
}
```

通过`@Bean`方式把此类注入到Spring容器中。
```java
@ComponentScan
@Configuration
public class Config {
	@Bean(initMethod = "beanInit")
	public InitCallbackBean initCallbackBean() {
		return new InitCallbackBean();
	}
}
```

首先运行看结果是怎么样的。
```shell
执行初始化回调@PostConstruct注解定义的init方法
执行初始化回调@Bean(initMethod = "beanInit")定义的方法，并且此方法也使用了@PostConstruct注解
执行初始化回调@PostConstruct注解定义的方法init2()
执行初始化回调InitializingBean.afterPropertiesSet()方法
执行初始化回调@Bean(initMethod = "beanInit")定义的方法，并且此方法也使用了@PostConstruct注解
执行初始化回调SmartInitializingSingleton.afterSingletonsInstantiated()方法
```
通过运行结果分析，可以看到使用`@PostConstruct`注解的方法会第一个执行，`beanInit`方法同时使用了`@PostConstruct`注解，并且同时在`@Bean`中也指定成bean自定义的初始化方法，所以这个方法执行了两次。如果同一个类中有个多个方法都使用了`@PostConstruct`注解，那么这些被注解的方法都会被执行，但是执行的顺序不一定是方法在类中的代码顺序。

做点好玩的事情，在`afterPropertiesSet()`方法上也使用`@PostConstruct`注解，再运行看看结果是怎么样，会不会执行两遍？

```shell
执行初始化回调@PostConstruct注解定义的init方法
执行初始化回调InitializingBean.afterPropertiesSet()方法
执行初始化回调@Bean(initMethod = "beanInit")定义的方法，并且此方法也使用了@PostConstruct注解
执行初始化回调@PostConstruct注解定义的方法init2()
执行初始化回调@Bean(initMethod = "beanInit")定义的方法，并且此方法也使用了@PostConstruct注解
执行初始化回调SmartInitializingSingleton.afterSingletonsInstantiated()方法
```
这个就比较有意思了，可以看到`afterPropertiesSet()`方法只执行了一遍。

这几种初始化方法执行顺序是怎么设定的呢？为何使用`@PostConstruct`注解的方法可以执行多次，而`afterPropertiesSet()`方法却执行一次呢？
带这些疑问，我们阅读Spring源码，从源码中找答案。

###  不同初始化方式的执行时机

这些初始化方法都是在有bean实例之后执行的，我们可以大概的猜测到这些初始化回调应该是在Spring框架完成bean实例化，进行属性填充的时候执行的。
> 如果你对Spring框架bean的实例化、初始化不是很了解的请看之前的系列博客：https://www.toutiao.com/i6887194772914733571

直接打开如下类方法，
```java
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean
```
这个方法是Spring容器非常核心的方法，它处理了很多事情：单例实例化、填充属性、执行初始化回调、注册销毁回调方法。

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
  throws BeanCreationException {

  // Instantiate the bean.
  BeanWrapper instanceWrapper = null;
  if (mbd.isSingleton()) {
    // 从缓存中查询，如果是bean定义是一个bean工厂实例可以直接拿到。
    instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
  }
  if (instanceWrapper == null) {
    // 普通bean，根据bean定义创建bean实例，并包装成BeanWrapper返回
    instanceWrapper = createBeanInstance(beanName, mbd, args);
  }
  // 获取经过jdk1.8的Optional类包装过的非空对象
  final Object bean = instanceWrapper.getWrappedInstance();
  // 获取bean的class类型
  Class<?> beanType = instanceWrapper.getWrappedClass();
  if (beanType != NullBean.class) {
    mbd.resolvedTargetType = beanType;
  }

  // Allow post-processors to modify the merged bean definition.
  synchronized (mbd.postProcessingLock) {
    if (!mbd.postProcessed) {
      try {
        // 执行后置处理器接口MergedBeanDefinitionPostProcessor，bean实例化之后，就可以通过反射获取到类或者属性上的注释信息
        // 处理@Resource、@Autowired、@Value注解的定义信息，并把这些注解的定义信息放在缓存中。待后续属性填充的时候使用。
        // 如果有则吧注解信息转换成AutowiredFieldElement对象或者AutowiredMethodElement对象或者ResourceElement对象
        // 实现类有：AutowiredAnnotationBeanPostProcessor、CommonAnnotationBeanPostProcessor等
        applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
      }
      catch (Throwable ex) {
        throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                        "Post-processing of merged bean definition failed", ex);
      }
      mbd.postProcessed = true;
    }
  } 

  // Eagerly cache singletons to be able to resolve circular references
  // even when triggered by lifecycle interfaces like BeanFactoryAware.
  // 单例bean && 允许循环依赖 && bean正在被创建
  boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                                    isSingletonCurrentlyInCreation(beanName));
  if (earlySingletonExposure) {
    if (logger.isTraceEnabled()) {
      logger.trace("Eagerly caching bean '" + beanName +
                   "' to allow for resolving potential circular references");
    }
    /**
			 * 执行后置处理器SmartInstantiationAwareBeanPostProcessor的getEarlyBeanReference()方法尝试获取一个早期的引用。
			 * 并加入的单例工厂缓存中
			 * @see DefaultSingletonBeanRegistry#addSingletonFactory(String, ObjectFactory)
			 * @see #getEarlyBeanReference
			 */
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
  }

  // Initialize the bean instance.
  // 初始化bean实例，填充属性，注入依赖（@Autowired，@Resource，@Value）注解的属性
  Object exposedObject = bean;
  try {
    populateBean(beanName, mbd, instanceWrapper);
    /**
			 执行bean的初始化回调方法以及执行后置处理器的初始化方法，包括：
			一，执行Aware接口 ，包括BeanFactoryAware，BeanClassLoaderAware，BeanNameAware
				注意：ApplicationContext的注入是在另外一个后置处理器ApplicationContextAwareProcessor中执行。
	 		 二，执行bean初始化回调，包括：
				0. 执行初始化回调BeanPostProcessor.postProcessBeforeInitialization()方法
				1. 执行初始化回调@PostConstruct注解定义的方法
	 			2. 执行初始化回调InitializingBean.afterPropertiesSet()方法
				3. 执行初始化回调@Bean(initMethod = "beanInit")定义的初始化方法beanInit()
	 			4. 执行初始化回调BeanPostProcessor.postProcessAfterInitialization()方法
				5. 执行初始化回调SmartInitializingSingleton.afterSingletonsInstantiated()方法
				按照上述执行顺序执行
			 */
    exposedObject = initializeBean(beanName, exposedObject, mbd);
  }
  catch (Throwable ex) {
    if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
      throw (BeanCreationException) ex;
    }
    else {
      throw new BeanCreationException(
        mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
    }
  }
  // 循环依赖检查
  if (earlySingletonExposure) {
    Object earlySingletonReference = getSingleton(beanName, false);
    if (earlySingletonReference != null) {
      if (exposedObject == bean) {
        exposedObject = earlySingletonReference;
      }
      else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
        String[] dependentBeans = getDependentBeans(beanName);
        Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
        for (String dependentBean : dependentBeans) {
          if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
            actualDependentBeans.add(dependentBean);
          }
        }
        if (!actualDependentBeans.isEmpty()) {
          throw new BeanCurrentlyInCreationException(beanName,
                                                     "Bean with name '" + beanName + "' has been injected into other beans [" +
                                                     StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
                                                     "] in its raw version as part of a circular reference, but has eventually been " +
                                                     "wrapped. This means that said other beans do not use the final version of the " +
                                                     "bean. This is often the result of over-eager type matching - consider using " +
                                                     "'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.");
        }
      }
    }
  }

  // Register bean as disposable.
  try {
    /**
			 * 注册bean销毁回调方法，这个方法和前面的initializeBean()方法是对应的。通常情况下初始化方法和销毁方法是同时出现的。
			 * 比如回调DisposableBean接口的destroy()方法，需要注意的是这里只是注册，并不会执行销毁回调方法。
			 * 销毁方法的调用是在手动执行容器的关闭方法的时候：
			 * @see org.springframework.context.support.AbstractApplicationContext#close()
			 *
			 * @see AbstractBeanFactory#registerDisposableBeanIfNecessary(String, Object, RootBeanDefinition)
			 */
    registerDisposableBeanIfNecessary(beanName, bean, mbd);
  }
  catch (BeanDefinitionValidationException ex) {
    throw new BeanCreationException(
      mbd.getResourceDescription(), beanName, "Invalid destruction signature", ex);
  }
  /**
		 * 回到调用处：
		 * @see #createBean(String, RootBeanDefinition, Object[])
		 */
  return exposedObject;
}
```
其中`initializeBean(beanName, exposedObject, mbd)`这行是本次我们需要关注的，所有的初始化回调都是在这个方法中执行的。
进入这个方法的代码：
```java
/**
	 * Initialize the given bean instance, applying factory callbacks
	 * as well as init methods and bean post processors.
	 * <p>Called from {@link #createBean} for traditionally defined beans,
	 * and from {@link #initializeBean} for existing bean instances.
	 * 本方法的作用有
	 * 一，执行Aware接口 ，包括BeanFactoryAware，BeanClassLoaderAware，BeanNameAware
   * 		注意：ApplicationContext的注入是在另外一个后置处理器ApplicationContextAwareProcessor中执行。
	 * 二，执行bean初始化回调，包括：
	 * 		0. 执行初始化回调BeanPostProcessor.postProcessBeforeInitialization()方法
	 * 		1. 执行初始化回调@PostConstruct注解定义的方法
	 * 		2. 执行初始化回调InitializingBean.afterPropertiesSet()方法
	 * 		3. 执行初始化回调@Bean(initMethod = "beanInit")定义的方法
	 * 		4. 执行初始化回调BeanPostProcessor.postProcessAfterInitialization()方法
	 * 		5. 执行初始化回调SmartInitializingSingleton.afterSingletonsInstantiated()方法
	 * 按照上述执行顺序执行
	 *
	 * @param beanName the bean name in the factory (for debugging purposes)
	 * @param bean the new bean instance we may need to initialize
	 * @param mbd the bean definition that the bean was created with
	 * (can also be {@code null}, if given an existing bean instance)
	 * @return the initialized bean instance (potentially wrapped)
	 * @see BeanNameAware
	 * @see BeanClassLoaderAware
	 * @see BeanFactoryAware
	 * @see #applyBeanPostProcessorsBeforeInitialization
	 * @see #invokeInitMethods
	 * @see #applyBeanPostProcessorsAfterInitialization
	 * 方法调用处：
	 * @see #doCreateBean(String, RootBeanDefinition, Object[]) 
	 */
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
  // 如果bean实现了XxxAware接口，则调用这些接口的setXxx()方法
  if (System.getSecurityManager() != null) {
    AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
      invokeAwareMethods(beanName, bean);
      return null;
    }, getAccessControlContext());
  }
  else {
    invokeAwareMethods(beanName, bean);
  }

  /**
		 * 执行后置处理器的postProcessorBeforeInitialization()方法
		 * 这里会首先执行第一个初始化回调@PostConstruct声明的方法，是这个类实现的
		 * @see org.springframework.beans.factory.annotation.InitDestroyAnnotationBeanPostProcessor#postProcessBeforeInitialization
		 */
  Object wrappedBean = bean;
  if (mbd == null || !mbd.isSynthetic()) {
    wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
  }

  try {
    // 执行InitializingBean接口和自定义的初始化方法（@Bean(initMethod = "beanInit"))
    invokeInitMethods(beanName, wrappedBean, mbd);
  }
  catch (Throwable ex) {
    throw new BeanCreationException(
      (mbd != null ? mbd.getResourceDescription() : null),
      beanName, "Invocation of init method failed", ex);
  }
  /**
		 * 执行后置处理器的postProcessorAfterInitialization()方法
		 */
  if (mbd == null || !mbd.isSynthetic()) {
    wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
  }
  /**
		 * 完成各种初始化回调方法，回到调用处：
		 * @see #doCreateBean(String, RootBeanDefinition, Object[]) 
		 */
  return wrappedBean;
}
```

在此方法中就可以看到初始化回调的执行顺序，首先是`applyBeanPostProcessorsBeforeInitialization()`这个方法里面会执行`@PostContruct`注解的方法，是通过后置处理器实现的。

其次是`invokeInitMethods()`方法，这个方法内会执行的`InitializingBean`接口的`afterPropertiesSet()`方法，以及执行在`@Bean`注解中定义的初始化方法。

最后的是`applyBeanPostProcessorsAfterInitialization()`方法，这个方法会执行后置处理器接口`SmartInitializingSingleton`的`afterSingletonsInstantiated()`方法。

到此解决了一个疑问，不同类型的回调的执行顺序就是在上述代码中设定的。



### PostConstruct注解方法的执行

此注解执行的处理类是`org.springframework.beans.factory.annotation.InitDestroyAnnotationBeanPostProcessor#postProcessBeforeInitialization`。

进入此方法。

```java
public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
  // 获取@PostConstruct注解定义的方法
  LifecycleMetadata metadata = findLifecycleMetadata(bean.getClass());
  try {
    metadata.invokeInitMethods(bean, beanName);
  }
  catch (InvocationTargetException ex) {
    throw new BeanCreationException(beanName, "Invocation of init method failed", ex.getTargetException());
  }
  catch (Throwable ex) {
    throw new BeanCreationException(beanName, "Failed to invoke init method", ex);
  }
  return bean;
}
```

这个方法逻辑比较简单，`findLifecycleMetadata()`把使用了注解`@PostConstruct`定义的方法包装成`LifecycleMetadata`，然后在`invokeInitMethods()`方法中执行。这两个方法的最底层都是通过反射实现的。

**回到前面的疑问，`@PostConstruct`注解的方法为何可以有多个？并且无法控制方法的执行顺序？**

第二个疑问，进入`findLifecycleMetadata()`方法的实现就知道了。

```java
private LifecycleMetadata findLifecycleMetadata(Class<?> clazz) {
  if (this.lifecycleMetadataCache == null) {
    // Happens after deserialization, during destruction...
    return buildLifecycleMetadata(clazz);
  }
  // Quick check on the concurrent map first, with minimal locking.
  LifecycleMetadata metadata = this.lifecycleMetadataCache.get(clazz);
  if (metadata == null) {
    synchronized (this.lifecycleMetadataCache) {
      metadata = this.lifecycleMetadataCache.get(clazz);
      if (metadata == null) {
        // 通过反射获取到使用@PostConstruct注解的方法，并包装成LifecycleMetadata
        metadata = buildLifecycleMetadata(clazz);
        this.lifecycleMetadataCache.put(clazz, metadata);
      }
      return metadata;
    }
  }
  return metadata;
}
```

这个方法还不是真正实现的地方，还需要继续进入`buildLifecycleMetadata()`方法。

```java
private LifecycleMetadata buildLifecycleMetadata(final Class<?> clazz) {
  if (!AnnotationUtils.isCandidateClass(clazz, Arrays.asList(this.initAnnotationType, this.destroyAnnotationType))) {
    return this.emptyLifecycleMetadata;
  }

  List<LifecycleElement> initMethods = new ArrayList<>();
  List<LifecycleElement> destroyMethods = new ArrayList<>();
  Class<?> targetClass = clazz;

  do {
    final List<LifecycleElement> currInitMethods = new ArrayList<>();
    final List<LifecycleElement> currDestroyMethods = new ArrayList<>();

    // 通过反射获取到目标类的方法列表，然后遍历这些方法判断是否有使用@PostConstruct注解
    // 如果有则保存到currInitMethods
    ReflectionUtils.doWithLocalMethods(targetClass, method -> {
      if (this.initAnnotationType != null && method.isAnnotationPresent(this.initAnnotationType)) {
        LifecycleElement element = new LifecycleElement(method);
        currInitMethods.add(element);
        if (logger.isTraceEnabled()) {
          logger.trace("Found init method on class [" + clazz.getName() + "]: " + method);
        }
      }
      if (this.destroyAnnotationType != null && method.isAnnotationPresent(this.destroyAnnotationType)) {
        currDestroyMethods.add(new LifecycleElement(method));
        if (logger.isTraceEnabled()) {
          logger.trace("Found destroy method on class [" + clazz.getName() + "]: " + method);
        }
      }
    });

    initMethods.addAll(0, currInitMethods);
    destroyMethods.addAll(currDestroyMethods);
    targetClass = targetClass.getSuperclass();
  }
  while (targetClass != null && targetClass != Object.class);

  return (initMethods.isEmpty() && destroyMethods.isEmpty() ? this.emptyLifecycleMetadata :
          new LifecycleMetadata(clazz, initMethods, destroyMethods));
}
```

在继续进入`doWithLocalMethods()`方法， 这个方法还做了一层封装。核心的方法实现在`getDeclaredMethods()`，这个方法里面最主要一句代码`Method[] declaredMethods = clazz.getDeclaredMethods();`，这行代码只要用过反射的应该很熟悉。直接通过class获取到类内部的所有方法（包括继承过来的方法）。`clazz.getDeclaredMethods()`方法返回的是一个数组，里面的元素没有提供排序的入口。所以最终执行的`@PostConstruct`注解的方法也不知道顺序。

```java
public static void doWithLocalMethods(Class<?> clazz, MethodCallback mc) {
  // 通过反射获取目标类内部的方法
  Method[] methods = getDeclaredMethods(clazz, false);
  for (Method method : methods) {
    try {
      mc.doWith(method);
    }
    catch (IllegalAccessException ex) {
      throw new IllegalStateException("Not allowed to access method '" + method.getName() + "': " + ex);
    }
  }
}
```

拿到所有注解的方法之后，直接通过`for`循环逐个执行。从入口一直到这里执行注解方法的过程中，没有对方法做任何的过滤，直接`for`循环遍历执行。所以同一个类是支持在多个方法上使用`@PostConstruct`注解，并且会把所有使用了注解的方法执行。



### InitializingBean接口执行

`@PostConstruct`注解的方法执行完毕之后，回到`initializeBean()`方法中，继续往下执行`invokeInitMethods()`方法。这个方法中就会执行所有实现了`InitializingBean`接口的实现类。

```java
protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd)
  throws Throwable {

  boolean isInitializingBean = (bean instanceof InitializingBean);
  // 使用@PostConstruct定义的方法在解析bean定义时候会初始化到bean定义属性externallyManagedInitMethods里面
  // 如果用户在afterPropertiesSet()方法上也使用了@PostConstruct注解则不会再执行。
  if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
    if (logger.isTraceEnabled()) {
      logger.trace("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
    }
    if (System.getSecurityManager() != null) {
      try {
        AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
          ((InitializingBean) bean).afterPropertiesSet();
          return null;
        }, getAccessControlContext());
      }
      catch (PrivilegedActionException pae) {
        throw pae.getException();
      }
    }
    else {
      ((InitializingBean) bean).afterPropertiesSet();
    }
  }

  if (mbd != null && bean.getClass() != NullBean.class) {
    // 获取自定义的初始化方法，比如@Bean(initMethod = "beanInit")，自定义的初始化方法就是beanInit()
    String initMethodName = mbd.getInitMethodName();
    if (StringUtils.hasLength(initMethodName) &&
        // 排除自定义的初始化方法也是afterPropertiesSet()方法，避免重复执行
        !(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
        // 使用@PostConstruct定义的方法在解析bean定义时候会初始化到bean定义属性externallyManagedInitMethods里面
        // 方式初始化方法重复执行，如果在前面执行@PostConstruct方法已经执行过同名方法则不再执行
        // 另外，如果@Bean自定义的回调方法也是afterPropertiesSet()方法，
        // 这里不会再次执行，因为在前面InitializingBean接口方法已经执行过
        !mbd.isExternallyManagedInitMethod(initMethodName)) {
      invokeCustomInitMethod(beanName, bean, mbd);
    }
  }
}
```

此方法的逻辑也很明了，分为两部分：前面部分就是判断当前bean是否实现了接口，如果是就直接执行接口的`afterPropertiesSet()`方法。

```java
((InitializingBean) bean).afterPropertiesSet();	
```

但是在执行之前有一段校验逻辑`mbd.isExternallyManagedInitMethod("afterPropertiesSet")`，这一行代码就是用于判断，如果前面使用`@PostConstruct`注解声明的方法中已经包含了`afterPropertiesSet`则不会在执行这个方法。避免重复执行。这也就是为何演示例子中`afterPropertiesSet()`方法只会执行一遍的原因。并且是在执行`@PostConstruct`方法列表的时候就执行了。

这个方法的第二部分也同样有校验。在执行`@Bean`自定义的初始化方法之前也是先判断`afterPropertiesSet()`方法是否已经执行过。避免`@Bean(initMethod = "afterPropertiesSet")`这种情况导致方法重复执行。

`@Bean`中自定义的方法的执行详细逻辑就不贴代码了，最底层也是通过反射执行的。

好了，完成了`@Bean`自定义方法和`InitializingBean`接口方法的执行之后，接着回到`initializeBean()`方法中，继续往下执行`applyBeanPostProcessorsAfterInitialization()`方法，这个方法主要是执行后置处理器的`postProcessAfterInitialization()`方法的。和本篇主题无关，不过多解释。



### afterSingletonsInstantiated()方法执行

最后就是`SmartInitializingSingleton.afterSingletonsInstantiated()`方法的执行，此方法的执行和前面的初始化回调不在一个地方，这个方法是在bean实例化完成，并且完成了属性填充之后。才执行的，代码位置在：`org.springframework.beans.factory.support.DefaultListableBeanFactory#preInstantiateSingletons`。

```java
/**
	 * 实例化所有除了Spring内部的单例（懒加载的、抽象类、非单例的除外）
 	 */
@Override
public void preInstantiateSingletons() throws BeansException {
  // 省略与本篇无关代码
  
  // =========================================================================
  // 到此单例bean都已经实例化完毕，紧接着可以对实例对象做一些增强，通过BeanPostProcessor后置处理器增强

  // Trigger post-initialization callback for all applicable beans...
  // 判断是否有实现了SmartInittializingSingleton的实现类，通常是Spring内部的实现类，也是Spring提供的一个很重要的扩展点
  // 初始化操作执行顺序：@PostConstruct是最先被执行的，然后是InitializingBean，最后是SmartInitializingSingleton
  for (String beanName : beanNames) {
    Object singletonInstance = getSingleton(beanName);
    if (singletonInstance instanceof SmartInitializingSingleton) {
      final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
      // bean实例化后调用bean的afterSingletonsInstantiated方法，用户可以实现SmartInitializingSingleton接口，
      // 在所有bean实例化后做一些自定义的操作，比如重置实例的某些属性，但是要注意只能处理非懒加载的单例bean
      if (System.getSecurityManager() != null) {
        AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
          smartSingleton.afterSingletonsInstantiated();
          return null;
        }, getAccessControlContext());
      }
      else {
        smartSingleton.afterSingletonsInstantiated();
      }
    }
  }
}
```

此处也是一个后置处理器，**这个是Spring框架提供的一个对所有bean初始化的入口**。前面的初始化回调都是针对某一个bean做得处理。这个是最大的不同之处。只要你实现了此接口方法`afterSingletonsInstantiated()`，所有的bean在初始化完成之后执行此方法逻辑。



好了，到此总算是把所有的初始化回调介绍完毕。你是否搞懂了呢？？



