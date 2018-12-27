

qa1：factory-method 为什么只能是静态的？
qa2：两个class属性的处理方式有什么不同？
<bean id="exampleBean" class="examples.ExampleBean"/>
<bean id="clientService" class="examples.ClientService"  factory-method="createInstance"/>

#### AbstractAutowireCapableBeanFactory
```java
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
  // Make sure bean class is actually resolved at this point.
  Class<?> beanClass = resolveBeanClass(mbd, beanName);

  if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
    throw new BeanCreationException(mbd.getResourceDescription(), beanName,
        "Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
  }

  Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
  if (instanceSupplier != null) {
    return obtainFromSupplier(instanceSupplier, beanName);
  }
  
  // 判断是否有工厂方法，如果有在这里实例化
  if (mbd.getFactoryMethodName() != null) {
    return instantiateUsingFactoryMethod(beanName, mbd, args);
  }

  // Shortcut when re-creating the same bean...
  boolean resolved = false;
  boolean autowireNecessary = false;
  if (args == null) {
    synchronized (mbd.constructorArgumentLock) {
      if (mbd.resolvedConstructorOrFactoryMethod != null) {
        resolved = true;
        autowireNecessary = mbd.constructorArgumentsResolved;
      }
    }
  }
  if (resolved) {
    if (autowireNecessary) {
      return autowireConstructor(beanName, mbd, null, null);
    } else {
      return instantiateBean(beanName, mbd);
    }
  }

  // Candidate constructors for autowiring?
  Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
  if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
      mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
    return autowireConstructor(beanName, mbd, ctors, args);
  }

  // Preferred constructors for default construction?
  ctors = mbd.getPreferredConstructors();
  if (ctors != null) {
    return autowireConstructor(beanName, mbd, ctors, null);
  }

  // No special handling: simply use no-arg constructor.
  return instantiateBean(beanName, mbd);
}
```

```java
protected BeanWrapper instantiateUsingFactoryMethod(
    String beanName, RootBeanDefinition mbd, @Nullable Object[] explicitArgs) {
  return new ConstructorResolver(this).instantiateUsingFactoryMethod(beanName, mbd, explicitArgs);
}
```


ConstructorResolver
```java
public ConstructorResolver(AbstractAutowireCapableBeanFactory beanFactory) {
  this.beanFactory = beanFactory;
  this.logger = beanFactory.getLogger();
}
```
























