---
title: Java线程池ThreadPoolExecutor你真的搞懂了吗
tag:	
	- Java
	- 多线程
---

线程池是一个在高并场景非常常用的技术。但是其中的奥秘你是否有真的了解过。比如线程池中的默认线程数和最大线程数是什么关系？缓存对象又是如何使用的？



通过一个简单的示例把线程池整明白了。自定义一个线程池，并且设置一个有界的缓冲队列；

```java
package com.ubuntuvim.spring.thread;

import java.util.concurrent.*;

public class MyThreadPoolExecutor {
	public static void main(String[] args) {
		// 线程池维护线程的最少数量
		int corePoolSize = 2;
		// 线程池维护线程的最大数量
		int maximumPoolSize = 4;
		// 线程池维护线程所允许的空闲时间的单位
		// TimeUnit.SECONDS
		// 线程池维护所允许的空闲时间
		long keepAliveTime = 10;
		// 线程池所使用的缓存队列，当任务达到最大线程数时会把任务放在缓冲队列中，然后有空的线程会逐个执行里面的任务
		int workQueueSize = 47;
		/**
		 * 处理任务的优先级为：核心线程corePoolSize、任务队列workQueue、最大线程maximumPoolSize，如果三者都满了
		 * 使用handler处理被拒绝的任务
		 * corePoolSize -> workQueueSize -> maximumPoolSize
		 *  一开始启动，线程池有两个线程可以接收并处理任务，但是任务比较耗时一下子就来了很多任务，导致2个线程无法满足，
		 *  然后会把任务放在缓冲队列中，但是如果任务量还在继续增加，并且两个线程消化任务比新增任务还慢。
		 *  如果corePoolSize <= maximumPoolSize就会创建新的线程接收任务，创建的最大数量就是maximumPoolSize。
		 *  于是就有了4个线程在接收任务（默认的两个+新创建的2个）。
		 * 此时基本已经到达了临界点，4个线程在连续不断的处理任务。
		 * 如果任务量太大，并且每个任务都处理很耗时（处理完一个就从缓冲队列拿走一个，队列元素数量也就减1）。导致缓冲队列也满了。
		 * 这种情况下就会抛出拒绝任务异常java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution（这个是默认的拒绝异常）
		 *
		 * 验证：
		 * 1. 缓冲队列设置比较小，比如workQueueSize=5时，很容易就出现任务拒绝异常
		 * 2. 如果直接设置缓冲队列workQueueSize=50，肯定不会有问题。所有任务都可以放到缓冲队列中，等待线程池中的线程来处理即可。
		 */
		// 定义一个线程池
		ThreadPoolExecutor executor = new ThreadPoolExecutor(
				corePoolSize,
				maximumPoolSize,
				keepAliveTime,
				TimeUnit.SECONDS,
				// ArrayBlockingQueue是有界的缓冲队列
				new ArrayBlockingQueue<>(workQueueSize));

		for (int i = 1; i <= 50; i++) {
			try {
				System.out.println("i = " + i);
				int task = i;
				executor.execute(() -> {
					System.out.println(Thread.currentThread().getName() + "在执行任务：\t" + task);
					try {
						Thread.sleep(10);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				});
				System.out.println("getActiveCount = " + executor.getActiveCount());
				System.out.println("getTaskCount = " + executor.getTaskCount());
				System.out.println("getCompletedTaskCount = " + executor.getCompletedTaskCount());
				System.out.println("getMaximumPoolSize = " + executor.getMaximumPoolSize());
				System.out.println("workQueueSize = " + executor.getQueue().size());
			} catch (Exception e) {
				System.err.println("任务队列已经满，任务被拒绝：" + e.getMessage());
			}
		}

		executor.shutdown();
	}
}
```

核心参数：
`corePoolSize`，线程池默认存活的线程数，或者说是最小的线程数
`maximumPoolSize`，线程池中可以创建的最大线程数，或者说是线程池中线程数上限
`workQueueSize`，缓冲队列上限

