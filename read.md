
阅读spring core笔记
=
图片格式对比
-

![jpg](https://github.com/c159cc/spring_read/blob/master/images/ClassPathXmlApplicationContext.gif)
```java
	static {
		// Eagerly load the ContextClosedEvent class to avoid weird classloader issues
		// on application shutdown in WebLogic 8.1. (Reported by Dustin Woods.)
		ContextClosedEvent.class.getName();
	}
```
