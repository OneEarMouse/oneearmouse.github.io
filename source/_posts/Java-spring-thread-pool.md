---
title: Java与Spring中的线程池
date: 2022-01-30 11:11:24
categories: Java
tags:
description: 简单介绍Java中Runnable与Callable的区别，以及Java与Spring中的线程池简单使用方法。最后介绍了Spring中线程池的调用机制以及@Async的使用
---

# Runnable 与 Callable<T>
## Runable
Thread接受Runnable类型，并且允许接受Runnable构造线程。线程启动后会自动调用Runnable中的run方法
```java
public class Main {
    public static void main(String[] args) {
        Thread t = new Thread(new MyRunnable());
        t.start(); // 启动新线程
    }
}

class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("start new thread!");
    }
}
```

问题：Runnable接口的run方法只能执行，没有返回值。如果任务需要读取多线程任务的返回值，则非常不方便

## Callable<T>
定义：
```java
class Task implements Callable<String> {
    public String call() throws Exception {
        return "SomeThing"; 
    }
}
```
Callable接口是一个泛型接口，可以返回指定类型的结果。但如何接受指定类型的结果？

ExecutorService.submit()方法，可以看到，它返回了一个Future类型，一个Future类型的实例代表一个未来能获取结果的对象：

```java
ExecutorService executor = Executors.newFixedThreadPool(3); 
// 定义任务:
Callable<String> task = new Task();
// 提交任务并获得Future:
Future<String> future = executor.submit(task);
// 等待线程执行完毕（非必须）
Thread.sleep(1000)
// 从Future获取异步执行返回的结果:
String result = future.get(); // 可能阻塞
```

当我们在线程池中提交Callable任务后，会得到一个```Future<T>```对象。之后，在主线程等待子线程执行完毕，然后再通过```Future.get()```获取子线程的输出结果。

## Future使用方法
```java
//阻塞等待，获取结果
Future.get();
//阻塞等待，获取结果。并设置了超时时间
//如果超时依然未获取，则抛出TimeoutException
Future.get(long timeout, TimeUnit unit);
//取消当前Future对应的线程执行的任务
Future.cancel(boolean mayInterruptIfRunning);
//判断Future对应的线程执行的任务是否完成（注：cancel也算入完成）
Future.isDone();
//判断Future对应的线程执行的任务是否在完成前被取消
Future.isCalcelled();
```

# 线程池
我们通常将Future与线程池结合使用。将task提交到线程池中，再等待线程执行完毕后，获取Future的结果。这里不详细介绍java的线程池，直接介绍Spring中的线程池


## java中的ThreadPoolExecutor
构造方法
```java
    public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, Executors.defaultThreadFactory(), defaultHandler);
    }

    public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory, defaultHandler);
    }

    public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, RejectedExecutionHandler handler) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, Executors.defaultThreadFactory(), handler);
    }

    public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler)
  
```
java中的ThreadPoolExecutor，可用的参数有：
- corePoolSize: 核心线程数量。决定了添加的任务是开辟新的线程去执行，还是放到workQueue任务队列中去；
- maximumPoolSize：最大线程数量。决定线程池最大线程数量
- keepAliveTime：当线程池中空闲线程数量超过corePoolSize时，多余的线程会在多长时间内被销毁；
- unit：keepAliveTime的时间单位
- workQueue：任务队列。被添加到线程池中，但仍在等待执行的任务
- threadFactory：线程工程，用于线程创建
- rejectedExecutionHandler：任务数量过多时，拒绝任务的策略

### 队列
通过workQueue中传入队列的不同，ThreadPoolExecutor可以有不同的工作模式

队列可以分为：
- 直接提交队列SynchronousQueue：提交的任务不会被保存，总是会马上提交执行，超过maxPoolSize上限则直接拒绝。
- 有界任务队列ArrayBlockingQueue：超过poolCoreSize的任务会被储存在队列中等待执行。超过队列上线后超过maxPoolSize则拒绝
- 无界任务队列LinkedBlockingQueue：超过poolCoreSize的任务会被储存在队列中等待执行。队列无上线（任务永远会被暂存在队列中）
- 优先任务队列PriorityBlockingQueue：放入队列中的任务按照优先级重排序（相当于特殊的无界队列）

