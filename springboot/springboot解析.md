## 概述：

​		前阵子看到了SpringCloud社区的一个开源项目，主要是对服务发现增强的功能。研究项目的时候发现代码简练，优雅，最主要是spring ioc和aop特性应用的得心应手。若非对源码有深入研究，不可能写出这么优秀的开源项目。另外在现有的springboot专栏中，大多数博文旨在应用，对一些中间件的整合之类，源码分析的博客数量有限。鉴于以上两方面，该系列应运而生。

​		该系列主要还是Spring的核心源码，不过目前Springboot大行其道，所以就从Springboot开始分析。最新版本是Springboot2.5.3，Spring5，所以新特性本系列后面也会着重分析。

> 整个系列会围绕springboot启动流程进行源码分析，在整个流程中，会遇到一些核心类或者核心流程，会着重讲解，所以篇幅可能会增多，做好准备。

```java
@SpringBootApplication
public class SpringbootApplication {

    public static void main(String[] args) {
        //提供主启动类对象，new SpringApplication(SpringbootApplication.class).run(args)
        SpringApplication.run(SpringbootApplication.class, args);
    }

}
```

点进run方法后

```java
//实例化SpringApplication后调用其run方法
@SuppressWarnings({ "unchecked", "rawtypes" })
	public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		//默认重载的this(null,SpringbootApplication.class),所以这里的resourceLoader是null
        this.resourceLoader = resourceLoader;
        //断言该主启动类不能传空值
		Assert.notNull(primarySources, "PrimarySources must not be null");
        //对其主类进行去重
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
        //通过判断classpath判断web类型
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
        //从 Spring 工厂获取 Bootstrap Registry Initializers
		this.bootstrapRegistryInitializers = getBootstrapRegistryInitializersFromSpringFactories();
        // 获取 Spring 工厂实例（spring.factory中定义的Initializers） -> 应用程序上下文初始化程序
		setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
        // 获取 Spring 工厂实例（spring.factory中定义的Application Listeners） -> 应用程序监听器
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
        // 推导出主应用程序类(根据程序栈stackTraceElement中调用的main方法反推，若存在多个main方法则会报错)
		this.mainApplicationClass = deduceMainApplicationClass();
	}
```

```java
//deduceFromClasspath方法
static WebApplicationType deduceFromClasspath() {
    //WEBFLUX_INDICATOR_CLASS = "org.springframework.web.reactive.DispatcherHandler"
    //通过判断应用内是否有该类判断是否是嵌入式网络工程
		if (ClassUtils.isPresent(WEBFLUX_INDICATOR_CLASS, null) && !ClassUtils.isPresent(WEBMVC_INDICATOR_CLASS, null)
				&& !ClassUtils.isPresent(JERSEY_INDICATOR_CLASS, null)) {
			return WebApplicationType.REACTIVE;
		}
    //SERVLET_INDICATOR_CLASSES = { "javax.servlet.Servlet",
			"org.springframework.web.context.ConfigurableWebApplicationContext" }
		for (String className : SERVLET_INDICATOR_CLASSES) {
            //若都没有则不是网络工程
			if (!ClassUtils.isPresent(className, null)) {
				return WebApplicationType.NONE;
			}
		}
		//否则则是web项目
		return WebApplicationType.SERVLET;
	}
```

> 简单的阅读过后，我们得出一个结论，这部分就是一些基本类信息的获取和设置，类似于环境准备的功能，那么我们最终得到的几个资源就是。
>
> - 主要资源列表
> - Web App类型Servlet
> - 应用程序上下文初始化程序
>   - org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener
> - 应用程序监听器
>   - "org.springframework.boot.cloud.CloudFoundryVcapEnvironmentPostProcessor"
>   - "org.springframework.boot.context.config.ConfigDataEnvironmentPostProcessor"
>   - "org.springframework.boot.env.RandomValuePropertySourceEnvironmentPostProcessor"
>   - "org.springframework.boot.env.SpringApplicationJsonEnvironmentPostProcessor"
>   - "org.springframework.boot.env.SystemEnvironmentPropertySourceEnvironmentPostProcessor"
>   -  "org.springframework.boot.reactor.DebugAgentEnvironmentPostProcessor"
>   - "org.springframework.boot.autoconfigure.integration.IntegrationPropertiesEnvironmentPostProcessor"

