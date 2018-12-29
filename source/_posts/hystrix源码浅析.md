---
title: Hystrix源码浅析
date: 2018-12-11 15:28:16
categories: spring cloud源码浅析
tags:
- spring cloud
- hystrix
---

从今年年初，笔者所在地的公司开始向微服务转型，到如今大多数服务已经抽离转型完毕。在此过程中，笔者接触到了spring cloud的相关内容，上手还算容易。在此不得不感谢致力于开源服务的业界前辈及捐献了一整套方案的Netflix公司，他们的努力让笔者所在公司这样的小公司也能快速的进行微服务化的转型。在转型的过程中，笔者跟随Leader虽然已经熟悉了spring cloud的相关内容，但有些东西还是知其然不知其所以然。因此在年终笔者有了些空闲时间，打算系统的了解下spring cloud全家桶的相关组件的实现原理，也便有了这组博客的诞生。

本篇博客主要介绍的是熔断器hystrix，该项目是Netflix API团队于2011年开发完成，后来与Ribbon一直集成在fegin中，以便可以使用注解来方便的声明式的使用它。现在该项目已经停止了开发，进入了维护阶段，当然或许后续广大的开源社区会对它进行二次开发。下面就来了解下hystrix的相关功能以及原理。

### Hystrix能干些什么

每一个开源项目的兴起，都是因为它有着我们需要的特性，那么Hystrix的特性是什么？或者说，它能为我们做些什么？我们都知道在微服务的架构中，服务之间相互依赖调用，一个请求的调用链可能需要经过许多个服务才能到达数据库。而在这个过程中就可能出现很多问题，比如，如果调用链中的某一个服务实例挂了，当然这并不是什么大问题，微服务中的服务一般都会有很多个实例，负载均衡算法会自动将请求导向另一个可用的实例，并且注册中心会自动无感知的将这个挂了的服务剔除。但是如果这个服务的问题是代码的问题，那么这个服务的所有实例都有可能存在这个问题，此时我们面对的问题是在这个调用链中的某一个环节完全不可用，请求无法进行下去，这个时候我们就需要一个降级策略返回给用户一个更友好的信息，而不是一个用户看不懂的异常。

但是现实中bug并不是这么容易发现的，他可能不是一个明显的快速失败的bug，而是仅仅造成了线程阻塞，比如某个同学的锁用的不当，此时请求走到这个服务时就会停留一段时间。如果这个调用的接口有很高的并发，那么就会有大量的请求在这个超时的服务上堆积（事实上服务之间的调用一般会有重试机制，因此实际上的请求会比想像的更多），它们将撑爆线程池，吃过内存，最终这个服务的上游服务会全部挂掉，导致可怕的雪崩。此时，我们就需要一个机制，在超时一定时间后不再重试，而是快速失败，避免造成级联雪崩。

在spring cloud全家桶中，Hystrix便是用来处理这个问题，它可以在服务调用失败时将请求路由到一个本地方法，我们可以在里面返回更加友好的信息，并且它可以设置一个超时机制（一般会比重试超时要长），快速失败，实现一种优雅降级。这看起来有点像保险丝的熔断，因此在汉译中，它被叫做熔断器。

### Hystrix该怎么用

Hystrix的入口有两个，HystrixCommand和HystrixObservableCommand，至于它俩的区别，代码中注释介绍的很简单，只是说了一个是阻塞一个是非阻塞的，至于它俩更多的区别，这里就不再介绍了，本文只介绍HystrixCommand的使用。这两个类我们都可以通过继承的方式来使用，但是官方也提供了一个更方便的方式来使用它们，我们可以使用hystrix-javanica中的@HystrixCommand注解来声明式的使用它们。@HystrixCommand的源码如下：

```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface HystrixCommand {
    String groupKey() default "";
    String commandKey() default "";
    String threadPoolKey() default "";
    String fallbackMethod() default "";
    HystrixProperty[] commandProperties() default {};
    HystrixProperty[] threadPoolProperties() default {};
    Class<? extends Throwable>[] ignoreExceptions() default {};
    ObservableExecutionMode observableExecutionMode() default ObservableExecutionMode.EAGER;
    HystrixException[] raiseHystrixExceptions() default {};
    String defaultFallback() default "";
}

```

这个注解中定义了很多方法，要想都正确使用我们可能需要详细了解Hystrix的使用原理，而本篇博客仅限于使用，笔者不想介绍的太过深入。感兴趣的读者可以详细研究下Hystrix的源码，其实更简单的方法可能是直接看注释，而笔者这里因为篇幅的原因注释并没有粘上来。这里笔者只说下fallbackMethod，这个方法用于指定一个降级方法，实际上指定它我们便可以简单使用HystrixCommand来实现一个熔断降级了。而要想使用HystrixObservableCommand，我们也可以使用observableExecutionMode，来指定Observable加载的模式。