### 拒绝策略
ThreadPoolExecutor自带的拒绝策略如下：
1. AbortPolicy策略：该策略会直接抛出异常，阻止系统正常工作；
2. CallerRunsPolicy策略：如果线程池的线程数量达到上限，该策略会把任务队列中的任务放在调用者线程当中运行；
3. DiscardOledestPolicy策略：该策略会丢弃任务队列中最老的一个任务，也就是当前任务队列中最先被添加进去的，马上要被执行的那个任务，并尝试再次提交；
4. DiscardPolicy策略：该策略会默默丢弃超过上限的任务，不予任何处理。当然使用此策略，业务场景中需允许任务的丢失；


## spring自带线程池ThreadPoolTaskExecutor
Spring异步线程池的接口类是TaskExecutor，本质其本质是对java.util.concurrent.ThreadPoolExecutor的包装。

配置
```java
public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        //最大线程数
        executor.setMaxPoolSize(maxPoolSize);
        //核心线程数
        executor.setCorePoolSize(corePoolSize);
        //任务队列的大小
        executor.setQueueCapacity(queueCapacity);
        //线程前缀名
        executor.setThreadNamePrefix(namePrefix);
        //线程存活时间
        executor.setKeepAliveSeconds(keepAliveSeconds);
        //拒绝策略
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.AbortPolicy());
        //线程初始化
        executor.initialize();
        return executor;
    }
```

可以看到，其使用方法与java自带的ThreadPoolExecutor非常类似。但用户不再需要通过管理复杂的队列来管理线程池的调度方式。

## 线程调度策略
spring自带线程池ThreadPoolTaskExecutor的调度策略综合了java中的ThreadPoolExecutor使用不同队列时的不同调度方式。通过改变```corePoolSize```、```maxPoolSize```、```queueCapacity```即可改变调度方式。我们来重新理解一下几个改变

1. corePoolSize
   线程池的基本大小，即在没有任务需要执行的时候线程池的大小，并且只有在工作队列满了的情况下才会创建超出这个数量的线程。
2. maximumPoolSize
   线程池中允许的最大线程数
3. poolSize:
   线程池中当前线程的数量
4. queueCapacity
   等待队列大小

查看源码（基于jdk8）：
```java
public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        /*
         * Proceed in 3 steps:
         *
         * 1. If fewer than corePoolSize threads are running, try to
         * start a new thread with the given command as its first
         * task.  The call to addWorker atomically checks runState and
         * workerCount, and so prevents false alarms that would add
         * threads when it shouldn't, by returning false.
         *
         * 2. If a task can be successfully queued, then we still need
         * to double-check whether we should have added a thread
         * (because existing ones died since last checking) or that
         * the pool shut down since entry into this method. So we
         * recheck state and if necessary roll back the enqueuing if
         * stopped, or start a new thread if there are none.
         *
         * 3. If we cannot queue task, then we try to add a new
         * thread.  If it fails, we know we are shut down or saturated
         * and so reject the task.
         */
        int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }
```

很明显，处理流程为：
1. 如果当前线程数量没有达到核心线程数量```poolSize<corePoolSize```，无论是否有空闲线程，都新增一个线程处理提交新的任务
2. 如当前线程数量大于核心线程数量```poolSize>=corePoolSize```，并且任务队列未满，则将任务加入队列中等待
3. 如果当前线程数量对等于核心线程数量```poolSize>=corePoolSize```，并且任务队列满时，
   3.1. 如果当前线程数量小于最大线程数量```poolSize<maximumPoolSize```，那么新增线程处理任务
   3.2. 如果当前线程数量等于最大线程数量```poolSize=maximumPoolSize```，那么执行拒绝策略拒绝任务

# Spring中的@Async

注：本文源码基于spring boot 1.5.12。 高版本的Async逻辑有所不同，但使用的依然是没有上限的线程池

## Async的使用方法
首先在application上标注```@EnableAsync```启用async
```java
@SpringBootApplication
@EnableFeignClients
@EnableAsync
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

在方法上标注```@Async```，被标注的方法即变为异步方法。在类上标注```@Async```则被标注的类中所有方法都会变为异步方法

建议将异步方法的返回值设为```void```或```Future<T>```。否则难以获取返回值

```java
@Async
public void useAsync(Object inputObj){
    // do something
}

