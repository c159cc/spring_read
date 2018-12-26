- [ ] [资源的缓存结构 `Map<Class<?>, Map<Resource, ?>>` ](read.md#defaultresourceloader-%E5%AE%9E%E4%BE%8B%E5%8C%96)
DefaultResourceLoader 
```java
public <T> Map<Resource, T> getResourceCache(Class<T> valueType) {
  return (Map<Resource, T>) this.resourceCaches.computeIfAbsent(valueType, key -> new ConcurrentHashMap<>());
}

// `Function<? super K, ? extends V> mappingFunction` 接收 `key -> new ConcurrentHashMap<>()` 如何理解？
// if ((val = mappingFunction.apply(key)) != null) 

```
```
// java8之前。从map中根据key获取value操作可能会有下面的操作
Object key = map.get("key");
if (key == null) {
    key = new Object();
    map.put("key", key);
}

// java8之后。上面的操作可以简化为一行，若key对应的value为空，会将第二个参数的返回值存入并返回
Object key2 = map.computeIfAbsent("key", k -> new Object());

所以对于 computeIfAbsent(valueType, key -> new ConcurrentHashMap<>()); 此处的key 应该是valueType，value就是new ConcurrentHashMap<>())

传递一个 A -> B 意味着什么？
仅仅意味着方法形参为A, 方法返回值为B
提供一个处理参数的方法，参数你来给，结果我来返回

接收一个 A -> B 意味着什么？
收到一个转换器，能对自己的某个属性进行转化

这个结构的含义：一个类型对对应了多个Resource，一个Resoure对应了一个T



```

- [ ] [AntPathMatcher继承关系](read.md#antpathmatcher)
 [查看这里](AntPathMatcher.md)

- [ ] [MutablePropertySources继承关系](read.md#mutablepropertysources)

- [ ] [PropertySourcesPropertyResolver继承关系](read.md#propertysourcepropertyresolver)

- [ ]  [initPropertySources(); 初始化哪些占位符？](refresh.md#abstractapplicationcontext-preparerefresh)

- [ ] Push my commits to GitHub
- [ ] Open a pull request