因此，当我们要使用HystrixCommand注解给一个方法实现降级时，我们可以像下面这样使用它。

```java
    @HystrixCommand(fallbackMethod = "serviceFallback")
    public String exchange(String arg1, String arg2,...) {
        ......
    }

    /**
     * 用于Hystrix降级回调使用
     * @param arg1
     * @param arg2
     * @return
     */
    private String serviceFallback(String arg1, String arg2, ..., Throwable throwable) {
        ......
    }
```

如上面所示，我们可以通过fallbackMethod指定降级方法，并在该方法中处理异常以及返回用户一个友好信息。

上面这个方法可以很方便的使用Hystrix，但实际情况中我们可能很少这样使用，因为在微服务中很多东西都不是独立的，它需要和其他工具一起配合共同完成微服务的功能。Hystrix用于处理服务调用出错的情况，就很容易想到将它与服务的调用框架放在一起使用，在Spring cloud中，服务之间的调用以及负载均衡是使用Ribbon实现的。因此在Spring cloud中它和Hystrix被整合在了一起，共同组成了一个Feign框架。在Feign中我们一般这样使用Hystrix。

```java
@FeignClient(name = "hello", fallback = HystrixClientFallback.class)
protected interface HystrixClient {
    @RequestMapping(method = RequestMethod.GET, value = "/hello")
    Hello iFailSometimes();
}

static class HystrixClientFallback implements HystrixClient {
    @Override
    public Hello iFailSometimes() {
        return new Hello("fallback");
    }
}
```

在FeignClient中，name是用于注定要调用的服务名的，而fallback即是指定一个本地的降级实现类，在这个实现类中对相应的方法进行降级处理。但是细心的同学可能会发现一个问题，采用实现类的方式的话，子类和接口就必须得是一样的签名，因此就没办法在降级时再捕获异常，输出异常信息。如果笔者希望对异常进行处理的话，可以这样实现：

```java
@FeignClient(name = "hello", fallbackFactory = HystrixClientFallbackFactory.class)
protected interface HystrixClient {
	@RequestMapping(method = RequestMethod.GET, value = "/hello")
	Hello iFailSometimes();
}

@Component
static class HystrixClientFallbackFactory implements FallbackFactory<HystrixClient> {
	@Override
	public HystrixClient create(Throwable cause) {
		return new HystrixClient() {
			@Override
			public Hello iFailSometimes() {
				return new Hello("fallback; reason was: " + cause.getMessage());
			}
		};
	}
}
```

### Hystrix做了些什么

上面介绍了Hystrix的用法，下面我们就通过第一个例子来跟踪源码，看看在加了注解HystrixCommand后，Hystrix到底做了些啥。HystrixCommand注解的切面在hystrix-javanica包中，下面是入口的源码。

```java
@Pointcut("@annotation(com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand)")

    public void hystrixCommandAnnotationPointcut() {
    }

    @Pointcut("@annotation(com.netflix.hystrix.contrib.javanica.annotation.HystrixCollapser)")
    public void hystrixCollapserAnnotationPointcut() {
    }

    @Around("hystrixCommandAnnotationPointcut() || hystrixCollapserAnnotationPointcut()")
    public Object methodsAnnotatedWithHystrixCommand(final ProceedingJoinPoint joinPoint) throws Throwable {
        Method method = getMethodFromTarget(joinPoint);
        Validate.notNull(method, "failed to get method from joinPoint: %s", joinPoint);
        if (method.isAnnotationPresent(HystrixCommand.class) && method.isAnnotationPresent(HystrixCollapser.class)) {
            throw new IllegalStateException("method cannot be annotated with HystrixCommand and HystrixCollapser " +
                    "annotations at the same time");
        }
        MetaHolderFactory metaHolderFactory = META_HOLDER_FACTORY_MAP.get(HystrixPointcutType.of(method));
        MetaHolder metaHolder = metaHolderFactory.create(joinPoint);
        HystrixInvokable invokable = HystrixCommandFactory.getInstance().create(metaHolder);
        ExecutionType executionType = metaHolder.isCollapserAnnotationPresent() ?
                metaHolder.getCollapserExecutionType() : metaHolder.getExecutionType();

        Object result;
        try {
            if (!metaHolder.isObservable()) {
                result = CommandExecutor.execute(invokable, executionType, metaHolder);
            } else {
                result = executeObservable(invokable, executionType, metaHolder);
            }
        } catch (HystrixBadRequestException e) {
            throw e.getCause();
        } catch (HystrixRuntimeException e) {
            throw hystrixRuntimeExceptionToThrowable(metaHolder, e);
        }
        return result;
    }
```

