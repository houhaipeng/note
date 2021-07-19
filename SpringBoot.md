# SpringBoot

## 1. springboot启动流程

几个重要的事件回调机制

​	以下配置在META-INF/spring.factories

- ApplicationContextInitializer
- SpringApplicationRunListener

​	以下只需要放在ioc容器中

- ApplicationRunner
- CommandLineRunner

1. 创建`SpringApplication`对象

	```
	public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
	        this.sources = new LinkedHashSet();
	        this.bannerMode = Mode.CONSOLE;
	        this.logStartupInfo = true;
	        this.addCommandLineProperties = true;
	        this.addConversionService = true;
	        this.headless = true;
	        this.registerShutdownHook = true;
	        this.additionalProfiles = new HashSet();
	        this.isCustomEnvironment = false;
	        this.lazyInitialization = false;
	        this.resourceLoader = resourceLoader;
	        Assert.notNull(primarySources, "PrimarySources must not be null");
	        this.primarySources = new LinkedHashSet(Arrays.asList(primarySources));
	        this.webApplicationType = WebApplicationType.deduceFromClasspath();
	//从类路径下找到META-INF/spring.factories配置的所有ApplicationContextInitializer      this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
	//从类路径下找到META-INF/spring.factories配置的所有ApplicationListener this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
					//从多个主配置类中找到有main方法的主配置类
	        this.mainApplicationClass = this.deduceMainApplicationClass();
	    }
	```

2. 执行`SpringApplication`的`run`方法

```
public ConfigurableApplicationContext run(String... args) {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        ConfigurableApplicationContext context = null;
        Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList();
        this.configureHeadlessProperty();
        //获取SpringApplicationRunListeners；从类路径下找到META-INF/spring.factories配置的所有ApplicationContextInitializer
        SpringApplicationRunListeners listeners = this.getRunListeners(args);
        //回调所有获取到的SpringApplicationRunListeners.starting（）方法
        listeners.starting();

        Collection exceptionReporters;
        try {
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            //准备环境，将environment保存到ioc中；而且
            ConfigurableEnvironment environment = this.prepareEnvironment(listeners, applicationArguments);
            this.configureIgnoreBeanInfo(environment);
            Banner printedBanner = this.printBanner(environment);
            //创建ApplicationContext;决定创建web的ioc还是普通的ioc
            context = this.createApplicationContext();
            exceptionReporters = this.getSpringFactoriesInstances(SpringBootExceptionReporter.class, new Class[]{ConfigurableApplicationContext.class}, context);
            //准备上下文环境
            //回调之前保存的所有的ApplicationContextInitializer的initialize方法
            //回调之前保存的所有的SpringApplicationRunListener的contextPrepared方法
            //
            this.prepareContext(context, environment, listeners, applicationArguments, printedBanner);
            //刷新容器；ioc容器初始化的过程
            //扫描，创建，加载所有组件的地方
            this.refreshContext(context);
            //从ioc容器中获取所有的ApplicationRunner和CommandLineRunner进行回调
            //ApplicationRunner先回调，CommandLineRunner再回调
            this.afterRefresh(context, applicationArguments);
            stopWatch.stop();
            if (this.logStartupInfo) {
                (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), stopWatch);
            }

            listeners.started(context);
            this.callRunners(context, applicationArguments);
        } catch (Throwable var10) {
            this.handleRunFailure(context, var10, exceptionReporters, listeners);
            throw new IllegalStateException(var10);
        }

        try {
            listeners.running(context);
            //整个SpringBoot应用启动完成以后返回启动的ioc容器
            return context;
        } catch (Throwable var9) {
            this.handleRunFailure(context, var9, exceptionReporters, (SpringApplicationRunListeners)null);
            throw new IllegalStateException(var9);
        }
    }
```