**处理任务的优先级为：核心线程corePoolSize处理任务、任务队列workQueue、最大线程maximumPoolSize，如果三者都满了使用handler处理被拒绝的任务。**

 一开始启动，线程池有两个线程可以接收并处理任务，但是任务比较耗时一下子就来了很多任务，导致2个线程无法满足，然后会把任务放在缓冲队列中，但是如果任务量还在继续增加，并且两个线程消化任务比新增任务还慢。如果 `corePoolSize <= maximumPoolSize`就会创建新的线程接收任务，创建的最大数量就是`maximumPoolSize`。于是就有了4个线程在接收任务（默认的两个+新创建的2个）。
此时基本已经到达了临界点，4个线程在连续不断的处理任务。
如果任务量太大，并且每个任务都处理很耗时（处理完一个就从缓冲队列拿走一个，队列元素数量也就减1）。导致缓冲队列也满了。
这种情况下就会抛出拒绝任务异常`java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution`（这个是默认的拒绝异常）

### 验证

1. 缓冲队列设置比较小，比如workQueueSize=5时，很容易就出现任务拒绝异常
2. 如果直接设置缓冲队列workQueueSize=50，肯定不会有问题。所有任务都可以放到缓冲队列中，等待线程池中的线程来处理即可。
  



workQueueSize=5的运行结果：

```shell
i = 1
pool-1-thread-1在执行任务：	1
getActiveCount = 1
getTaskCount = 1
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 0
i = 2
getActiveCount = 2
getTaskCount = 2
getCompletedTaskCount = 0
pool-1-thread-2在执行任务：	2
getMaximumPoolSize = 4
workQueueSize = 0
i = 3
getActiveCount = 2
getTaskCount = 3
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 1
i = 4
getActiveCount = 2
getTaskCount = 4
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 2
i = 5
getActiveCount = 2
getTaskCount = 5
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 3
i = 6
getActiveCount = 2
getTaskCount = 6
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 4
i = 7
getActiveCount = 2
getTaskCount = 7
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 5
i = 8
getActiveCount = 3
getTaskCount = 8
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 5
i = 9
pool-1-thread-3在执行任务：	8
pool-1-thread-4在执行任务：	9
getActiveCount = 4
getTaskCount = 9
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 5
i = 10
i = 11
i = 12
i = 13
i = 14
i = 15
i = 16
i = 17
i = 18
i = 19
i = 20
pool-1-thread-1在执行任务：	3
i = 21
getActiveCount = 4
getTaskCount = 10
getCompletedTaskCount = 1
getMaximumPoolSize = 4
workQueueSize = 5
i = 22
i = 23
pool-1-thread-2在执行任务：	4
i = 24
getActiveCount = 4
getTaskCount = 11
getCompletedTaskCount = 2
getMaximumPoolSize = 4
workQueueSize = 5
i = 25
i = 26
i = 27
i = 28
i = 29
i = 30
i = 31
i = 32
i = 33
pool-1-thread-4在执行任务：	5
pool-1-thread-3在执行任务：	6
i = 34
getActiveCount = 4
getTaskCount = 12
getCompletedTaskCount = 4
getMaximumPoolSize = 4
workQueueSize = 4
i = 35
getActiveCount = 4
getTaskCount = 13
getCompletedTaskCount = 4
getMaximumPoolSize = 4
workQueueSize = 5
i = 36
i = 37
i = 38
i = 39
i = 40
i = 41
i = 42
i = 43
pool-1-thread-1在执行任务：	7
getActiveCount = 4
pool-1-thread-2在执行任务：	21
getTaskCount = 14
getCompletedTaskCount = 6
getMaximumPoolSize = 4
workQueueSize = 4
i = 44
getActiveCount = 4
getTaskCount = 15
getCompletedTaskCount = 6
getMaximumPoolSize = 4
workQueueSize = 5
i = 45
pool-1-thread-4在执行任务：	24
pool-1-thread-3在执行任务：	34
i = 46
getActiveCount = 4
getTaskCount = 16
getCompletedTaskCount = 8
getMaximumPoolSize = 4
workQueueSize = 4
i = 47
getActiveCount = 4
getTaskCount = 17
getCompletedTaskCount = 8
getMaximumPoolSize = 4
workQueueSize = 5
i = 48
pool-1-thread-1在执行任务：	35
i = 49
getActiveCount = 4
getTaskCount = 18
getCompletedTaskCount = 9
getMaximumPoolSize = 4
workQueueSize = 5
i = 50
pool-1-thread-2在执行任务：	43
pool-1-thread-4在执行任务：	44
pool-1-thread-3在执行任务：	46
pool-1-thread-1在执行任务：	47
pool-1-thread-2在执行任务：	49

BUILD SUCCESSFUL in 5s
35 actionable tasks: 2 executed, 33 up-to-date
任务队列已经满，任务被拒绝：Task com.ubuntuvim.spring.thread.MyThreadPoolExecutor$$Lambda$1/142257191@7229724f rejected from java.util.concurrent.ThreadPoolExecutor@4c873330[Running, pool size = 4, active threads = 4, queued tasks = 5, completed tasks = 0]
任务队列已经满，任务被拒绝：Task com.ubuntuvim.spring.thread.MyThreadPoolExecutor$$Lambda$1/142257191@119d7047 rejected from java.util.concurrent.ThreadPoolExecutor@4c873330[Running, pool size = 4, active threads = 4, queued tasks = 5, completed tasks = 0]
任务队列已经满，任务被拒绝：Task
// ………… 省略部分日志
任务队列已经满，任务被拒绝：Task com.ubuntuvim.spring.thread.MyThreadPoolExecutor$$Lambda$1/142257191@5f184fc6 rejected from java.util.concurrent.ThreadPoolExecutor@4c873330[Running, pool size = 4, active threads = 4, queued tasks = 5, completed tasks = 8]
任务队列已经满，任务被拒绝：Task com.ubuntuvim.spring.thread.MyThreadPoolExecutor$$Lambda$1/142257191@3feba861 rejected from java.util.concurrent.ThreadPoolExecutor@4c873330[Running, pool size = 4, active threads = 4, queued tasks = 5, completed tasks = 9]
下午2:38:53: Task execution finished 'MyThreadPoolExecutor.main()'.

```

