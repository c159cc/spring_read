
#### BeanDefinitionParserDelegate
```java
public static final String MULTI_VALUE_ATTRIBUTE_DELIMITERS = ",; ";
 
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, @Nullable BeanDefinition containingBean) {
	String id = ele.getAttribute(ID_ATTRIBUTE);
	String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);

	List<String> aliases = new ArrayList<>();
	if (StringUtils.hasLength(nameAttr)) {
		// 根据逗号，分号，空格 拆分名字
		String[] nameArr = StringUtils.tokenizeToStringArray(nameAttr, MULTI_VALUE_ATTRIBUTE_DELIMITERS);
		// 将拆分后的名字存放到aliases中
		aliases.addAll(Arrays.asList(nameArr));
	}
  
	String beanName = id;
	if (!StringUtils.hasText(beanName) && !aliases.isEmpty()) {
		// 将aliases的第一个元素设置为beanName
		beanName = aliases.remove(0);
		if (logger.isTraceEnabled()) {
			logger.trace("No XML 'id' specified - using '" + beanName +
					"' as bean name and " + aliases + " as aliases");
		}
	}
	// 上面的代码，如果存在id， 那么beanName=id，否则aliases的第一个元素为beanName

	if (containingBean == null) {
		checkNameUniqueness(beanName, aliases, ele);
	}

	AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);
	if (beanDefinition != null) {
		if (!StringUtils.hasText(beanName)) {
			try {
				if (containingBean != null) {
					beanName = BeanDefinitionReaderUtils.generateBeanName(
							beanDefinition, this.readerContext.getRegistry(), true);
				}
				else {
					beanName = this.readerContext.generateBeanName(beanDefinition);
					// Register an alias for the plain bean class name, if still possible,
					// if the generator returned the class name plus a suffix.
					// This is expected for Spring 1.2/2.0 backwards compatibility.
					String beanClassName = beanDefinition.getBeanClassName();
					if (beanClassName != null &&
							beanName.startsWith(beanClassName) && beanName.length() > beanClassName.length() &&
							!this.readerContext.getRegistry().isBeanNameInUse(beanClassName)) {
						aliases.add(beanClassName);
					}
				}
				if (logger.isTraceEnabled()) {
					logger.trace("Neither XML 'id' nor 'name' specified - " +
							"using generated bean name [" + beanName + "]");
				}
			}
			catch (Exception ex) {
				error(ex.getMessage(), ele);
				return null;
			}
		}
		String[] aliasesArray = StringUtils.toStringArray(aliases);
		return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
	}

	return null;
}
```


```java
// 在已经使用过的名字中找到 aliases,如果找到其中任何一个，就报错
protected void checkNameUniqueness(String beanName, List<String> aliases, Element beanElement) {
  String foundName = null;

  if (StringUtils.hasText(beanName) && this.usedNames.contains(beanName)) {
    foundName = beanName;
  }
  if (foundName == null) {
    // 遍历aliases集合，this.usedNames如果包含集合中的任何一个元素，返回该元素
    foundName = CollectionUtils.findFirstMatch(this.usedNames, aliases);
  }
  if (foundName != null) {
    error("Bean name '" + foundName + "' is already used in this <beans> element", beanElement);
  }

  this.usedNames.add(beanName);
  this.usedNames.addAll(aliases);
}
```

new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
```java
public BeanDefinitionHolder(BeanDefinition beanDefinition, String beanName, @Nullable String[] aliases) {
  Assert.notNull(beanDefinition, "BeanDefinition must not be null");
  Assert.notNull(beanName, "Bean name must not be null");
  this.beanDefinition = beanDefinition;
  this.beanName = beanName;
  this.aliases = aliases;
}
```
所以一个BeanDefinitionHolder中包含了一个beanName aliases beandefinition
之后会将它们注册到SimpleAliasRegistry中
```java
// BeanDefinitionReaderUtils
public static void registerBeanDefinition(
    BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
    throws BeanDefinitionStoreException {

  // Register bean definition under primary name.
  String beanName = definitionHolder.getBeanName();
  registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

  // Register aliases for bean name, if any.
  String[] aliases = definitionHolder.getAliases();
  if (aliases != null) {
    for (String alias : aliases) {
      registry.registerAlias(beanName, alias);
    }
  }
}

// SimpleAliasRegistry
public void registerAlias(String name, String alias) {
  Assert.hasText(name, "'name' must not be empty");
  Assert.hasText(alias, "'alias' must not be empty");
  synchronized (this.aliasMap) {
    if (alias.equals(name)) {
      this.aliasMap.remove(alias);
      if (logger.isDebugEnabled()) {
        logger.debug("Alias definition '" + alias + "' ignored since it points to same name");
      }
    }
    else {
      String registeredName = this.aliasMap.get(alias);
      if (registeredName != null) {
        if (registeredName.equals(name)) {
          // An existing alias - no need to re-register
          return;
        }
        if (!allowAliasOverriding()) {
          throw new IllegalStateException("Cannot define alias '" + alias + "' for name '" +
              name + "': It is already registered for name '" + registeredName + "'.");
        }
        if (logger.isDebugEnabled()) {
          logger.debug("Overriding alias '" + alias + "' definition for registered name '" +
              registeredName + "' with new target name '" + name + "'");
        }
      }
      checkForAliasCircle(name, alias);
      this.aliasMap.put(alias, name);
      if (logger.isTraceEnabled()) {
        logger.trace("Alias definition '" + alias + "' registered for name '" + name + "'");
      }
    }
  }
}
```

```java
//DefaultListableBeanFactory
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
    throws BeanDefinitionStoreException {

    ...
    this.beanDefinitionMap.put(beanName, beanDefinition);
    this.beanDefinitionNames.add(beanName);
    this.manualSingletonNames.remove(beanName);
    ...

}
```

```java
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
    @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

  final String beanName = transformedBeanName(name);
  ...
}

// AbstractBeanFactory
protected String transformedBeanName(String name) {
  return canonicalName(BeanFactoryUtils.transformedBeanName(name));
}

// SimpleAliasRegistry
public String canonicalName(String name) {
  String canonicalName = name;
  // Handle aliasing...
  String resolvedName;
  do {
    resolvedName = this.aliasMap.get(canonicalName);
    if (resolvedName != null) {
      canonicalName = resolvedName;
    }
  }
  while (resolvedName != null);
  return canonicalName;
}
```
