
阅读spring core笔记
=
父类初始化流程
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
从这里可以看出，对父类的初始化是**直接**子类触发的  
DefaultResourceLoader  实例化  
```java	
	//创建ProtocolResolver容器
	private final Set<ProtocolResolver> protocolResolvers = new LinkedHashSet<>(4);
	// 创建Resource缓存
	private final Map<Class<?>, Map<Resource, ?>> resourceCaches = new ConcurrentHashMap<>(4);
	
	// DefaultResourceLoader构造器，设置类加载器
	public DefaultResourceLoader() {
		// 默认的资源加载类包含了一个类加载器
		this.classLoader = ClassUtils.getDefaultClassLoader();
	}
```
先执行实例字段的初始化才会执行构造函数

AbstractApplicationContext 构造
```java
	// 初始化日志
	protected final Log logger = LogFactory.getLog(getClass());
	
	// 为上下文设置唯一的id
	private String id = ObjectUtils.identityToString(this);
	
	// 初始化 bean 工厂后置处理器容器
	private final List<BeanFactoryPostProcessor> beanFactoryPostProcessors = new ArrayList<>();
	
	// 当前上下文激活状态，默认为false
	private final AtomicBoolean active = new AtomicBoolean();
	
	// 当前上下文是否关闭，默认未关闭
	private final AtomicBoolean closed = new AtomicBoolean();
	
	// 静态指定的监听器，难道动态指定的监听器不放在这儿？
	private final Set<ApplicationListener<?>> applicationListeners = new LinkedHashSet<>();
	
	public AbstractApplicationContext() {
		// 资源文件解析器  patern表示动态匹配
		this.resourcePatternResolver = getResourcePatternResolver();
	}
	
	// 将位置模式解析为 Resource 实例
	protected ResourcePatternResolver getResourcePatternResolver() {
		// 继承了DefaultResourceLoader 因此this是一个资源加载器
		// PathMatchingResourcePatternResolver 的 getResource方法用来载入资源
		return new PathMatchingResourcePatternResolver(this);
	}
```
将上下文设置为默认的资源加载器  
PathMatchingResourcePatternResolver
```java
	public PathMatchingResourcePatternResolver(ResourceLoader resourceLoader) {
		Assert.notNull(resourceLoader, "ResourceLoader must not be null");
		this.resourceLoader = resourceLoader;
	}
```

AbstractApplicationContext构造 setParent(parent);  
环境整合  
```java
	@Override
	public void setParent(@Nullable ApplicationContext parent) {
		this.parent = parent;
		if (parent != null) {
			Environment parentEnvironment = parent.getEnvironment();
			if (parentEnvironment instanceof ConfigurableEnvironment) {
				getEnvironment().merge((ConfigurableEnvironment) parentEnvironment);
			}
		}
	}
```

AbstractRefreshableApplicationContext  
```java
	// 实例化beanFactory监视器
	private final Object beanFactoryMonitor = new Object();
```

AbstractRefreshableConfigApplicationContext
```java
	private boolean setIdCalled = false;
```

AbstractXmlApplicationContext
```java
	private boolean validating = true;
```

设置资源路径
-
ClassPathXmlApplicationContext 构造函数内 setConfigLocations(configLocations);

