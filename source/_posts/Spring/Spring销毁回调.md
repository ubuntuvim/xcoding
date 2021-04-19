---
title: Spring销毁回调方法
tag:
	- Spring
---


接着前篇，有初始化对应着就有销毁。Spring提供了多种方式的销毁回调方法，这些方法在手动关闭容器的时候就会触发。

销毁回调方式：

1. 后置处理器`DestructionAwareBeanPostProcessor`的`postProcessBeforeDestruction()`方法，此方式是对所有bean有效
2. 在类方法上使用`@PreDestroy`注解
3. 实现接口`DisposableBean`的`destory()`方法
4. 自定义回调方法，`@Bean(destroyMethod = "beanDestoryCallbackMethod")`
5. 实现`AutoCloseable`接口或者`Closeable`接口的bean（当且仅当此bean没有任何自定义销毁回调，也就是说此bean没有使用前面的2，3，4这三种方式）


仍然按照前篇的思路，先演示，再提疑问，再读源码。



### 演示案例

自定义一个实现前面四种方式中的后三种方式的bean，实现`DisposableBean`接口，在bean中使用`@PreDestory`注解。然后在注入容器的时候使用`@Bean`注解。

```java
package com.ubuntuvim.spring.createbean;

import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.support.DefaultSingletonBeanRegistry;
import org.springframework.context.support.AbstractApplicationContext;

import javax.annotation.PreDestroy;
import java.io.IOException;

/**
 * 和InitCallbackBean是相互呼应的，有销毁回调对应着就有销毁的回调。
 * 验证各种销毁回调方法的执行顺序：
 * 1. 执行销毁回调DestructionAwareBeanPostProcessor.postProcessBeforeDestruction()方法
 * 2. 执行销毁回调@PreDestroy注解的回调方法twoDestroy()
 * 3. 执行销毁回调@PreDestroy注解的回调方法oneDestroy()
 * 4. 执行销毁回调DisposableBean.destory()方法
 * 5. 执行销毁回调@Bean(destroyMethod = "beanDestoryCallbackMethod")的方法
 *
 * 需要注意的是，这些销毁的回调方法需要手动调用容器的关闭方法才会触发。触发方法如下：
 * @see AbstractApplicationContext#close()
 * 最终会调用到这个类的方法处理上述回调接口。
 * @see DisposableBeanAdapter#destroy()
 *
 * @Author: ubuntuvim
 * @Date: 2020/10/18 下午5:17
 */
public class DestoryCallbackBean implements DisposableBean, /*Closeable, */AutoCloseable {

	// 实现DisposableBean接口的方法
	@Override
	public void destroy() throws Exception {
		System.out.println("执行销毁回调DisposableBean.destory()方法");
	}

	// 自定义销毁方法：@Bean(destroyMethod = "beanDestoryCallbackMethod")
	private void beanDestoryCallbackMethod() {
		System.out.println("执行销毁回调@Bean(destroyMethod = \"beanDestoryCallbackMethod\")的方法");
	}
	// 如果有一个同名销毁方法会不会被读取使用？？不会，因为@Bean只支持无参方法
	private void beanDestoryCallbackMethod(Fruit apple) {
		apple.eatable();
		System.out.println("执行销毁回调@Bean(destroyMethod = \"beanDestoryCallbackMethod\")的方法");
	}

	/**
	 * 如果一个类中有两个方法都使用了@PreDestory注解会怎么执行？
	 * 都会执行，执行代码：
	 * @see DefaultSingletonBeanRegistry#destroySingletons()
	 * 但是如果DispoableBean的destory()方法也使用了@PreDestory注解则只会执行一次。
	 * 这个和初始化的InitializingBean.afterPropertiesSet()方法类似，只会执行一次。
	 */
	@PreDestroy
	public void twoDestroy() {
		System.out.println("执行销毁回调@PreDestroy注解的回调方法twoDestroy()");
	}
	@PreDestroy
	public void oneDestroy() {
		System.out.println("执行销毁回调@PreDestroy注解的回调方法oneDestroy()");
	}

	/**
	 * 实现接口AutoCloseable的方法。
	 * bean没有实现任何销毁回调，也没有在@Bean中自定义任何销毁方法下，当容器执行close事件时此方法会被执行。
	 * 比如SimpleDestoryBean类的例子。
	 * @throws IOException
	 */
	@Override
	public void close() throws IOException {
		System.out.println("执行销毁回调Closeable.close()方法");
	}
	
	public void shutdown() {
		System.out.println("执行销毁回调shutdown()方法");
	}

}
```

还有一种默认的回调，就是实现`AutoCloseable`接口或者`Closeable`接口。

```java
package com.ubuntuvim.spring.createbean;

import org.springframework.stereotype.Component;

import java.io.Closeable;

/**
 * 当前bean既不实现DispoableBean接口，也没有自定义的销毁方法
 * 但是有一个AutoCloseable.close()方法，验证容器关闭时是否会回调AutoCloseable.close()方法。
 * 实现Closeable接口也是同样的效果。
 * @Author: ubuntuvim
 * @Date: 2020/10/18 下午10:38
 */
@Component
public class SimpleDestoryBean implements Closeable {

	@Override
	public void close() {
		System.out.println("既不实现DispoableBean接口，也没有自定义的销毁方法的情况下执行最后默认的AutoCloseable.close方法");
	}
}
```