从运行结果上看，只有少数几个人能执行成功，其他的任务都被直接拒绝了。



workQueueSize=50的运行结果：

```shell
> Task :test-env:MyThreadPoolExecutor.main()
i = 1
pool-1-thread-1在执行任务：	1
getActiveCount = 1
getTaskCount = 1
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 0
i = 2
getActiveCount = 2
getTaskCount = 2
getCompletedTaskCount = 0
getMaximumPoolSize = 4
pool-1-thread-2在执行任务：	2
workQueueSize = 0
i = 3
getActiveCount = 2
getTaskCount = 3
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 1
i = 4
getActiveCount = 2
getTaskCount = 4
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 2
// 省略类似中间日志
i = 47
getActiveCount = 2
getTaskCount = 47
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 45
i = 48
getActiveCount = 2
getTaskCount = 48
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 46
i = 49
getActiveCount = 2
getTaskCount = 49
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 47
i = 50
getActiveCount = 2
getTaskCount = 50
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 48
pool-1-thread-1在执行任务：	3
pool-1-thread-2在执行任务：	4
pool-1-thread-2在执行任务：	5
pool-1-thread-1在执行任务：	6
pool-1-thread-1在执行任务：	7
pool-1-thread-2在执行任务：	8
pool-1-thread-1在执行任务：	9
// ………… 省略中间的日志
pool-1-thread-2在执行任务：	47
pool-1-thread-1在执行任务：	48
pool-1-thread-2在执行任务：	50
pool-1-thread-1在执行任务：	49

BUILD SUCCESSFUL in 56s
35 actionable tasks: 2 executed, 33 up-to-date
下午11:32:38: Task execution finished 'MyThreadPoolExecutor.main()'.
```

从两次运行日志可以分析出线程池是怎么运行的。

