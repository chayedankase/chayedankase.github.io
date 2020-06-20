---
layout: post
title: SpringBoot-缓存
categories: [spring]
description: springboot对缓存的操作
keywords: spring
---



# SpringBoot-缓存

> 对于服务器端而言，缓存分为：
>
> 1. **服务器本地缓存** -  本地缓存是一级缓存，位于服务器本机的内存中，直接从内存中读取数据，因而访问速度最快。
> 2. **分布式缓存** -  本机缓存中没有的数据会去查分布式缓存，在分布式缓存中查询到结果后会将结果直接放置到本地缓存中。分布式缓存主要使用NOSQL数据库实现，常用的NOSQL数据库有 redis、Memcached、MongDB等。
> 3. **数据库缓存** - 数据库会将相同的查询语句结果进行缓存，重复执行时将从缓存中获取结果。

## 1、缓存的作用

> 使用缓存可以极大的提升用户体验，对于重复信息的查询效率提升较高。查询数据时先再缓存中查找，如果没有才去查数据库，数据库中查询到后在存储到缓存中，二次查询时将不必再访问数据库。比如网站首页的信息缓存，商品信息的缓存。 一些临时信息的存储也可以使用缓存，比如验证码等。

## 2、缓存的规范JSR-107

> Java Caching定义了5个核心接口，分别是CachingProvider, CacheManager, Cache, Entry 和 Expiry。
>
> - **CachingProvider**：定义了创建、配置、获取、管理和控制多个CacheManager。一个应用可以在运行期访问多个CachingProvider。
> - **CacheManager**：定义了创建、配置、获取、管理和控制多个唯一命名的Cache，这些Cache存在于CacheManager的上下文中。一个CacheManager仅被一个CachingProvider所拥有。
> - **Cache**：是一个类似Map的数据结构并临时存储以Key为索引的值。一个Cache仅被一个CacheManager所拥有。
> - **Entry**：是一个存储在Cache中的key-value对。
> - **Expiry** ：每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期的状态。一旦过期，条目将不可访问、更新和删除。缓存有效期可以通过ExpiryPolicy设置。



![cache]({{site.url}}imgs\cache\cache1.png)

> 一个应用可以访问多个CachingProvider（缓存提供者）、缓存提供者管理多个CacheManager(缓存管理者)、缓存管理者又管理了Cache(缓存)，一般不同的缓存存储着不同的对象缓存，比如 员工缓存，对象缓存等。



> JSR107实现较为复杂，因而spring进行了缓存抽象，包含了JSR107的注解，使用简便。

## 3、Spring缓存抽象

> spring从3.1版本开始定义了 Cache及CacheManager接口来统一不同的缓存技术并支持使用JSR107注解来简化开发。

### 3.1 几个重要的概念及缓存注解

| Cache          | 缓存接口，定义缓存操作。实现有：RedisCache、EhCacheCache、ConcurrentMapCache等 |
| -------------- | ------------------------------------------------------------ |
| CacheManager   | 缓存管理器，管理各种缓存组件（Cache）。                      |
| @Cacheable     | 主要针对方法配置，能够根据方法的参数对其返回值进行缓存。     |
| @CacheEvict    | 清空缓存                                                     |
| @CachePut      | 保证方法被调用，又希望结果被缓存。一般用于缓存更新。         |
| @EnableCaching | 开启基于注解的缓存                                           |
| keyGenerator   | 缓存数据时key生成策略                                        |
| serialize      | 缓存数据时value序列化策略                                    |

> 缓存相当于一堆Map,一个CacheManager就是一个类，比如RedisCacheManager；而Cache就是类中的map,一个CacheManager中可以有多个cache,每个cache都有自己唯一的名字。相当于一个类中有多个map，每个map有自己的名字。

#### 3.1.1 、CacheManager

> CacheManager管理多个Cache组件，对cache的真正CRUD操作在组件中，每个缓存（cache）组件都有一个自己唯一的名字。

#### 3.1.2、 @Cacheable

