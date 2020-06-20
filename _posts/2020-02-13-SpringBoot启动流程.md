---
layout: post
title: springboot启动流程
categories: [spring]
description: springboot容器启动流程
keywords: springboot
---

# springboot启动流程

### 1. 内置容器启动

```java
@SpringBootApplication
public class SpringbootdemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootdemoApplication.class, args);
    }
}
```

springboot启动类入口方法如上，调用了 SpringApplication.run的静态方法run,并将启动类传了进去,如下所示。

```java
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
    return run(new Class<?>[] { primarySource }, args);
}
```

又调用了重载的run方法。

```java
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    return new SpringApplication(primarySources).run(args);
}
```

在重载的run中通过传递进来的启动类，创建了一个SpringApplication对象，并调用了对象的run方法，下面我们先看看spring应用对象的创建

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    //配置ApplicationContextInitializer，
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

如上的getSpringFactoriesInstances方法，根据参数传递的类型来获取实例。

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
    ClassLoader classLoader = getClassLoader();
    //SpringFactoriesLoader.loadFactoryNames 通过加载器加载工厂名称，
    Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
    //根据名称反射创建响应对象。
    List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
    //排序
    AnnotationAwareOrderComparator.sort(instances);
    return instances;
}
```

通过SpringFactoriesLoader.loadFactoryNames(type, classLoader)方法，又调用 loadSpringFactories(classLoader) 

```java
private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
    MultiValueMap<String, String> result = cache.get(classLoader);
    if (result != null) {
        return result;
    }

    try {
        //通过FACTORIES_RESOURCE_LOCATION来加载资源，
       //FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories"
        Enumeration<URL> urls = (classLoader != null ?
                                 classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
                                 ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
        result = new LinkedMultiValueMap<>();
        while (urls.hasMoreElements()) {
            URL url = urls.nextElement();
            UrlResource resource = new UrlResource(url);
            Properties properties = PropertiesLoaderUtils.loadProperties(resource);
            for (Map.Entry<?, ?> entry : properties.entrySet()) {
                String factoryClassName = ((String) entry.getKey()).trim();
                for (String factoryName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
                    result.add(factoryClassName, factoryName.trim());
                }
            }
        }
        cache.put(classLoader, result);
        return result;
    }
    catch (IOException ex) {
        throw new IllegalArgumentException("Unable to load factories from location [" +
                                           FACTORIES_RESOURCE_LOCATION + "]", ex);
    }
}
```

综上：

​	创建SpringApplication对象时会查找META-INF/spring.factories文件，并将文件内容读取为properties,保存到缓存中。将其中 ApplicationContextInitializer.class、ApplicationListener.class进行实例化，并保存到对象中。

对象SpringApplication创建好后，会调用它的run方法，也就是spring应用的真正运行过程。