AbstractRefreshableConfigApplicationContext  
```java
	public void setConfigLocations(@Nullable String... locations) {
		if (locations != null) {
			Assert.noNullElements(locations, "Config locations must not be null");
			this.configLocations = new String[locations.length];
			for (int i = 0; i < locations.length; i++) {
				this.configLocations[i] = resolvePath(locations[i]).trim();
			}
		}
		else {
			this.configLocations = null;
		}
	}
	
	// 替换资源占位符，比如classpath
	protected String resolvePath(String path) {
		return getEnvironment().resolveRequiredPlaceholders(path);
	}
```
AbstractApplicationContext 获取环境，如果没有，创建一个标准环境
```java
	public ConfigurableEnvironment getEnvironment() {
		if (this.environment == null) {
			this.environment = createEnvironment();
		}
		return this.environment;
	}
	protected ConfigurableEnvironment createEnvironment() {
		return new StandardEnvironment();
	}
```
环境结构图
![StandardEnvironment](https://github.com/c159cc/spring_read/blob/master/images/StandardEnvironment.png)
StandardEnvironment extends AbstractEnvironment
AbstractEnvironment  
```java
	// 设置日志
	protected final Log logger = LogFactory.getLog(getClass());
	// profile容器初始化
	private final Set<String> activeProfiles = new LinkedHashSet<>();
	// 设置默认的profile
	private final Set<String> defaultProfiles = new LinkedHashSet<>(getReservedDefaultProfiles());
	// 创建propertysource
	private final MutablePropertySources propertySources = new MutablePropertySources();
	// 实例化sourceProperty解析器
	private final ConfigurablePropertyResolver propertyResolver = new PropertySourcesPropertyResolver(this.propertySources);
	public AbstractEnvironment() {
		customizePropertySources(this.propertySources);
	}
```

![MutablePropertySources](https://github.com/c159cc/spring_read/blob/master/images/MutablePropertySources.png)
MutablePropertySources
```java
	// 初始化sourceList
	private final List<PropertySource<?>> propertySourceList = new CopyOnWriteArrayList<>();
	
```

![PropertySourcesPropertyResolver](https://github.com/c159cc/spring_read/blob/master/images/PropertySourcesPropertyResolver.png)


AbstractPropertyResolver
```java
	protected final Log logger = LogFactory.getLog(getClass());
	// 嵌套的占位符默认不忽略
	private boolean ignoreUnresolvableNestedPlaceholders = false;
	// ${
	private String placeholderPrefix = SystemPropertyUtils.PLACEHOLDER_PREFIX;
	// }
	private String placeholderSuffix = SystemPropertyUtils.PLACEHOLDER_SUFFIX;
	// :
	@Nullable
	private String valueSeparator = SystemPropertyUtils.VALUE_SEPARATOR;

	private final Set<String> requiredProperties = new LinkedHashSet<>();
```
PropertySourcesPropertyResolver
```java
	public PropertySourcesPropertyResolver(@Nullable PropertySources propertySources) {
		this.propertySources = propertySources;
	}
```

StandardEnvironment
```java
	protected void customizePropertySources(MutablePropertySources propertySources) {
		propertySources.addLast(new MapPropertySource(SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME, getSystemProperties()));
		propertySources.addLast(new SystemEnvironmentPropertySource(SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME, getSystemEnvironment()));
	}
```

![MapPropertySource](https://github.com/c159cc/spring_read/blob/master/images/MapPropertySource.png)
MapPropertySource
```java
	public MapPropertySource(String name, Map<String, Object> source) {
		super(name, source);
	}	
```

EnumerablePropertySource
```java
	public EnumerablePropertySource(String name, T source) {
		super(name, source);
	}
```
PropertySource
```java
	protected final Log logger = LogFactory.getLog(getClass());

	public PropertySource(String name, T source) {
		Assert.hasText(name, "Property source name must contain at least one character");
		Assert.notNull(source, "Property source must not be null");
		this.name = name;
		this.source = source;
	}
```
SystemEnvironmentPropertySource
![SystemEnvironmentPropertySource](https://github.com/c159cc/spring_read/blob/master/images/SystemEnvironmentPropertySource.png)
```java
	public SystemEnvironmentPropertySource(String name, Map<String, Object> source) {
		super(name, source);
	}
```


AbstractEnvironment
```java
	public String resolveRequiredPlaceholders(String text) throws IllegalArgumentException {
		return this.propertyResolver.resolveRequiredPlaceholders(text);
	}
```

AbstractPropertyResolver
```java
	public String resolveRequiredPlaceholders(String text) throws IllegalArgumentException {
		if (this.strictHelper == null) {
			this.strictHelper = createPlaceholderHelper(false);
		}
		return doResolvePlaceholders(text, this.strictHelper);
	}
	private PropertyPlaceholderHelper createPlaceholderHelper(boolean ignoreUnresolvablePlaceholders) {
		return new PropertyPlaceholderHelper(this.placeholderPrefix, this.placeholderSuffix,
				this.valueSeparator, ignoreUnresolvablePlaceholders);
	}
	private String doResolvePlaceholders(String text, PropertyPlaceholderHelper helper) {
		return helper.replacePlaceholders(text, this::getPropertyAsRawString);
	}
```
PropertyPlaceholderHelper
```java
	public PropertyPlaceholderHelper(String placeholderPrefix, String placeholderSuffix,
			@Nullable String valueSeparator, boolean ignoreUnresolvablePlaceholders) {

		Assert.notNull(placeholderPrefix, "'placeholderPrefix' must not be null");
		Assert.notNull(placeholderSuffix, "'placeholderSuffix' must not be null");
		this.placeholderPrefix = placeholderPrefix;
		this.placeholderSuffix = placeholderSuffix;
		String simplePrefixForSuffix = wellKnownSimplePrefixes.get(this.placeholderSuffix);
		if (simplePrefixForSuffix != null && this.placeholderPrefix.endsWith(simplePrefixForSuffix)) {
			this.simplePrefix = simplePrefixForSuffix;
		}
		else {
			this.simplePrefix = this.placeholderPrefix;
		}
		this.valueSeparator = valueSeparator;
		this.ignoreUnresolvablePlaceholders = ignoreUnresolvablePlaceholders;
	}
	public String replacePlaceholders(String value, PlaceholderResolver placeholderResolver) {
		Assert.notNull(value, "'value' must not be null");
		return parseStringValue(value, placeholderResolver, new HashSet<>());
	}
```
