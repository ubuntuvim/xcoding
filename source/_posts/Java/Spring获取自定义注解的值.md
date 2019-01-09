---
title: Spring获取自定义注解的值
tag:
    - Spring
    - Value
    - Aspect
    - Java
---

Spring的`@Value`注解提供了非常强大的功能。可以直接使用SpEL表达式。比如：

```java
@Value("v")

@Value("${prop.key}")

@Value("#{beanId.method('args')}")

```

但是对于自定义的注解是没有这么强大的功能的，那么如何能做到像`@Value`这么强大的功能呢？

#### 自定义注解

```java
package com.ubuntuvim.sb.aop;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.METHOD, ElementType.CONSTRUCTOR})
@Retention(RetentionPolicy.RUNTIME)
public @interface TargetDataSource {
	String value() default "default";
}
```

如上代码，定义了一个用于普通方法和构造方法上面的注解`TargetDataSource`。

自定义的注解如果没有增加解析代码是没有任何效果的。可以借助于Spring AOP帮忙拦截使用了这个注解的所有方法，在执行注解所在方法之前或者之后可以增加自己的处理逻辑。

```java
package com.ubuntuvim.sb.aop;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.beans.factory.config.BeanExpressionContext;
import org.springframework.beans.factory.config.BeanExpressionResolver;
import org.springframework.beans.factory.config.ConfigurableBeanFactory;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.PropertySource;
import org.springframework.context.expression.StandardBeanExpressionResolver;
import org.springframework.stereotype.Component;

import com.ubuntuvim.sb.SpringBoot2Application;

@Aspect
@Component
@PropertySource("classpath:application.properties")
public class TargetDataSourceAspect {
	
	private static final BeanExpressionResolver resolver = new StandardBeanExpressionResolver();

	
	@Before("@annotation(targetDataSource)")
	public void setDataSource(TargetDataSource targetDataSource) {
		System.out.println(targetDataSource.value());
//		SpelParserConfiguration config = new SpelParserConfiguration(true,true);
//		ExpressionParser parser = new SpelExpressionParser(config);
////		ExpressionParser parser = new SpelExpressionParser();
//		Expression exp = parser.parseExpression(targetDataSource.value());
//		Object v = exp.getValue();
//		System.out.println("====== " + v);
		
//		StringValueResolver svr = new StringValueResolver() {
//			@Override
//			public String resolveStringValue(String strVal) {
//				System.out.println(strVal);
//				return strVal;
//			}
//		};
//		String s = svr.resolveStringValue(targetDataSource.value());
//		System.out.println("s = " + s);
		Object s = "";
		String value = targetDataSource.value();
    	AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringBoot2Application.class);
    	ConfigurableBeanFactory beanFactory = context.getBeanFactory();
    	String resolvedValue = beanFactory.resolveEmbeddedValue(value);
		System.out.println("resolvedValue = " + resolvedValue);
		// 字符串类型
		if (!((resolvedValue.startsWith("${") || resolvedValue.startsWith("#{")) && resolvedValue.endsWith("}"))) {
			s = resolvedValue;
		} else {
			s = resolver.evaluate(resolvedValue, new BeanExpressionContext(beanFactory, null));
		}
		System.out.println("s = " + s);
		
		System.out.println("使用了注解【com.ubuntuvim.sb.aop.TargetDataSource】的方法之前，先指定本方法。");
	}
	
}
```

关键是这行代码：`@Before("@annotation(targetDataSource)")`，在方法执行之前会先执行方法`before`。在这个方法里面获取到注解`TargetDataSouce`设置的值`value`。
根据value的值解析。

```java
// 字符串类型
if (!((resolvedValue.startsWith("${") || resolvedValue.startsWith("#{")) && resolvedValue.endsWith("}"))) {
    s = resolvedValue;
} else {
    s = resolver.evaluate(resolvedValue, new BeanExpressionContext(beanFactory, null));
}
```

#### 使用注解

```java
package com.ubuntuvim.sb.aop;

import org.springframework.stereotype.Service;

@Service
public class UserServiceImpl {

	@TargetDataSource("test1")
	public String test1() {
		return "test1 method...";
	}
	
	@TargetDataSource("${ds}")
	public String test2() {
		return "test2 method...";
	}
	
	@TargetDataSource("#{EnvUtil.getValue('1')}")
	public String test3() {
		return "test3 method...";
	}
	
	@TargetDataSource("#{EnvUtil.getValue('2')}")
	public String test4() {
		return "test4 method...";
	}
	
}
```

```java
package com.ubuntuvim.sb.aop;

import org.springframework.stereotype.Component;

@Component("EnvUtil")
public class EnvUtil {

	public static String getValue(String key) {
		if ("1".equals(key))
			return "key1";
		
		if ("2".equals(key))
			return "ds";
		
		if ("3".equals(key))
			return "key"+key;
		
		return "";
	}
	
}
```

这三种是最常见的使用方式，

```java
@TargetDataSource("test1")

@TargetDataSource("${ds}")

@TargetDataSource("#{EnvUtil.getValue('1')}")
```


#### 测试

增加一个测试类验证是否能解析到注解的值。

```java
package com.ubuntuvim.sb.aop;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import com.ubuntuvim.sb.SpringBoot2Application;

import static org.junit.Assert.*;

@RunWith(SpringRunner.class)
@SpringBootTest(classes = SpringBoot2Application.class)
public class TargetDataSourceAspectTest {

	@Autowired
	private UserServiceImpl userServiceImpl;
	
	@Test
	public void testSetDataSource() {
		String s = userServiceImpl.test1();
		assertEquals("test1 method...", s);
		System.out.println(s);
	}
	
	@Test
	public void testSetDataSource2() {
		String s = userServiceImpl.test2();
		assertEquals("test2 method...", s);
		System.out.println(s);
	}
	
	@Test
	public void testSetDataSource3() {
		String s = userServiceImpl.test3();
		assertEquals("test3 method...", s);
		System.out.println(s);
	}
	
	@Test
	public void testSetDataSource4() {
		String s = userServiceImpl.test4();
		assertEquals("test4 method...", s);
		System.out.println(s);
	}

}
```

用Junit运行，可以看到如下日志：

```shell
2019-01-10 00:16:37.686  INFO 9290 --- [           main] c.u.sb.aop.TargetDataSourceAspectTest    : Started TargetDataSourceAspectTest in 6.43 seconds (JVM running for 9.397)
#{EnvUtil.getValue('2')}
resolvedValue = #{EnvUtil.getValue('2')}
s = ds
使用了注解【com.ubuntuvim.sb.aop.TargetDataSource】的方法之前，先指定本方法。
test4 method...
#{EnvUtil.getValue('1')}
resolvedValue = #{EnvUtil.getValue('1')}
s = key1
使用了注解【com.ubuntuvim.sb.aop.TargetDataSource】的方法之前，先指定本方法。
test3 method...
test1
resolvedValue = test1
s = test1
使用了注解【com.ubuntuvim.sb.aop.TargetDataSource】的方法之前，先指定本方法。
test1 method...
${ds}
resolvedValue = xxx
s = xxx
使用了注解【com.ubuntuvim.sb.aop.TargetDataSource】的方法之前，先指定本方法。
test2 method...
2019-01-10 00:16:40.945  INFO 9290 --- [       Thread-5] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'
```


可以看到可以正确解析出来了。