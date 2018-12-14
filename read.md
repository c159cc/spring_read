
阅读spring core笔记
=
图片格式对比
-

![jpg](https://github.com/c159cc/spring_read/blob/master/images/ClassPathXmlApplicationContext.gif)
AbstractApplicationContext 静态代码块
```java
	static {
		// Eagerly load the ContextClosedEvent class to avoid weird classloader issues
		// on application shutdown in WebLogic 8.1. (Reported by Dustin Woods.)
		ContextClosedEvent.class.getName();
	}
```

ClassPathXmlApplicationContext重载的构造函数
```java
	public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
		this(new String[]{configLocation}, true, null);
	}
	
	public ClassPathXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {

		// 动态设置该上下文为资源加载器
		// 将parent上下文中的信息merge到该上下文
		super(parent);
		// 设置资源路径
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();
		}
	}
```

AbstractXmlApplicationContext 构造函数
```java
	public AbstractXmlApplicationContext(@Nullable ApplicationContext parent) {
		super(parent);
	}
```

AbstractRefreshableConfigApplicationContext 构造
```java
	public AbstractRefreshableConfigApplicationContext(@Nullable ApplicationContext parent) {
		super(parent);
	}
```

AbstractRefreshableApplicationContext 构造
```java
	public AbstractRefreshableApplicationContext(@Nullable ApplicationContext parent) {
		super(parent);
	}
```

AbstractApplicationContext构造
```java
	public AbstractApplicationContext(@Nullable ApplicationContext parent) {
		this();
		setParent(parent);
	}
```

this()设置资源解析器
因为 public abstract class AbstractApplicationContext extends DefaultResourceLoader 所以要先执行父类的构造器
```java	
	//创建ProtocolResolver容器
	private final Set<ProtocolResolver> protocolResolvers = new LinkedHashSet<>(4);
	// 创建Resource缓存
	private final Map<Class<?>, Map<Resource, ?>> resourceCaches = new ConcurrentHashMap<>(4);
	
	// DefaultResourceLoader构造器，设置类加载器
	public DefaultResourceLoader() {
		this.classLoader = ClassUtils.getDefaultClassLoader();
	}
```
先执行final字段的初始化才会执行构造函数

AbstractApplicationContext 构造
```java
	public AbstractApplicationContext() {
		// 资源文件解析器  patern表示动态匹配
		this.resourcePatternResolver = getResourcePatternResolver();
	}
```