```java
/**
 * run 方法
 */
public ConfigurableApplicationContext run(String... args) {
    // 启动一个简单的秒表
    // tip：用于计时，可以忽略
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    // 创建引导上下文
    DefaultBootstrapContext bootstrapContext = createBootstrapContext();
    // 定义可配置的应用程序上下文变量
    ConfigurableApplicationContext context = null;
    // 配置无头属性
    configureHeadlessProperty();
    // 获取运行监听器
    SpringApplicationRunListeners listeners = getRunListeners(args);
    // 启动监听器
    listeners.starting(bootstrapContext, this.mainApplicationClass);
    try {
        // 默认应用程序参数
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        // 准备环境
        ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
        // 配置忽略 Bean 信息
        configureIgnoreBeanInfo(environment);
        // 打印横幅 Spring图标（可自定义）
        Banner printedBanner = printBanner(environment);
        // 创建应用程序上下文
        context = createApplicationContext();
        // 设置应用程序启动
        context.setApplicationStartup(this.applicationStartup);
        // 准备上下文
        prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
        // 刷新上下文
        // 这里会涉及启动Spring
        // 启动Web服务
        refreshContext(context);
        // 刷新后执行
        // tip：未实现为了扩展性
        afterRefresh(context, applicationArguments);
        // 时钟结束
        stopWatch.stop();
        // 耗时打印
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
        }
        listeners.started(context);
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, listeners);
        throw new IllegalStateException(ex);
    }
```

## 逐行阅读

### 从类路径推断Web类型

> org.springframework.boot.WebApplicationType#deduceFromClasspath

```java
static WebApplicationType deduceFromClasspath() {
    // 判断org.springframework.web.reactive.DispatcherHandler存在，并且org.springframework.web.servlet.DispatcherServlet和org.glassfish.jersey.servlet.ServletContainer不存在
    // 则采用reactive方式
    if (ClassUtils.isPresent(WEBFLUX_INDICATOR_CLASS, null) && !ClassUtils.isPresent(WEBMVC_INDICATOR_CLASS, null)
            && !ClassUtils.isPresent(JERSEY_INDICATOR_CLASS, null)) {
        return WebApplicationType.REACTIVE;
    }
    // 存在"javax.servlet.Servlet", "org.springframework.web.context.ConfigurableWebApplicationContext"其中一个则返回NONE
    for (String className : SERVLET_INDICATOR_CLASSES) {
        if (!ClassUtils.isPresent(className, null)) {
            return WebApplicationType.NONE;
        }
    }
    // 如果都匹配不到则采用默认Servlet
    return WebApplicationType.SERVLET;
}
```

### 从 Spring 工厂获取 Bootstrap Registry Initializers

> org.springframework.boot.SpringApplication#getBootstrapRegistryInitializersFromSpringFactories

```java
private List<BootstrapRegistryInitializer> getBootstrapRegistryInitializersFromSpringFactories() {
    // 初始化列表
    ArrayList<BootstrapRegistryInitializer> initializers = new ArrayList<>();
    getSpringFactoriesInstances(Bootstrapper.class).stream()
            .map((bootstrapper) -> ((BootstrapRegistryInitializer) bootstrapper::initialize))
            .forEach(initializers::add);
    // 引导注册表初始化程序
    initializers.addAll(getSpringFactoriesInstances(BootstrapRegistryInitializer.class));
    return initializers;
}
```

### 加载Spring工厂

> org.springframework.core.io.support.SpringFactoriesLoader#loadSpringFactories
>
> 框架内内部使用的通用工厂加载机制。
> SpringFactoriesLoader从“META-INF/spring.factories”文件加载和实例化给定类型的工厂，这些文件可能存在于类路径中的多个 JAR 文件中。 spring.factories文件必须是Properties格式，其中 key 是接口或抽象类的完全限定名称，value 是逗号分隔的实现类名称列表。 例如：
> example.MyService=example.MyServiceImpl1,example.MyServiceImpl2
> 其中example.MyService是接口名称， MyServiceImpl1和MyServiceImpl2是两个实现。