@Async
public Future<String> useAsync(Object inputObj){
    return "something";
}

@Async
public class asyncClass{
    public void test(){}
}

```

## 为什么阿里编程规范不建议使用@Async
我们来看看Async的源码
@EnableAsync源码
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AsyncConfigurationSelector.class)
public @interface EnableAsync {

    // 支持用户自定义注解，扫描用户标注的@Async
	/**
	 * Indicate the 'async' annotation type to be detected at either class
	 * or method level.
	 * <p>By default, both Spring's @{@link Async} annotation and the EJB 3.1
	 * {@code @javax.ejb.Asynchronous} annotation will be detected.
	 * <p>This attribute exists so that developers can provide their own
	 * custom annotation type to indicate that a method (or all methods of
	 * a given class) should be invoked asynchronously.
	 */
	Class<? extends Annotation> annotation() default Annotation.class;

    // 标明是否需要创建CGLIB子代理
	/**
	 * Indicate whether subclass-based (CGLIB) proxies are to be created as opposed
	 * to standard Java interface-based proxies.
	 * <p><strong>Applicable only if the {@link #mode} is set to {@link AdviceMode#PROXY}</strong>.
	 * <p>The default is {@code false}.
	 * <p>Note that setting this attribute to {@code true} will affect <em>all</em>
	 * Spring-managed beans requiring proxying, not just those marked with {@code @Async}.
	 * For example, other beans marked with Spring's {@code @Transactional} annotation
	 * will be upgraded to subclass proxying at the same time. This approach has no
	 * negative impact in practice unless one is explicitly expecting one type of proxy
	 * vs. another &mdash; for example, in tests.
	 */
	boolean proxyTargetClass() default false;

    // 标明异步通知将会如何实现，默认PROXY
	/**
	 * Indicate how async advice should be applied.
	 * <p><b>The default is {@link AdviceMode#PROXY}.</b>
	 * Please note that proxy mode allows for interception of calls through the proxy
	 * only. Local calls within the same class cannot get intercepted that way; an
	 * {@link Async} annotation on such a method within a local call will be ignored
	 * since Spring's interceptor does not even kick in for such a runtime scenario.
	 * For a more advanced mode of interception, consider switching this to
	 * {@link AdviceMode#ASPECTJ}.
	 */
	AdviceMode mode() default AdviceMode.PROXY;

    // 标明标明异步注解bean处理器应该遵循的执行顺序，默认最低的优先级
	/**
	 * Indicate the order in which the {@link AsyncAnnotationBeanPostProcessor}
	 * should be applied.
	 * <p>The default is {@link Ordered#LOWEST_PRECEDENCE} in order to run
	 * after all other post-processors, so that it can add an advisor to
	 * existing proxies rather than double-proxy.
	 */
	int order() default Ordered.LOWEST_PRECEDENCE;

}

```

好，看到了```@Import(AsyncConfigurationSelector.class)```，来看看
```java
public class AsyncConfigurationSelector extends AdviceModeImportSelector<EnableAsync> {

	private static final String ASYNC_EXECUTION_ASPECT_CONFIGURATION_CLASS_NAME =
			"org.springframework.scheduling.aspectj.AspectJAsyncConfiguration";

	// 决定选择哪个ProxyAsyncConfiguration
	@Override
	public String[] selectImports(AdviceMode adviceMode) {
		switch (adviceMode) {
			case PROXY:
				return new String[] { ProxyAsyncConfiguration.class.getName() };
			case ASPECTJ:
				return new String[] { ASYNC_EXECUTION_ASPECT_CONFIGURATION_CLASS_NAME };
			default:
				return null;
		}
	}

}
```

这里的```ProxyAsyncConfiguration.class```继承于```AbstractAsyncConfiguration```，再看
```java
@Configuration
public abstract class AbstractAsyncConfiguration implements ImportAware {

	protected AnnotationAttributes enableAsync;

	protected Executor executor;

	protected AsyncUncaughtExceptionHandler exceptionHandler;

    // 找到所有AsyncConfiguer并使用其配置
	/**
	 * Collect any {@link AsyncConfigurer} beans through autowiring.
	 */
	@Autowired(required = false)
	void setConfigurers(Collection<AsyncConfigurer> configurers) {
		if (CollectionUtils.isEmpty(configurers)) {
			return;
		}
		if (configurers.size() > 1) {
			throw new IllegalStateException("Only one AsyncConfigurer may exist");
		}
		AsyncConfigurer configurer = configurers.iterator().next();
		this.executor = configurer.getAsyncExecutor();
		this.exceptionHandler = configurer.getAsyncUncaughtExceptionHandler();
	}

}

```