> Cacheable注解能够将方法的执行结果进行缓存，一般以方法的参数为key,下次使用同样的参数调用方法时直接从缓存中获取，不重复执行方法。

##### 3.1.2.1、 属性说明

> - **cacheNames/value **:  指定缓存组件的名字；将方法的结果放入那个缓存组件中，可以同时放多个。
> - **key** : 缓存数据的key;可以使用SPEL表达式 @Cacheable(value=”testcache”,key=”#userName”)；也可以从通过#root来获取，详见下表格。
> - **keyGenerator** : key的生成器，通过何种方式来生成key;与key属性任选一个使用。
> - **condition**： 缓存的条件，可以为空，使用SPEL表达式编写，为true时执行该缓存注解。@Cacheable(value=”testcache”,condition=”#userName.length()>2”)
> - **CacheManager** ：指定缓存管理器，确定从哪个缓存管理器中获取缓存，一个系统中可能同时使用Redis和其他缓存，不同的缓存通过缓存管理器来区分。与cacheResolver任选其一使用。
> - **unless** : 否定缓存，与condition刚好相反，为true时不进行缓存。unless属性可以使用SPEL表达式获取方法运行结果后进行判断，比如  #result==null 当方法的返回值为null时不进行缓存。
> - **sync** : 是否使用异步模式。

缓存的SPEL表达式

|    名字     |    位置     |          描述          |       示例        |
| :---------: | :---------: | :--------------------: | :---------------: |
| methodName  | root object |   当前被调用的方法名   | #root.methodName  |
|   method    | root object |    当前被调用的方法    | #root.method.name |
|   target    | root object |  当前被调用的目标对象  |   #root.target    |
| targetClass | root object | 当前被调用的目标对象类 | #root.targetClass |
|             | root object |                        |                   |
|             | root object |                        |                   |
|             | root object |                        |                   |

##### 3.1.2.2、 运行原理

>自动配置类 ：CacheAutoConfiguration,通过CacheConfigurationImportSelector.class来加载各个缓存的配置类。

```java
@Configuration
@ConditionalOnClass(CacheManager.class)
@ConditionalOnBean(CacheAspectSupport.class)
@ConditionalOnMissingBean(value = CacheManager.class, name = "cacheResolver")
@EnableConfigurationProperties(CacheProperties.class)
@AutoConfigureAfter({ CouchbaseAutoConfiguration.class, HazelcastAutoConfiguration.class,
		HibernateJpaAutoConfiguration.class, RedisAutoConfiguration.class })
@Import(CacheConfigurationImportSelector.class)
public class CacheAutoConfiguration {
    。。。。
}
```

> 缓存配置类 ：通过 selectImports方法加载了如下9种缓存配置类（springboot 2.05版本），每个缓存都会在容器中注册一个自己的 cacheManager 并且缓存配置类的生效条件是 @ConditionalOnMissingBean(CacheManager.class)，因而只会有有个缓存配置类生效，检查是否生效的顺序如下 :
>
> ​		0 = "org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration"
> ​		1 = "org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration"
> ​		2 = "org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration"
> ​		3 = "org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration"
> ​		4 = "org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration"
> ​		5 = "org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration"
> ​		6 = "org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration"
> ​		7 = "org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration"
> ​		8 = "org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration"
> ​		9 = "org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration"	

```java
static class CacheConfigurationImportSelector implements ImportSelector {

		@Override
		public String[] selectImports(AnnotationMetadata importingClassMetadata) {
			CacheType[] types = CacheType.values();
			String[] imports = new String[types.length];
			for (int i = 0; i < types.length; i++) {
				imports[i] = CacheConfigurations.getConfigurationClass(types[i]);
			}
			return imports;
		}

	}
```

> 默认那个配置类会生效，设置 配置文件 debug=true后查看容器注入的CacheConfiguration为SimpleCacheConfiguration，因而默认生效的就是 SimpleCacheConfiguration配置。它给容器中放入了一个 ConcurrentMapCacheManager,这个对象实现了 CacheManager接口。

```java
@Bean
	public ConcurrentMapCacheManager cacheManager() {
		ConcurrentMapCacheManager cacheManager = new ConcurrentMapCacheManager();
		List<String> cacheNames = this.cacheProperties.getCacheNames();
		if (!cacheNames.isEmpty()) {
			cacheManager.setCacheNames(cacheNames);
		}
		return this.customizerInvoker.customize(cacheManager);
	}
```

> CacheManager可以通过cache的名字来获取对应cache.调用getCache（String name）方法。
>
> 如果获取不到则会创建缓存通过createConcurrentMapCache(name)，为了防止重复创建的并发问题，
>
> 此处使用了双重验证来防止并发造成的重复创建。

```java
@Override
	@Nullable
	public Cache getCache(String name) {
        //从map中获取缓存
		Cache cache = this.cacheMap.get(name);
        //如果没有获取到就给资源加锁，获取到了则直接返回。
		if (cache == null && this.dynamic) {
			synchronized (this.cacheMap) {
                //加锁后再次验证防止加锁前缓存已经被创建
				cache = this.cacheMap.get(name);
				if (cache == null) {
                    //如果仍旧为空，则进行缓存创建
					cache = createConcurrentMapCache(name);
					this.cacheMap.put(name, cache);
				}
			}
		}
		return cache;
	}
```

> 通过 createConcurrentMapCache(name)创建的Cache对象实际为 ConcurrentMapCache对象。

```java
protected Cache createConcurrentMapCache(String name) {
		SerializationDelegate actualSerialization = (isStoreByValue() ? this.serialization : null);
		return new ConcurrentMapCache(name, new ConcurrentHashMap<>(256),
				isAllowNullValues(), actualSerialization);
	}
```

> ConcurrentMapCache对象中包含 ConcurrentMap<Object, Object> store 对象，这就是保存缓存的map对象。

```java
public class ConcurrentMapCache extends AbstractValueAdaptingCache {

	private final String name;
	//缓存存储的真正位置。
	private final ConcurrentMap<Object, Object> store;
    ....
}
```

##### 3.1.2.3、运行流程

>根据如上原理，调用标注有@Cacheable的方法时：
>
>1. 在方法运行之前，首先会先通过CacheManager获取Cache（缓存组件），依据cacheNames。调用的方法为 getCache(String name)；此处的name即为 cacheNames属性。
>
>2. 调用getCache方法实际上是从CacheManager的cacheMap属性中获取对应cacheNames的cache,从如果获取到的Cache为null的话（即第一次从这个缓存组件中获取缓存）则会创建一个Cache对象，维护进CacheManager中（放入cacheMap属性中），最终会返回一个Cache对象。
>
>3. 获取到Cache后会调用lookup(Object key)方法(此处的key为注解的属性key),用来获取缓存的结果。实际上是从Cache对象的store（map对象）属性中获取。
>
>   如果key属性为默认的话会通过某种策略进行生成；默认使用**keyGenerator**来生成key(该接口默认实现类为SimpleKeyGenerator) 如果没有参数 key = new SimpleKey，如果有1个参数则key=这个参数，如果有多个参数 key = new SimpleKey(params);
>
>4. 如果没有获取到缓存，则会调用目标方法。
>
>5. 从目标方法获取到结果后会执行Cache的put方法来将结果存入缓存中。
>
>**总结：运行标注有@Cacheable的方法前会先去缓存中查找，如果没有找到才会调用目标方法，并将目标方法的结果放入缓存中。如果在缓存中查找到了，则不会直接方法，会将缓存中的结果直接返回。**

##### 3.1.2.4、自定义KeyGenerator

> 使用自定义的keyGenerator,用来定义自己的key生成策略。此处自定义的key为方法名加参数。

```java
  /**
     * 配置自己的缓存key生成策略,可以在注解@Cacheable的属性keyGenerator上使用
     */
    @Bean("myKeyGenerator")
    public KeyGenerator keyGenerator(){
        //Lamda表达式写法
        return (target, method, params) -> method.getName() + Arrays.asList(params).toString();
        //正常写法
        return new KeyGenerator(){
            @Override
            public Object generate(Object target, Method method, Object... params) {
                return method.getName() + Arrays.asList(params).toString();
            }
        };
    }
```

#### 3.1.3、@CachePut

> 使用该注解标注方法，会既执行方法，又将缓存进行更新。其余参数参考@Cacheable。
>
> 适用场景为数据库的修改操作，在修改数据库内数据的同时将对缓存进行更新，保证查询时使用的缓存与数据库同步。

##### 3.1.3.1、运行流程

> 1. 先执行方法。
> 2. 将方法的返回值存入缓存，此处注意，要实现更新效果，需要设置key与查询的key一致才能正常完成更新缓存的效果。
>
> CachePut由于是先执行方法，再操作缓存，因而key的SPEL表达式中可以引用结果 #result

#### 3.1.4、@CacheEvict

> 使用该注解标注的方法在执行后会将指定的缓存清除掉。
>
> 如下指定将cacheNames = emp的cache中的指定key=id的缓存清除掉。
>
> allEntries表示是否清除掉emp中所有的缓存数据，默认为false。当参数为true时不用指定key，会清除所有。
>
> beforeInvocation表示清除缓存操作在执行方法之前执行。默认为false，是在方法之后执行，如果方法执行出错，则不会清除缓存。

```java
@CacheEvict(value = "emp",key = "#id",allEntries = false，beforeInvocation = false)	
```

#### 3.1.5、@Caching

> 复杂缓存，此注解可以进行较为复杂的缓存操作,如下示例，同时以电话号码、id、身份证号为key对员工对象进行了缓存。复杂注解中只要使用了put，则目标方法一定会执行。

```java
  @Caching(
            cacheable = {
                    @Cacheable(value = "emp",key = "#callPhone")
            },
            put = {
                    @CachePut(value = "emp",key = "#result.id"),
                    @CachePut(value = "emp",key = "#result.idCard")
            }
    )
```

#### 3.1.6、@CacheConfig

> 此注解为配置缓存注解，标注在类上，表示此类中的所有缓存注解的属性都适用这个注解内的配置

```java
@CacheConfig(cacheNames = "",keyGenerator = "",cacheManager = "")
```

## 4、Redis缓存

> 默认的CacheManager为 ConcurrentMapCacheManager，其中管理的Cache是将数据保存在ConcurrentMap里，在实际开发中经常使用缓存中间件比如：redis,memcached、ehcache等。
>
> 如何引入这些缓存中间件呢，在缓存运行原理中可以看到，缓存自动配置引入了多种CacheConfiguration，而默认生效的是SimpleCacheConfiguration,这里我们以Redis为例看看如何使其他的缓存配置生效。

### 4.1、引入springboot-redis相关包

> 在使用springboot时我们只需要引入对应的redis启动器即可。（内部引入了jedis客户端）

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

```java
@Configuration
//生效条件：存在org.springframework.data.redis.connection.RedisConnectionFactory类;
@ConditionalOnClass(RedisConnectionFactory.class)
//在此配置类生效后引入RedisAutoConfiguration.class
@AutoConfigureAfter(RedisAutoConfiguration.class)
//生效条件：容器中有RedisConnectionFactory.class的对象
@ConditionalOnBean(RedisConnectionFactory.class)
//生效条件：容器中不存在org.springframework.cache.CacheManager的对象;
@ConditionalOnMissingBean(CacheManager.class)
@Conditional(CacheCondition.class)
class RedisCacheConfiguration {
	。。。。
}
```

> 如上，只有我们导入了Redis与spring-data的相关包并且在容器中维护了相应的对象后才会开启redis缓存配置。并且在redis缓存配置创建 RedisCacheManager之前容器中存在了其他的CacheManager,则缓存配置仍旧不会生效。 

### 4.2、配置redis

```yaml
  redis:
    host: XXX.XXX.XXX.XX(redis服务器地址)
    password: xxxxxx (密码)
    port: （端口号，默认为6379）
```

### 4.3 redis的使用

> redis的自动配置类RedisAutoConfiguration.class 在容器中维护了 RedisTemplate、StringRedisTemplate
>
> 对象，它们是操作redis的模板对象。

```java
@Bean
	@ConditionalOnMissingBean(name = "redisTemplate")
	public RedisTemplate<Object, Object> redisTemplate(
			RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
		RedisTemplate<Object, Object> template = new RedisTemplate<>();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

	@Bean
	@ConditionalOnMissingBean
	public StringRedisTemplate stringRedisTemplate(
			RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
		StringRedisTemplate template = new StringRedisTemplate();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}
```

#### 4.3.1、StringRedisTemplate

> StringRedisTemplate是针对k-v都是String的数据操作。
>
> 它可以操作的对象类型有 string(字符串),list（列表）,set（集合）,hash（散列）,zset（有序集合）

```java
  @Autowired
    private StringRedisTemplate stringRedisTemplate;

	@Test
	public void contextLoads() {
	    //存入一个数据
        stringRedisTemplate.opsForValue().append("emp", "1");
        //存入一个list
        stringRedisTemplate.opsForList().leftPush("list", "a");
        stringRedisTemplate.opsForList().leftPush("list", "b");
        stringRedisTemplate.opsForList().leftPush("list", "c");
	}
```

#### 4.3.2、RedisTemplate

> RedisTemplate基本用法与StringRedisTemplate一致，是针对k-v都为对象的数据操作。
>
> 在存储对象时，该对象必须实现序列化接口。因为RedisTemplate在保存对象的时候默认使用jdk序列化机制，将序列化后的数据进行保存。
>
> 保存后的对象在redis直接查看时显示的是乱码，只能反序列化后查看。这样不利于查看，一般转化为json数据进行存储。

##### 4.3.2.1 将对象以json格式存储

> 转化的办法有两种：
>
> 1. 自己将对象转化为json串后进行存储。
> 2. 改变RedisTemplate的默认序列化规则。

> RedisAutoConfiguration 在注入 RedisTemplate时代码如下

```java
	@Bean
	@ConditionalOnMissingBean(name = "redisTemplate")
	public RedisTemplate<Object, Object> redisTemplate(
			RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        //此处直接new了一个RedisTemplate
		RedisTemplate<Object, Object> template = new RedisTemplate<>();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}
```

> 我们要自定义序列化规则，则可以仿照上述方式，new RedisTemplate,并设置自己的序列化规则后将对象放入容器中以供自己使用。
>
> 如下代码创建的redisTemplate替换了原有的。这里的序列化规则使用了 阿里的fastjson中的GenericFastJsonRedisSerializer。

```java
	@Bean
    public RedisTemplate<Object, String> redisTemplate(
            RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        //创建模板对象
        RedisTemplate<Object, String> redisTemplate = new RedisTemplate<>();
        //创建序列化规则
        GenericFastJsonRedisSerializer serializer = new GenericFastJsonRedisSerializer();
        //设置默认的序列化规则
        redisTemplate.setDefaultSerializer(serializer);
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        return redisTemplate;
    }
```

#### 4.3.3、基于注解的Redis

> 上述使用Template进行缓存，需要在缓存数据时手动加入缓存，现在介绍如何利用注解完成redis的缓存操作。
>
> 其实在我们配置完redis后，使用JSR107注解就已经可以完成缓存的操作了，但是缓存对象时会出现与RedisTemplate一样的问题，序列化规则为默认的jdk。虽然我们重新注入了Preisolated，但是好像基于注解的缓存并没有使用我们新添加的序列化规则。我们来看看基于注解的序列化是怎么回事。
>
> <!--(注：此处为springboot2.0,在1.0版本配置完RedisTemplate后注解就可以使用了。)-->

>从前面的 SimpleCacheConfiguration运行原理我们知道基于注解的缓存操作是通过CacheManager过去Cache,再用Cache来操作缓存的（其实是操作内部的map）。同样我们就从RedisCacheCongifuration创建的CacheManager来开始。由于springboot2.0对RedisCacheManager进行了重写，不容易理解，我们先来看看1.0版本的RedisCacheManager。

```java
    @Bean
    public RedisCacheManager cacheManager(RedisTemplate<Object, Object> redisTemplate) {
        //创建CacheManager时传入了redisTemplate，序列化时就使用了这个template里的序列化规则
        //这个redisTemplate是从容器中获取的。如果我们自定义了并且名称就叫redisTemplate,这里
        //就会使用我们自己的序列化规则。
        RedisCacheManager cacheManager = new RedisCacheManager(redisTemplate);
        cacheManager.setUsePrefix(true);
        List<String> cacheNames = this.cacheProperties.getCacheNames();
        if (!cacheNames.isEmpty()) {
            cacheManager.setCacheNames(cacheNames);
        }
        return (RedisCacheManager)this.customizerInvoker.customize(cacheManager);
    }
```

> 可以看出在spring1.0版本，就是使用了redisTemplate的序列化规则。因而当我们配置好后就可以使用注解进行缓存管理了，而2.0进行了改版请看下面代码

```java
	@Bean
	public RedisCacheManager cacheManager(RedisConnectionFactory redisConnectionFactory,
			ResourceLoader resourceLoader) {
        //创建一个builder
		RedisCacheManagerBuilder builder = RedisCacheManager
				.builder(redisConnectionFactory)
            	//设置默认的缓存配置
				.cacheDefaults(determineConfiguration(resourceLoader.getClassLoader()));
		List<String> cacheNames = this.cacheProperties.getCacheNames();
		if (!cacheNames.isEmpty()) {
			builder.initialCacheNames(new LinkedHashSet<>(cacheNames));
		}
		return this.customizerInvoker.customize(builder.build());	
	}

/*
* 此方法获取了一个默认的RedisCacheConfiguration 缓存配置
*/
private org.springframework.data.redis.cache.RedisCacheConfiguration determineConfiguration(
			ClassLoader classLoader) {
		if (this.redisCacheConfiguration != null) {
			return this.redisCacheConfiguration;
		}
    	//从配置文件中获取redis的配置信息
		Redis redisProperties = this.cacheProperties.getRedis();
		org.springframework.data.redis.cache.RedisCacheConfiguration config = 	org.springframework.data.redis.cache.RedisCacheConfiguration
				.defaultCacheConfig();
    	//设置序列化解析规则
		config = config.serializeValuesWith(SerializationPair
				.fromSerializer(new JdkSerializationRedisSerializer(classLoader)));
    	//如果设置了过期时间就配置上
		if (redisProperties.getTimeToLive() != null) {
			config = config.entryTtl(redisProperties.getTimeToLive());
		}
        //如果设置了前缀就配置上
		if (redisProperties.getKeyPrefix() != null) {
			config = config.prefixKeysWith(redisProperties.getKeyPrefix());
		}
    	//默认存储空值
		if (!redisProperties.isCacheNullValues()) {
			config = config.disableCachingNullValues();
		}
    	//配置是否使用前缀，默认使用
		if (!redisProperties.isUseKeyPrefix()) {
			config = config.disableKeyPrefix();
		}
		return config;
	}
```

> 从上面代码可以看到默认的解析规则为 new JdkSerializationRedisSerializer(classLoader)，想变更解析规则，我们只能通过重新创建CacheManager设置新的序列化规则。
>
> RedisCacheManager是通过内部类 RedisCacheManagerBuilder对象的build()方法创建的。创建前需要传入RedisConnectionFactory和RedisCacheConfiguration对象。我们只需要自定义RedisCacheConfiguration对象即可。

```java
@Bean
public RedisCacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
    //创建序列化规则
    GenericFastJsonRedisSerializer serializer = new GenericFastJsonRedisSerializer()
    //创建默认的缓存配置类，通过链式调用设置序列化规则和缓存过期时间
 RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()               .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(serializer))
        .entryTtl(Duration.ofHours(1));//设置默认过期时间
    return RedisCacheManager
        .builder(redisConnectionFactory).cacheDefaults(config).build();
}
```

未完待续。。。。。。。