上面是**org.springframework.core.io.support.SpringFactoriesLoader**的类介绍，如果看完这部分介绍，已经有点眉目了，那么我下面就是这个类中具体干活的一个方法的逐行解析。

```java
private static Map<String, List<String>> loadSpringFactories(ClassLoader classLoader) {
    // 获取相应类加载器中内容
    Map<String, List<String>> result = cache.get(classLoader);
    // 存在则返回类加载器中内容
    if (result != null) {
        return result;
    }

    // 不存在则初始化类加载器中内容
    result = new HashMap<>();
    try {
        // 获取资源 -> META-INF/spring.factories 列表
        // 可能存在多个META-INF/spring.factories 文件
        Enumeration<URL> urls = classLoader.getResources(FACTORIES_RESOURCE_LOCATION);
        // 循环加载
        while (urls.hasMoreElements()) {
            // 获取 META-INF/spring.factories 文件URL地址
            URL url = urls.nextElement();
            // 加载资源
            UrlResource resource = new UrlResource(url);
            // 加载资源中配置
            Properties properties = PropertiesLoaderUtils.loadProperties(resource);
            // 循环配置 类似properties key:value形式
            for (Map.Entry<?, ?> entry : properties.entrySet()) {
                String factoryTypeName = ((String) entry.getKey()).trim();
                // 逗号分隔列表到字符串数组
                String[] factoryImplementationNames =
                        StringUtils.commaDelimitedListToStringArray((String) entry.getValue());    
                // 循环value中子项到列表中
                for (String factoryImplementationName : factoryImplementationNames) {
                    result.computeIfAbsent(factoryTypeName, key -> new ArrayList<>())
                            .add(factoryImplementationName.trim());
                }
            }
        }

        // 列表去重
        result.replaceAll((factoryType, implementations) -> implementations.stream().distinct()
                .collect(Collectors.collectingAndThen(Collectors.toList(), Collections::unmodifiableList)));
        // 保存该列表
        cache.put(classLoader, result);
    }
    catch (IOException ex) {
        throw new IllegalArgumentException("Unable to load factories from location [" +
                FACTORIES_RESOURCE_LOCATION + "]", ex);
    }
    return result;
}
```

具体解析一下这部分代码，从缓存（cache）中根据不同的类加载器获取其中的键值对，如果存在则返回类加载器中的内容，不重复读取。如果没找到键值对，则获取类加载器中能获取到的META-INF/spring.factories文件中的内容，逐一解析最后放到缓存中，等待下次获取。

这部分方法可以看出，SpringBoot设置了一些默认的接口类和具体实现类到配置文件中，类似于以前经常用到的properties文件的方式。

例如：

> example.MyService=example.MyServiceImpl1,example.MyServiceImpl2

前者是接口类，后者是该接口的具体实现方法。

### 获取 Spring 工厂实例