再定义一个后置处理器，实现`DestructionAwareBeanPostProcessor`接口。

```java
/**
 * 自定义销毁回调的后置处理器
 * @Author: ubuntuvim
 * @Date: 2020/10/18 下午5:23
 */
@Component
public class MyDestructionAwareBeanPostProcessorImpl implements DestructionAwareBeanPostProcessor {
	@Override
	public void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException {
		if ("DestoryCallbackBean".equalsIgnoreCase(beanName))
			System.out.println("执行销毁回调DestructionAwareBeanPostProcessor.postProcessBeforeDestruction()方法");
	}

	@Override
	public boolean requiresDestruction(Object bean) {
		// 返回true，表示需要执行销毁回调方法postProcessBeforeDestruction()
		return true;
	}
}
```

最后，通过`@Bean`注入到容器中。

```java
package com.ubuntuvim.spring.createbean;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
 * @Author: ubuntuvim
 * @Date: 2020/10/17 下午4:46
 */
@ComponentScan
@Configuration
public class Config {
	@Bean(destroyMethod = "beanDestoryCallbackMethod")
	public DestoryCallbackBean destoryCallbackBean() {
		return new DestoryCallbackBean();
	}
}
```

运行案例;

```java
package com.ubuntuvim.spring.createbean;

import org.springframework.beans.factory.support.RootBeanDefinition;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.AbstractApplicationContext;

/**
 * @Author: ubuntuvim
 * @Date: 2020/10/17 下午4:48
 */
public class CreateBeanTest {
	public static void main(String[] args) {
		AbstractApplicationContext applicationContext = new AnnotationConfigApplicationContext(Config.class);

		// 执行各种销毁回调
		applicationContext.close();
	}
}
```

**`applicationContext.close();`这句代码非常关键，必须要手动关闭容器才会触发销毁回调的执行**。你可以把这行代码注释掉在运行，注释掉之后不会打印如下的日志。即使是后置处理器接口也不会执行。

```shell
执行销毁回调DestructionAwareBeanPostProcessor.postProcessBeforeDestruction()方法
执行销毁回调@PreDestroy注解的回调方法oneDestroy()
执行销毁回调@PreDestroy注解的回调方法twoDestroy()
执行销毁回调DisposableBean.destory()方法
执行销毁回调@Bean(destroyMethod = "beanDestoryCallbackMethod")的方法
既不实现DispoableBean接口，也没有自定义的销毁方法的情况下执行最后默认的AutoCloseable.close方法
```



### Spring源码



接下来，我们从Spring源码入手，看看这些回调是在什么地方执行的。首先在`main`方法中的`close`方法上打个断点，然后用debug方式运行`main`方法。

进入到`close()`方法的逻辑里面。

```java
@Override
public void close() {
  synchronized (this.startupShutdownMonitor) {
    doClose();
    // If we registered a JVM shutdown hook, we don't need it anymore now:
    // We've already explicitly closed the context.
    if (this.shutdownHook != null) {
      try {
        Runtime.getRuntime().removeShutdownHook(this.shutdownHook);
      }
      catch (IllegalStateException ex) {
        // ignore - VM is already shutting down
      }
    }
  }
}
```

按照Spring框架的尿性，不用想，具体处理逻辑肯定是在`doClose()`里面。

```java
protected void doClose() {
  // Check whether an actual close attempt is necessary...
  if (this.active.get() && this.closed.compareAndSet(false, true)) {
    if (logger.isDebugEnabled()) {
      logger.debug("Closing " + this);
    }

    LiveBeansView.unregisterApplicationContext(this);

    try {
      // Publish shutdown event.
      // 发布容器关闭事件，任何监听ContextClosedEvent事件的监听器都可以收到一条消息
      publishEvent(new ContextClosedEvent(this));
    }
    catch (Throwable ex) {
      logger.warn("Exception thrown from ApplicationListener handling ContextClosedEvent", ex);
    }

    // Stop all Lifecycle beans, to avoid delays during individual destruction.
    if (this.lifecycleProcessor != null) {
      try {
        this.lifecycleProcessor.onClose();
      }
      catch (Throwable ex) {
        logger.warn("Exception thrown from LifecycleProcessor on context close", ex);
      }
    }

    // Destroy all cached singletons in the context's BeanFactory.
    // 这个方法里面会执行自定义的销毁方法（@Bean），实现DisposableBean接口的方法，
    // 以及InitDestroyAnnotationBeanPostProcessor#postProcessBeforeDestruction方法
    destroyBeans();

    // Close the state of this context itself.
    // 设置容器上下文状态，
    closeBeanFactory();

    // Let subclasses do some final clean-up if they wish...
    // 留给子类实现的自定义销毁逻辑
    onClose();

    // Reset local application listeners to pre-refresh state.
    // 清理事件监听
    if (this.earlyApplicationListeners != null) {
      this.applicationListeners.clear();
      this.applicationListeners.addAll(this.earlyApplicationListeners);
    }

    // Switch to inactive.
    // 设置容器状态为关闭
    this.active.set(false);
  }
}
```