1. 第一种情况，缓冲队列比较小，并且任务数比缓冲队列大的多。

    此情况缓存队列比较小，并且每个任务都很耗时，导致队列也装不下。然后新开了2个线程，一共4个线程连续处理任务。但是让人处理不完，只能把新增的任务拒绝掉。此时任务还在一直往里添加，4个线程也一直在处理任务。当有任务处理完了，就会又接收新任务。所以从日志可以看出来，并不是在第一次出现拒绝异常就不在接收新任务。

2. 第二种情况，直接定义一个比较大缓冲队列，这个队列可以容纳所有的任务

    首先看前面三次运行日志：

    ```
    i = 1
    pool-1-thread-1在执行任务：	1
    getActiveCount = 1
    getTaskCount = 1
    getCompletedTaskCount = 0
    getMaximumPoolSize = 4
    workQueueSize = 0
    i = 2
    getActiveCount = 2
    getTaskCount = 2
    getCompletedTaskCount = 0
    getMaximumPoolSize = 4
    pool-1-thread-2在执行任务：	2
    workQueueSize = 0
    i = 3
    getActiveCount = 2
    getTaskCount = 3
    getCompletedTaskCount = 0
    getMaximumPoolSize = 4
    workQueueSize = 1
    ```

    `i = 1`和`i = 2`的时候，队列是空的。因为此时有两个核心线程，刚好可以接受并处理2个任务。所以不需要缓冲。

    `i = 3`的时候，由于前面两个任务还没处理完成（因为在任务中手动加了休眠，模拟耗时）。此时也没有空余的线程可以接受任务。所以只能缓存到队列中，此时`workQueueSize = 1`。

    `i = 50`的时候，由于队列设置的是50，可以容纳所有的任务。不需要创建新的线程来处理，并且也没有设置任务的超时时间。所以只要等待两个核心线程慢慢处理任务即可。从最后一个日志可以看出来`workQueueSize = 48`和`getActiveCount = 2`是符合预期的。一个是队列大小，一个是线程大小。

```i = 50
getActiveCount = 2
getTaskCount = 50
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 48
```

我们再把`workQueueSize`改成47，预期效果是`getActiveCount`3，运行。

```
i = 1
pool-1-thread-1在执行任务：	1
getActiveCount = 1
getTaskCount = 1
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 0
i = 2
getActiveCount = 2
getTaskCount = 2
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 0
i = 3
getActiveCount = 2
pool-1-thread-2在执行任务：	2
getTaskCount = 3
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 1
i = 4
getActiveCount = 2
getTaskCount = 4
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 2

// 省略部分类似日志

i = 49
getActiveCount = 2
getTaskCount = 49
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 47
i = 50
getActiveCount = 3
getTaskCount = 50
getCompletedTaskCount = 0
getMaximumPoolSize = 4
workQueueSize = 47
```

从运行结果看，是符合预期的。由于队列容纳不下50个任务，只能新开一个线程处理任务，所以`getActiveCount = 3`，并且三个线程是可以处理完50个任务，所以没有出现任务拒绝的情况。



最后再回顾一下前面的要点：

处理任务的优先级为：核心线程`corePoolSize`处理任务、任务队列`workQueue`、最大线程`maximumPoolSize`，如果三者都满了使用handler处理被拒绝的任务。

也就是说先用默认的线程数处理任务，处理不过来则放在缓冲队列中，任务排队处理。处理不过来则创建新的线程帮忙处理，如果还是处理不完，直接把新增的任务拒绝掉。

那么是不是只要设置队列足够大就可以了呢？当然不是，队列设置越大占用CPU、内存也越高，一台机器的硬件资源是固定的，整个程序运行期都需要CPU和内存，不能把资源都给了你这一个线程池使用，那么肯定会影响其他程序的运行。

核心线程`corePoolSize`处理任务、任务队列`workQueue`、最大线程`maximumPoolSize`这三个参数要根据机器硬件和任务耗时的情况做调整。

如果你的任务运行很快，队列可以适当调整大一些，避免任务被拒绝，最大线程数不用设置太大，和CPU核数一致就差不多了。

如果你的任务比较耗时，那么最大线程数就可以适当调整大一下，加大任务的并行数量，提高吞吐。