那么如果没有实现AsyncConfigurer，程序默认会创建什么？带着这个疑问，我们找到了
```java
@Configuration
@ConditionalOnProperty(value = "spring.sleuth.async.enabled", matchIfMissing = true)
@ConditionalOnBean(Tracer.class)
@AutoConfigureAfter(AsyncCustomAutoConfiguration.class)
public class AsyncDefaultAutoConfiguration {

	@Autowired private BeanFactory beanFactory;

    // 如果找不到AsyncConfigurer的bean就执行
	@Configuration
	@ConditionalOnMissingBean(AsyncConfigurer.class)
	@ConditionalOnProperty(value = "spring.sleuth.async.configurer.enabled", matchIfMissing = true)
	static class DefaultAsyncConfigurerSupport extends AsyncConfigurerSupport {

		@Autowired private BeanFactory beanFactory;

		@Override
		public Executor getAsyncExecutor() {
			return new LazyTraceExecutor(this.beanFactory, new SimpleAsyncTaskExecutor());
		}
	}

}
```

好了，我们知道了。如果我们没有自定义任何一个AsyncConfigurer，则程序会选择```SimpleAsyncTaskExecutor```作为默认的线程池。

再来看看这个默认线程池
```java
/**
 * {@link TaskExecutor} implementation that fires up a new Thread for each task,
 * executing it asynchronously.
 *
 * <p>Supports limiting concurrent threads through the "concurrencyLimit"
 * bean property. By default, the number of concurrent threads is unlimited.
 *
 * <p><b>NOTE: This implementation does not reuse threads!</b> Consider a
 * thread-pooling TaskExecutor implementation instead, in particular for
 * executing a large number of short-lived tasks.
 *
 * @author Juergen Hoeller
 * @since 2.0
 * @see #setConcurrencyLimit
 * @see SyncTaskExecutor
 * @see org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor
 * @see org.springframework.scheduling.commonj.WorkManagerTaskExecutor
 */
@SuppressWarnings("serial")
public class SimpleAsyncTaskExecutor extends CustomizableThreadCreator
		implements AsyncListenableTaskExecutor, Serializable {
            // 省略
        }
```

好了，我们知道了这个默认线程池的特点：
- 每个task都创建一个新线程
- 如果不创建concurrencyLimit bean，则线程数量无上限
- SimpleAsyncTaskExecutor不会复用线程，每次都是新创建

所以，SimpleAsyncTaskExecutor其实不是真的线程池，而是一个不断生成新线程的工具，对线程的利用管理非常低下。


那么，如果我们只使用了```@EnableAsync```和```@Async```。那么spring就会给我们创建这个默认的```SimpleAsyncTaskExecutor```。每次进行线程异步调用，系统都会创建新线程执行任务并丢弃。好的，问题出现了：如果异步调用的线程数量不多，那么没有任何问题。如果异步调用的线程面临高并发的问题，那么就会出现大量的线程创建及占用，对系统造成大量负担。最终可能导致线上服务爆线程等问题


## 安全使用方法
### 方式一：指定线程池
@Async方法可以通过传入Executor的Bean名称，指定线程池。因此我们可以通过自定义线程池，配置参数，来控制@Async方法使用的线程池
```java
    @Bean("Executor")
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        // add thread config
        executor.initialize();
        return executor;
    }

    @Async("Executor")
    public void asyncMethod(){
        // do something
    }
```

### 方式二：实现接口AsyncConfigurer
这种方法相当于配置一个全局的线程池供Async使用。所有标注@Async并且没有指定线程池的方法，都会放入这个默认的Async线程池中执行。
```java
public class asyncConfig implements AsyncConfigurer{
        private Executor newExecutor() {
            ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
            // add thread config
            executor.initialize();
            return executor;
        }

        @Override
        public Executor getAsyncExecutor() {
            return newExecutor();
        }

        @Override
        public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
            return null;
        }
    }
```