这个方法做的事情很多，主要有：触发所有bean的销毁回调、发布关闭事件、执行bean的生命周期关闭方法、设置容器的状态为关闭、执行销毁的后置处理器。

本篇重点关注的是`destroyBeans()`方法。这个方法里面会执行自定义的销毁方法，`@PreDestory`注释的方法，后置处理器的`postProcessBeforeDestruction()`方法。

进入`destoryBeans()`方法后，发现这还不是最后的执行逻辑，这个方法又交给`DefaultListableBeanFactory`的`destroySingletons()`方法处理。

进入`destorySingletons()`方法后，发现还不是最好的执行逻辑，它的实现是调用父类的同名方法，继续进入它父类同名方法。`DefaultSingletonBeanRegistry`的`destroySingletons()`。

```java
// DefaultSingletonBeanRegistry的destroySingletons()方法
public void destroySingletons() {
  if (logger.isTraceEnabled()) {
    logger.trace("Destroying singletons in " + this);
  }
  synchronized (this.singletonObjects) {
    this.singletonsCurrentlyInDestruction = true;
  }

  String[] disposableBeanNames;
  synchronized (this.disposableBeans) {
    disposableBeanNames = StringUtils.toStringArray(this.disposableBeans.keySet());
  }
  // 遍历所有bean，逐个执行这些bean的销毁方法
  for (int i = disposableBeanNames.length - 1; i >= 0; i--) {
    destroySingleton(disposableBeanNames[i]);
  }

  this.containedBeanMap.clear();
  this.dependentBeanMap.clear();
  this.dependenciesForBeanMap.clear();

  clearSingletonCache();
}
```

最核心的逻辑在for循环里面，遍历所有可销毁的bean数组`disposableBeanNames`。

继续一步步进入调用逻辑，按照如下调用顺序就可以找到最后的执行逻辑：

> DefaultSingletonBeanRegistry.destroySingletons() -> DefaultSingletonBeanRegistry.destroySingleton() -> DisposableBeanAdapter.destroy()

DisposableBeanAdapter.destory()才是最后的处理逻辑。不得不说这个调用层次实在是太深了！！

```java
public void destroy() {
  if (!CollectionUtils.isEmpty(this.beanPostProcessors)) {
    for (DestructionAwareBeanPostProcessor processor : this.beanPostProcessors) {
      /**
				 * 执行后置处理器的销毁回调方法
				 * @PreDestroy注释的方法就是在这里执行的。默认实现有
				 * @see InitDestroyAnnotationBeanPostProcessor#postProcessBeforeDestruction(Object, String)
				 */
      processor.postProcessBeforeDestruction(this.bean, this.beanName);
    }
  }

  if (this.invokeDisposableBean) {
    if (logger.isTraceEnabled()) {
      logger.trace("Invoking destroy() on bean with name '" + this.beanName + "'");
    }
    // 执行DisposableBean接口的destory()方法。
    try {
      if (System.getSecurityManager() != null) {
        AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
          ((DisposableBean) this.bean).destroy();
          return null;
        }, this.acc);
      }
      else {
        ((DisposableBean) this.bean).destroy();
      }
    }
    catch (Throwable ex) {
      String msg = "Invocation of destroy method failed on bean with name '" + this.beanName + "'";
      if (logger.isDebugEnabled()) {
        logger.warn(msg, ex);
      }
      else {
        logger.warn(msg + ": " + ex);
      }
    }
  }
  // 1. 执行自定义的销毁方法，比如下面自定义的销毁方法就是beanDestoryCallbackMethod()
	// 		@Bean(destroyMethod = "beanDestoryCallbackMethod")
	// 2. 执行实现Closable接口的close()方法。
  if (this.destroyMethod != null) {
    invokeCustomDestroyMethod(this.destroyMethod);
  }
  else if (this.destroyMethodName != null) {
    Method methodToInvoke = determineDestroyMethod(this.destroyMethodName);
    if (methodToInvoke != null) {
      invokeCustomDestroyMethod(ClassUtils.getInterfaceMethodIfPossible(methodToInvoke));
    }
  }
}
```

1. 首先遍历后置处理器的`postProcessBeforeDestruction()`方法，这个方法是对所有bean生效的。

2. `@PreDestory`注解标注的方法也是通过后置处理器执行的。当遍历到`InitDestroyAnnotationBeanPostProcessor`这个后置处理器的时候，执行的原理和初始化回调是一样的，底层都是通过反射实现。

3. 然后是执行单个bean实现的`DisposableBean`接口的`destory()`方法，

4. 最后是执行在`@Bean`中自定义的销毁回调方法。
5. 执行接口`Closable`接口的`close()`方法。



到此，已经把所有的销毁回调的执行都介绍完了。你是否看懂了呢？？
