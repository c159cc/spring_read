- [ ] [资源的缓存结构 `Map<Class<?>, Map<Resource, ?>>` ](read.md#defaultresourceloader-%E5%AE%9E%E4%BE%8B%E5%8C%96)
DefaultResourceLoader 
```java
public <T> Map<Resource, T> getResourceCache(Class<T> valueType) {
  return (Map<Resource, T>) this.resourceCaches.computeIfAbsent(valueType, key -> new ConcurrentHashMap<>());
}

// `Function<? super K, ? extends V> mappingFunction` 接收 `key -> new ConcurrentHashMap<>()` 如何理解？
// if ((val = mappingFunction.apply(key)) != null) 

```
- [ ] [AntPathMatcher继承关系](read.md#antpathmatcher)

- [ ] [MutablePropertySources继承关系](read.md#mutablepropertysources)

- [ ] [PropertySourcesPropertyResolver继承关系](read.md#propertysourcepropertyresolver)


- [ ]  [initPropertySources(); 初始化哪些占位符？](refresh.md#abstractapplicationcontext-preparerefresh)

- [ ] Push my commits to GitHub
- [ ] Open a pull request
