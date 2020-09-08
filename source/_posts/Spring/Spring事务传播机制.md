---
title: Spring事务传播机制
tag:
	- Spring
	- Transactional
---



Spring事务传播设置有如下类型：

```java
	/**
	 * 保存user，id是唯一的，否则insert会报错
	 * REQUIRED ：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。==> 会跟随service层方法回滚事务
	 * SUPPORTS ：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。==> 会跟随service层方法回滚事务
	 * MANDATORY ：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。==> 会跟随service层方法回滚事务
	 * REQUIRES_NEW ：创建一个新的事务，如果当前存在事务，则把当前事务挂起。 ==> 即使service层的方法也加了事务，也无法回滚dao层的事务（内层已经提交了，外层回滚个屁啊）。
	 * NOT_SUPPORTED ：以非事务方式运行，如果当前存在事务，则把当前事务挂起。 ==>  即使service层的方法也加了事务，也无法回滚dao层的事务（内层已经提交了，外层回滚个屁啊）。
	 * NEVER ：以非事务方式运行，如果当前存在事务，则抛出异常。 ==> service层无事务的方法可以运行，也就是说，他只能被一个父事务调用。否则，他就要抛出异常。
	 * NESTED ：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于 REQUIRED
	 */
```

上述只是单事务方法的情况，如果存在事务嵌套呢？？比如`serviceA.methodA()`内部调用了`serviceB.methodB()`方法，`methodA()`和`methodB()`都是有事务的。这种情况下可以根据如下表格对照：

| **PROPAGATION TYPE**          | **DESCRIPTION**                                              |
| ----------------------------- | ------------------------------------------------------------ |
| **PROPAGATION_REQUIRED**      | 支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。 如果正要执行的事务不在另外一个事务里，那么就起一个新的事务; 比如说，`ServiceB.methodB`的事务级别定义为PROPAGATION_`REQUIRED`, 那么由于执行`ServiceA.methodA`的时候， `ServiceA.methodA`已经起了事务，这时调用`ServiceB.methodB`，`ServiceB.methodB`看到自己已经运行在`ServiceA.methodA` 的事务内部，就不再起新的事务。而假如`ServiceA.methodA`运行的时`候ServiceB.method`B发现自己没有在事务中，他就会为自己新建一个事务。 这样，在`ServiceA.methodA`或者在`ServiceB.methodB`内的任何地方出现异常，事务都会被回滚。即使`ServiceB.methodB`的事务已经被提交，但是`ServiceA.methodA`在接下来fail要回滚，`ServiceB.methodB`也要回滚。 |
| **PROPAGATION_SUPPORTS**      | 支持当前事务，如果当前没有事务，就以非事务方式执行。 如果当前在事务中，即以事务的形式运行，如果当前不在一个事务中，那么就以非事务的形式运行。 |
| **PROPAGATION_MANDATORY**     | 支持当前事务，如果当前没有事务，就抛出异常。 必须在一个事务中运行，也就是说，他只能被一个父事务调用。否则，他就要抛出异常。 |
| **PROPAGATION_REQUIRES_NEW**  | 新建事务，如果当前存在事务，把当前事务挂起。 比如我们设计`ServiceA.methodA`的事务级别为PROPAGATION_`REQUIRED`，`ServiceB.methodB`的事务级别为PROPAGATION_`REQUIRES_NEW`，那么当执行到`ServiceB.methodB`的时候，`ServiceA.methodA`所在的事务就会挂起，`ServiceB.methodB`会起一个新的事务，等待`ServiceB.methodB`的事务完成以后，他才继续执行。他与PROPAGATION_`REQUIRED `的事务区别在于事务的回滚程度了。因为`ServiceB.methodB`是新起一个事务，那么就是存在两个不同的事务。如果`ServiceB.methodB`已经提交，那么`ServiceA.methodA`失败回滚，`ServiceB.methodB`是不会回滚的。如果`ServiceB.methodB`失败回滚，如果他抛出的异常被`ServiceA.methodA`捕获，`ServiceA.methodA`事务仍然可能提交。 |
| **PROPAGATION_NOT_SUPPORTED** | 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。 不支持当前事务。比如`ServiceA.methodA`的事务级别是PROPAGATION_`REQUIRED`，而`ServiceB.methodB`的事务级别是PROPAGATION_`NOT_SUPPORTED`，那么当执行到`ServiceB.methodB`时，`ServiceA.methodA`的事务挂起，而`ServiceB.methodB`则以非事务的方式运行完，再继续`ServiceA.methodA`的事务。 |
| **PROPAGATION_NEVER**         | 以非事务方式执行，如果当前存在事务，则抛出异常。 不能在事务中运行。假设`ServiceA.methodA`的事务级别是PROPAGATION_`REQUIRED`，而ServiceB.methodB的事务级别是PROPAGATION_`NEVER`， 那么`ServiceB.methodB`就要抛出异常了。 |
| **PROPAGATION_NESTED**        | 支持当前事务，新增Savepoint点，与当前事务同步提交或回滚。 理解Nested的关键是savepoint。他与PROPAGATION_`REQUIRES_NEW`的区别是，PROPAGATION_`REQUIRES_NEW`另起一个事务，将会与他的父事务相互独立，而Nested的事务和他的父事务是相依的，他的提交是要等和他的父事务一块提交的。也就是说，如果父事务最后回滚，他也要回滚的。 而`Nested`事务的好处也是他有一个savepoint。 |



简单示例：