> org.springframework.boot.SpringApplication#getSpringFactoriesInstances(java.lang.Class<T>, java.lang.Class<?>[], java.lang.Object...)

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
    // 获取类加载器
    ClassLoader classLoader = getClassLoader();
    // 使用名称并确保唯一以防止重复
    // Use names and ensure unique to protect against duplicates
    Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
    // 创建 Spring 工厂实例
    List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
    AnnotationAwareOrderComparator.sort(instances);
    return instances;
}
```

### 获取类加载器

> org.springframework.boot.SpringApplication#getClassLoader

```java
public ClassLoader getClassLoader() {
    if (this.resourceLoader != null) {
        return this.resourceLoader.getClassLoader();
    }
    return ClassUtils.getDefaultClassLoader();
}
```

### 获取 Spring 工厂实例

> org.springframework.boot.SpringApplication#createSpringFactoriesInstances

```java
private <T> List<T> createSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes,
            ClassLoader classLoader, Object[] args, Set<String> names) {
    // 创建实例列表
    List<T> instances = new ArrayList<>(names.size());
    // 循环列表
    for (String name : names) {
        try {
            // 根据类名获取Class
            Class<?> instanceClass = ClassUtils.forName(name, classLoader);
            // 判断子类是否可分配
            Assert.isAssignable(type, instanceClass);
            // 获取构造器
            Constructor<?> constructor = instanceClass.getDeclaredConstructor(parameterTypes);
            // 实例化对象
            T instance = (T) BeanUtils.instantiateClass(constructor, args);
            // 将实例化结果放到实例化列表中
            instances.add(instance);
        }
        catch (Throwable ex) {
            throw new IllegalArgumentException("Cannot instantiate " + type + " : " + name, ex);
        }
    }
    return instances;
}
```

### 创建引导上下文

> org.springframework.boot.SpringApplication#createBootstrapContext

```java
private DefaultBootstrapContext createBootstrapContext() {
    DefaultBootstrapContext bootstrapContext = new DefaultBootstrapContext();
    this.bootstrapRegistryInitializers.forEach((initializer) -> initializer.initialize(bootstrapContext));
    return bootstrapContext;
}
```

### 实例化类

> org.springframework.beans.BeanUtils#instantiateClass(java.lang.reflect.Constructor<T>, java.lang.Object...)

```java
public static <T> T instantiateClass(Constructor<T> ctor, Object... args) throws BeanInstantiationException {
    Assert.notNull(ctor, "Constructor must not be null");
    try {
        ReflectionUtils.makeAccessible(ctor);
        if (KotlinDetector.isKotlinReflectPresent() && KotlinDetector.isKotlinType(ctor.getDeclaringClass())) {
            return KotlinDelegate.instantiateClass(ctor, args);
        }
        else {
            Class<?>[] parameterTypes = ctor.getParameterTypes();
            Assert.isTrue(args.length <= parameterTypes.length, "Can't specify more arguments than constructor parameters");
            Object[] argsWithDefaultValues = new Object[args.length];
            for (int i = 0 ; i < args.length; i++) {
                if (args[i] == null) {
                    Class<?> parameterType = parameterTypes[i];
                    argsWithDefaultValues[i] = (parameterType.isPrimitive() ? DEFAULT_TYPE_VALUES.get(parameterType) : null);
                }
                else {
                    argsWithDefaultValues[i] = args[i];
                }
            }
            return ctor.newInstance(argsWithDefaultValues);
        }
    }
    catch (InstantiationException ex) {
        throw new BeanInstantiationException(ctor, "Is it an abstract class?", ex);
    }
    catch (IllegalAccessException ex) {
        throw new BeanInstantiationException(ctor, "Is the constructor accessible?", ex);
    }
    catch (IllegalArgumentException ex) {
        throw new BeanInstantiationException(ctor, "Illegal arguments for constructor", ex);
    }
    catch (InvocationTargetException ex) {
        throw new BeanInstantiationException(ctor, "Constructor threw exception", ex.getTargetException());
    }
}
```

> 这部分的因为涉及Spring源码太多，最主要的就是
> ctor.newInstance(argsWithDefaultValues)
> 这个方法，用构造器实例化对象。

### Spring刷新

> org.springframework.context.support.AbstractApplicationContext#refresh

```java
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");

        // 准备刷新
        // tip：一些设置参数，可不细看
        prepareRefresh();

        // 告诉子类刷新内部 bean 工厂
        // 获取刷新 bean 工厂
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // 准备 bean 工厂
        prepareBeanFactory(beanFactory);

        try {
            // 允许在上下文子类中对 bean 工厂进行后处理。
            // tip：这部分涉及Web服务器的启动，如servlet
            postProcessBeanFactory(beanFactory);

            StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
            // 调用在上下文中注册为 bean 的工厂处理器。
            invokeBeanFactoryPostProcessors(beanFactory);

            // 注册拦截 bean 创建的 bean 处理器。
            registerBeanPostProcessors(beanFactory);
            beanPostProcess.end();

            // 初始化此上下文的消息源。
            initMessageSource();

            // 为此上下文初始化事件多播器。
            initApplicationEventMulticaster();

            // 初始化特定上下文子类中的其他特殊 bean。
            onRefresh();

            // 检查侦听器 bean 并注册它们。
            registerListeners();

            // 实例化所有剩余的（非延迟初始化）单例。
            finishBeanFactoryInitialization(beanFactory);

            // 最后一步：发布相应的事件。
            finishRefresh();
        }

        catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " +
                        "cancelling refresh attempt: " + ex);
            }

            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
            contextRefresh.end();
        }
    }
}
```

这部分不过多阅读，涉及Spring部分太深。