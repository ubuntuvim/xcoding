---
title: FactoryBean接口深入理解
tag:
	- Spring
	- Java
---

Spring提供的FactoryBean接口是一个非常强大而且常用的扩展，可以通过实现接口的`getObject()`方法往Spring容器中注册bean。
但是这个接口有一些特殊，我们通过这个接口的实现类获取到并不是实现类本身而是`getObject()`方法返回的实例。

```java
package com.ubuntuvim.spring.fb;


import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.ToString;

/**
 * @Author: ubuntuvim
 * @Date: 2020/7/26 22:18
 */
@AllArgsConstructor
@NoArgsConstructor
@Data
@ToString
public class Car {
	String type;
	String color;
}
```
定义一个简单的对象，通过`FactoryBean`接口往容器注入不同类型的实例。

```java
package com.ubuntuvim.spring.fb;


import org.springframework.beans.factory.FactoryBean;

/**
 * 通过xml设置不同的Car，并且属性不同
 * @Author: ubuntuvim
 * @Date: 2020/7/26 22:19
 */
public class CarFactoryBean implements FactoryBean<Car> {

	/**
	 * 通过XML配置注入值
	 */
	String color;
	String type;

	@Override
	public Car getObject() throws Exception {
		return new Car(type,color);
	}

	@Override
	public Class<?> getObjectType() {
		return Car.class;
	}

	/**
	 * 返回单例Car
	 * @return
	 */
	@Override
	public boolean isSingleton() {
		return true;
	}

	public String getColor() {
		return color;
	}

	public void setColor(String color) {
		this.color = color;
	}

	public String getType() {
		return type;
	}

	public void setType(String type) {
		this.type = type;
	}
}
```

通过xml定义不同的Car实例。

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xmlns:context="http://www.springframework.org/schema/context"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

	<context:component-scan base-package="com.ubuntuvim.spring.fb" />

	<!--
	FactoryBean实例返回的实际是getObject()方法返回的对象。
	所以并不是拿到一个CarFactoryBean的实例，而是拿到一个Car实例
	-->
	<bean name="bmw" class="com.ubuntuvim.spring.fb.CarFactoryBean">
		<property name="color" value="black" />
		<property name="type" value="bmw" />
	</bean>

	<bean name="benz" class="com.ubuntuvim.spring.fb.CarFactoryBean">
		<property name="color" value="black" />
		<property name="type" value="benz" />
	</bean>

	<bean name="testRefrenceBean" class="com.ubuntuvim.spring.fb.RefrenceBean" />

</beans>
```

在另外一个类中通过`@Resource`注入到属性上。
```java
package com.ubuntuvim.spring.fb;


import javax.annotation.Resource;

import org.springframework.stereotype.Component;

/**
 * @Author: ubuntuvim
 * @Date: 2020/7/26 22:28
 */
@Component
public class MyCar {
	/**
	 * 注入的是一个Car实例，而不是CarFactoryBean实例
	 */
	@Resource(name = "bmw")
	Car bmw;

	public void myCar() {
		System.out.println(bmw instanceof Car);
		System.out.println(bmw);
	}
}
```

验证获取到的类型是否Car还是CarFactoryBean的对象。

```java
package com.ubuntuvim.spring.fb;


import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @Author: ubuntuvim
 * @Date: 2020/7/26 22:24
 */
public class CarTest {
	public static void main(String[] args) {
		ApplicationContext ac = new ClassPathXmlApplicationContext("application.xml");
		Car bmw = (Car) ac.getBean("bmw");
		System.out.println(bmw);

		Car benz = (Car) ac.getBean("benz");
		System.out.println(benz);

		// 验证注入的对象是什么？
		MyCar myCar = ac.getBean(MyCar.class);
		myCar.myCar();
	}
}
```

运行结果：
```
Car(type=bmw, color=black)
Car(type=benz, color=black)
true
Car(type=bmw, color=black)
```