```java
package com.ubuntuvim.mybatis.service;


import java.util.Random;
import java.util.UUID;

import javax.annotation.Resource;

import com.ubuntuvim.mybatis.dao.UserMapper;
import com.ubuntuvim.mybatis.entity.User;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * @Author: ubuntuvim
 * @Date: 2020/9/8 下午10:18
 */
@Service
public class UserServiceImpl implements UserService {

	@Resource
	UserMapper userMapper;

	@Override
	public void saveOneNoTransition() {
		userMapper.save(new User(UUID.randomUUID().toString(), "王五service无事务"+new Random().nextInt(10), "M"));
	}

	@Transactional(rollbackFor = Exception.class)
	@Override
	public void saveOneHasTransition() {
		userMapper.save(new User(UUID.randomUUID().toString(), "ubuntuvim，service有事务"+new Random().nextInt(10), "M"));
	}

	@Override
	public void saveUsersNoTransition() {
		String id = UUID.randomUUID().toString();
		userMapper.save(new User(UUID.randomUUID().toString(), "张三service无事务", "M"));
		userMapper.save(new User(id, "张三2service无事务", "F"));
		userMapper.save(new User(UUID.randomUUID().toString(), "张三3service无事务", "F"));
		userMapper.save(new User(UUID.randomUUID().toString(), "张三4service无事务", "M"));
		// 这个save的id和第二个相同会导致数据库id主键冲突报错，因为当前方法没有加事务不会回滚前面的save持久化
		userMapper.save(new User(id, "张三5service无事务", "M"));
		userMapper.save(new User(UUID.randomUUID().toString(), "张三6service无事务", "F"));
		userMapper.save(new User(UUID.randomUUID().toString(), "张三7service无事务", "M"));
	}

	/**
	 * 由于此方法上添加了事务，即使在调用的userMapper.save()方法上也添加事务也会回滚。
	 * 但是如果dao的save方法的事务传播属性声明为propagation = Propagation.REQUIRES_NEW则无法回滚，在dao层已经提交了事务。service层回滚不了了。
	 *
	 */
	@Transactional(rollbackFor = Exception.class)
	@Override
	public void saveUsersHasTransition() {

		String id = UUID.randomUUID().toString();
		userMapper.save(new User(UUID.randomUUID().toString(), "李四service有事务", "F"));
		userMapper.save(new User(id, "李四2service有事务", "F"));
		userMapper.save(new User(UUID.randomUUID().toString(), "李四3service有事务", "F"));
		userMapper.save(new User(UUID.randomUUID().toString(), "李四4service有事务", "M"));
		/*
		 @Transactional(rollbackFor = Exception.class)
		 这个save的id和第二个相同会导致数据库id主键冲突报错，但是当前方法加了事务会从异常这个回滚前面的save
		 最终结果是这个方法内的save都不会持久化
		 */
		userMapper.save(new User(id, "张三5service有事务", "M"));
		userMapper.save(new User(UUID.randomUUID().toString(), "李四6service有事务", "M"));
		userMapper.save(new User(UUID.randomUUID().toString(), "李四7service有事务", "F"));

	}
}
```

service调用dao的方法。

```java
package com.ubuntuvim.mybatis.dao;

import com.ubuntuvim.mybatis.entity.User;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;

import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

/**
 * @Author: ubuntuvim
 * @Date: 2020/8/2 20:54
 */
public interface UserMapper {

	@Select("select * from user t where t.name = #{name}")
	User getUser(User user);

	/**
	 * 保存user，id是唯一的，否则insert会报错
	 * REQUIRED ：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。==> 会跟随service层方法回滚事务
	 * SUPPORTS ：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。==> 会跟随service层方法回滚事务
	 * MANDATORY ：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。==> 会跟随service层方法回滚事务
	 * REQUIRES_NEW ：创建一个新的事务，如果当前存在事务，则把当前事务挂起。 ==> 即使service层的方法也加了事务，也无法回滚dao层的事务（内层已经提交了，外层回滚个屁啊）。
	 * NOT_SUPPORTED ：以非事务方式运行，如果当前存在事务，则把当前事务挂起。 ==>  即使service层的方法也加了事务，也无法回滚dao层的事务（内层已经提交了，外层回滚个屁啊）。
	 * NEVER ：以非事务方式运行，如果当前存在事务，则抛出异常。 ==> service层无事务的方法可以运行，也就是说，他只能被一个父事务调用。否则，他就要抛出异常。
	 * NESTED ：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于 REQUIRED
	 * @param user
	 */
	@Transactional(rollbackFor = Exception.class, propagation = Propagation.NESTED)
	@Insert("insert into user(id, name, sex) value (#{id}, #{name}, #{sex})")
	void save(User user);
}
```

测试类：

```java
package com.ubuntuvim.mybatis.service;

import javax.annotation.Resource;

import com.ubuntuvim.mybatis.MybatisTest;
import org.junit.Test;
import org.junit.runner.RunWith;

import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

/** 
* UserServiceImpl Tester. 
* 
* @author <Authors name> 
* @since <pre>9月 8, 2020</pre> 
* @version 1.0 
*/
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = MybatisTest.class)
public class UserServiceImplTest { 

    @Resource
	UserService userServiceImpl;

	@Test
	public void testSaveUsersNoTransition() throws Exception {
		userServiceImpl.saveUsersNoTransition();
	}

	@Test
	public void testSaveUsersHasTransition() throws Exception {
		userServiceImpl.saveUsersHasTransition();
	}

	@Test
	public void testSaveOneNoTransation() {
		userServiceImpl.saveOneNoTransition();
	}

	@Test
	public void testSaveOneHasTransation() {
		userServiceImpl.saveOneHasTransition();
	}
}
```

