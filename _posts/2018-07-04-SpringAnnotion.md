---
layout: post
title: Spring注解
categories: [spring]
description: spring注解的作用
keywords: spring
---



# Spring注解

> 常规的spring配置采用了xml文件配置，对于容器内bean的注册，使用xml配置不利于类型检查。spring另外的一种配置即基于注解的配置。

## 1.配置类

> 添加有@Configuration注解的java类就是配置类，配置类的作用于xml配置文件类似。有相应的注解来对应xml中的标签，并且进行了增强
>
> |      注解      |            标签            |           备注           |
> | :------------: | :------------------------: | :----------------------: |
> |     @Bean      |          < bean >          |         注册组件         |
> | @ComponentScan | < context:component-scan > | 扫描指定包，自动加载组件 |
> | includeFilters | < context:include-filter > |       定制扫描规则       |
> |                |                            |                          |
> |                |                            |                          |
> |                |                            |                          |
> |                |                            |                          |
> |                |                            |                          |
> |                |                            |                          |
>

## 2.ICO

>IOC控制反转，spring采用了容器管理对象，需要对象从容器中获取的方式来管理来管理对象，下面对springIOC的注解进行分类型详解

### 2.1组件注入

>组件注入的方式
>
>1. 通过@Bean注解注入。
>2. 通过@ComponentScan注解，利用包扫描注入。由于包扫描注入需要添加固定的识别标识，因而只能注入自己写的类，外部包的类无法通过包扫描注入
>3. @Import注解注入,能够快速的给容器中导入组件。

#### 2.1.1@Bean

##### 2.1.1.1使用方式

> 注解添加在配置类的方法上，给容器中注册一个bean,类型为返回值的类型，id默认是用方法名作为id，
>
> 作用与<bean>标签一致。

```java
@Bean(value="emp")
public Employee employee(){
        return new Employee();
    }
```

> 如上，给容器中注册一个Employee对象，对象名称默认为方法名即：employee。
>
> @Bean中的value属性可以对对象的名称进行指定，这里指定为emp。
>
> @Bean注解标注的方法的入参如果在容器中有相同的类型，则容器会把对应的对象传递进来。

```java
@Bean
public Car car(){
    return new Car();
}
@Bean(value="emp")
public Employee employee(Car car){
    Employee emp = new Employee();
    emp.setCar(car);
    return emp;
}
```

> 如上，car对象会被自动放入emp对象内。

##### 2.1.1.2 @ scpoe

> 当我们从容器中使用同样的方法获取bean实例时，默认的获取到的总是同一个，即容器中的bean是单实例的。我们可以在@Bean标签的基础上添加@Scope注解来修改这个实例的作用范围。这个注解与<bean  scope >作用相同

```java
	@Bean(value = "emp")
    @Scope(value = "prototype") //默认的value值是singleton单实例
    public Employee emp(){
        return new Employee();
    }
```

> 默认单实例的情况下，在容器启动的时候会将对象创建好，每次获取都是从容器中将对象取出。
>
> prototype多实例的情况下，容器启动时不创建对象，而是在每次获取的时候都创建一个新的对象返回。

##### 2.1.1.3@Lazy

> 单实例的bean是在容器启动的时候创建的，通过@Lazy懒加载注解，我们可以改变单实例对象的创建时机，由启动时创建，改为第一次获取时创建。

```java
	@Lazy
	@Bean(value = "emp")
    public Employee emp(){
        return new Employee();
    }
```



#### 2.1.2@ComponentScan

##### 2.1.2.1 如何使用

> 通常我们给容器中注入组件时，更习惯于使用包扫描的方式，这样更加简洁。通过包扫描会自动将带有@Controller、@Service、@Repository、@Component注解的组件注入容器中。包扫描的方式仅可以将自己写的类注入容器中，外部包的类注入无法通过包扫描注入。

```java
@Configuration
@ComponentScan
public class MainConfig {
}
```

>如上，将@ComponentScan注解添加在配置类上，将默认扫描与配置类同级以及子包中所有标记的类。还可以通过设置注解的 value属性，或者basePackages属性来指定要扫描的包，这两个属性都支持传入多个包位置。

```java
@Configuration
@ComponentScan(value = {"com.pansky.controller","com.pansky.entity"})
public class MainConfig {
}
```

##### 2.1.2.2 扫描规则定制

> 在xml配置中可以在< context:component-scan >标签里添加子标签 <context:exclude-filter>  和 <context:include-filter>来进行扫描规则定制。在注解中也可以使用includeFilters或excludeFilters属性来定制扫描规则。includeFilters扫描时按照规则扫描组件。excludeFilters扫描时按照规则排除组件。

```java
@Configuration
/*表示此类为注解类*/
@ComponentScan(basePackages = {
        "com.pansky"
},includeFilters = {
       //@ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Controller.class, Component.class}) 只扫描Controller及Component注解
       @ComponentScan.Filter(type = FilterType.CUSTOM,classes = MyTypeFilter.class)
        // type = FilterType.ASSIGNABLE_TYPE,classes = XXX.class 按照给定的类型进行过滤
        // type = FilterType.REGEX,classes = XXX.class 按照正则进行过滤
        // type = FilterType.CUSTOM,classes = XXX.class 按照自定义规则进行过滤
},useDefaultFilters = false

/*excludeFilters = {
        //过滤注解的写法
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Controller.class, Service.class})
}*/)
public class MainConfig {
}
```

> 定制扫描规则的两个Filters属性中填写的是Filte注解数组，@Filter注解是@ComponentScan的内部注解，这个注解需要传入type过滤的标识，默认type为注解过滤，classes属性表示过滤的类型。通过对Filter的配置，就可以实现扫描规则的定制。
>
> Filter中type常用的有 FilterType.ANNOTATION根据注解过滤，FilterType.ASSIGNABLE_TYPE根据类型过滤，还有一种FilterType.CUSTOM根据自定义要求过滤。其余过滤类型基本不用。
>
> 在使用FilterType.CUSTOM时需要自定义TypeFilter，实现TypeFilter接口

```java
public class MyTypeFilter implements TypeFilter{

    /**
     * 自定义过滤规则必须实现TypeFilter的match方法，返回true则匹配成功
     * @param metadataReader 读取到的当前正在扫描的类的元数据
     * @param metadataReaderFactory 读取工厂元数据，可以获取到其他类的信息
     * @return
     * @throws IOException
     */
    @Override
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();//获取当前类的注解信息
        ClassMetadata classMetadata = metadataReader.getClassMetadata();//获取当前类的信息
        Resource resource = metadataReader.getResource();//获取当前的类的资源（类路径等）
        System.out.println(classMetadata.getClassName().contains("Emp"));
        return classMetadata.getClassName().contains("Emp");
    }
}
```



> **这里特别要注意的是 在使用includeFilters时，需要禁用掉默认的扫描规则，因为默认的扫描规则是扫描所有。** 禁用默认扫描规则及设置useDefaultFilters属性为false

#### 2.1.3 @Conditional

>条件注解，按照一定的条件进行判断，如果满足条件则给容器中注入bean。注解中可以传入条件数组
>
>下面举个例子： 
>
>​	条件- 如果容器中有Boss类，那么就给容器中注入一个Wife4Boss的对象，如果没有，就不注入。

```java
 	@Conditional(WifeCondition.class) //放在类上，则表示条件生效时这个配置类的所有配置才能生效
    @Bean
    public Wife4Boss wife(){
        return new Wife4Boss();
    }
```

> WifeCondition.class的代码,条件类必须实现Condition接口并实现matches方法，方法返回true表示满足条件返回false表示不满足条件

```java
public class WifeCondition implements Condition {
    /**
     * 筛选条件类
     * @param context 判断条件能使用的上下文对象
     * @param metadata 当前对象的注解信息JAVA
     * @return
     */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        //获取bean工厂，用来创建对象等
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        //获取环境对象,获取当前环境信息通过getProperty()
        Environment environment = context.getEnvironment();
        String osName = environment.getProperty("os.name");
        System.out.println(osName);
        //获取bean定义的注册类。可以查询注册bean,判断bean的注册情况
        BeanDefinitionRegistry registry = context.getRegistry();
        String[] beanDefinitionNames = registry.getBeanDefinitionNames();
        List<String> strings = Arrays.asList(beanDefinitionNames);
        System.out.println(strings);
        return registry.containsBeanDefinition("boss");
    }
}
```

#### 2.1.4 @Import

##### 2.1.4.1 快速注入

>通过Import注解，可以直接快速的导入组件，导入的组件名为类的全名小写。

```java
@Configuration
@ComponentScan(basePackages = "com.pansky")
//@Import快速给容器中注册组件，导入的组件的名称是类的全类名
@Import({Boss.class, Cat.class, Employee.class})
public class MainConfig2 {
}
```

> 如上，给容器中注入了Boss,Cat,Employee对象。

##### 2.1.4.2 选择注入 

> @Import注解中还可以填入ImportSelector的实现类，用以进行选择注入。

```java
public class MySelect implements ImportSelector {
    /**
     *
     * @param importingClassMetadata 注解对象，可以获取加载此类的注解的类的所有注解
     * @return 返回的数组内是全类名数组，数组内的类会被注册到容器中
     */
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        //方法不能返回null
        return new String[]{"com.pansky.entity.Appler","com.pansky.entity.Orager"};
    }
}
```

##### 2.1.4.3 手动注册

>@Import注解中也可以填入 ImportBeanDefinitionRegistrar 的实现类。通过该实现类可以完成手动注册组件。

```java
public class MyBeanDifinationRegester implements ImportBeanDefinitionRegistrar {

    /**
     *
     * @param importingClassMetadata 添加import注释的类的所有注释信息
     * @param registry bean定义的注册类，可以用来注册bean
     */
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        //获取容器内所有bean的name
        String[] beanDefinitionNames = registry.getBeanDefinitionNames();
        //判断容器内是否含有一个name为 com.pansky.entity.Boss的bean
        boolean b = registry.containsBeanDefinition("com.pansky.entity.Boss");
        if(b){
            //如果含有则创建一个Wife4Boss
            BeanDefinition beanDefinition = new RootBeanDefinition(Wife4Boss.class);
            registry.registerBeanDefinition("bossWife",beanDefinition);
        }
    }
}
```

> 如上，通过BeanDefinitionRegistry注册类完成。

#### 2.1.5 FactoryBean

> 注入实现FactoryBean接口的类，在获取bean时是通过FactoryBean的getObject方法获取的对象。
>
> 如果想要获取FactoryBean对象本身，则要在getBean("name")的beanname前加&符即 &name。

```java
/**
 * 描述: 注入这个bean,从容器中获取时实际上获取的是getObject的返回值
 *
 * @author X_P
 * @create 2018-12-10 11:02
 */
public class ColorFactoryBean implements FactoryBean<Orager> {
    @Override
    public Orager getObject() throws Exception {
        return new Orager();
    }
    @Override
    public Class<?> getObjectType() {
        return Orager.class;
    }
    /**
     * 是否是单例的
     * @return
     */
    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

### 2.2 组件的生命周期

>在spring中对于组件的生命周期控制有多种方式，下面我们一一介绍。

#### 2.2.1 init-method,destroy-method

> 常规的使用配置文件，可以在<bean>标签中定义 init-method属性，和destroy-method属性来指定bean创建时运行的方法，和销毁时运行的方法。在配置文件中，我们可以使用@Bean注解的initMethod、destroyMethod这两个属性来指定方法

```java
@Bean(initMethod="init",destroyMethod="destroy")
public Car car(){
    return new Car();
}
```

#### 2.2.2 InitializingBean、DisposableBean

> 实现InitializingBean、DisposableBean接口，重写相应方法后完成组件生命周期的控制

##### 2.2.2.1 InitializingBean 

>  对象初始化赋值完成后会调用此接口的afterPropertiesSet方法。

```java
public interface InitializingBean {
	/**
	* 对象初始化属性赋值完成后回调的方法
	*/
	void afterPropertiesSet() throws Exception;

}
```

##### 2.2.2.2 DisposableBean

> 在对象销毁前会调用此接口的destroy方法。

```java
public interface DisposableBean {

	/**
	 * 销毁前调用 
	 */
	void destroy() throws Exception;

}
```

#### 2.2.3 使用JSR250标准

> 从Java EE 5规范开始，Servlet中增加了两个影响Servlet生命周期的注解（Annotion）；@PostConstruct和@PreDestroy。这两个注解被用来修饰一个非静态的void()方法 。 

```java
public class Orager {

    @PostConstruct
    public void init(){
        System.out.println("橘子剥皮");
    }

    @PreDestroy
    public void eat(){
        System.out.println("橘子被吃了");
    }
}
```

> @PostConstruct注解修饰的方式是在构造器执行完成之后，init方法执行之前。
>
> @PreDestroy注解修饰的方法是在销毁方法执行之后执行。

#### 2.2.4 BeanPostProcessor

>Bean的后置处理器，它可以在bean的初始化前后进行操作。

```JAVA
public interface BeanPostProcessor {

	/**
	 * 在初始化之前进行后置处理工作
     */
	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;

	/**
	* 在初始化之后进行后置处理
	 */
	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

}
```

> 这是一个非常重要的接口，spring底层大量使用到了这个接口的实现类，下面我们详细了解一下这个接口的使用。

##### 2.2.4.1 BeanPostProcessor的执行原理

>首先我们通过跟踪源码的方式学习一下BeanPostProcessor的执行原理。
>
>1. 创建容器
>
>   ```JAVA
>   AnnotationConfigApplicationContext aac = new AnnotationConfigApplicationContext(MainConfig2.class);
>   //容器的构造器
>   public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
>   		this();
>   		register(annotatedClasses);
>       	//刷新容器
>   		refresh();
>   }
>   ```
>
>2. 刷新容器refresh(); (其余方法此处暂不讨论)
>
>   ```JAVA
>   public void refresh() throws BeansException, IllegalStateException {
>   		synchronized (this.startupShutdownMonitor) {
>   			prepareRefresh();
>   			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
>   			prepareBeanFactory(beanFactory);
>   			try {
>   				postProcessBeanFactory(beanFactory);
>   				invokeBeanFactoryPostProcessors(beanFactory);
>   				registerBeanPostProcessors(beanFactory);
>   				initMessageSource();
>   				initApplicationEventMulticaster();
>   				onRefresh();
>   				registerListeners();
>   				// 初始化剩余的所有非懒加载的单例实例
>   				finishBeanFactoryInitialization(beanFactory);
>   				finishRefresh();
>   			}
>   ```
>
>3. 初始化剩余实例时调用了preInstantiateSingletons()
>
>   ```java
>   @Override
>   	public void preInstantiateSingletons() throws BeansException {
>   		if (this.logger.isDebugEnabled()) {
>   			this.logger.debug("Pre-instantiating singletons in " + this);
>   		}
>           //获取所有的beanNames
>   		List<String> beanNames = new ArrayList<String>(this.beanDefinitionNames);
>           //遍历所有的beanName，判断是否为工厂bean
>   		for (String beanName : beanNames) {
>   			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
>   			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
>   				if (isFactoryBean(beanName)) {
>   					final FactoryBean<?> factory = (FactoryBean<?>) getBean(FACTORY_BEAN_PREFIX + beanName);
>   					boolean isEagerInit;
>   					if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
>   						isEagerInit = AccessController.doPrivileged(new PrivilegedAction<Boolean>() {
>   							@Override
>   							public Boolean run() {
>   								return ((SmartFactoryBean<?>) factory).isEagerInit();
>   							}
>   						}, getAccessControlContext());
>   					}
>   					else {
>   						isEagerInit = (factory instanceof SmartFactoryBean &&
>   								((SmartFactoryBean<?>) factory).isEagerInit());
>   					}
>   					if (isEagerInit) {
>   						getBean(beanName);
>   					}
>   				}
>   				else {
>                       //不是工厂bean则执行
>   					getBean(beanName);
>   				}
>   			}
>   		}
>   ```
>
>4. 执行getBean(beanName);->dogetBean()-->singletonFactory.getObject();-->createBean(beanName, mbd, args)
>
>5. 执行doCreateBean()
>
>   ```java
>   try {
>       //准备bean的属性值
>   			populateBean(beanName, mbd, instanceWrapper);
>   			if (exposedObject != null) {
>                   //bean的初始化
>   				exposedObject = initializeBean(beanName, exposedObject, mbd);
>   			}
>   		}
>   ```
>
>6. 执行initializeBean
>
>   ```JAVA
>   if (mbd == null || !mbd.isSynthetic()) {
>       //执行bean的前置处理器的初始化前调用方法
>   			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
>   		}
>   		try {
>               //bean的初始化方法
>   			invokeInitMethods(beanName, wrappedBean, mbd);
>   		}
>   		catch (Throwable ex) {
>   			throw new BeanCreationException(
>   					(mbd != null ? mbd.getResourceDescription() : null),
>   					beanName, "Invocation of init method failed", ex);
>   		}
>   		if (mbd == null || !mbd.isSynthetic()) {
>               //执行bean的前置处理器的初始化后调用发放
>   			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
>   		}
>   ```



> 从上述流程，可以看出在bean的初始化方法执行之前会调用applyBeanPostProcessorsBeforeInitialization，该方法会获取到将容器中所有的BeanPostProcessor，执行他们的postProcessBeforeInitialization方法。
>
> ```JAVA
> public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
> 			throws BeansException {
> 		Object result = existingBean;
> 		for (BeanPostProcessor beanProcessor : getBeanPostProcessors()) {
> 			result = beanProcessor.postProcessBeforeInitialization(result, beanName);
> 			if (result == null) {
> 				return result;
> 			}
> 		}
> 		return result;
> 	}
> ```

##### 2.2.4.2 BeanPostProcessor 在底层的应用

spring中很多注解的实现都是通过BeanPostProcessor的。比如AutoWared组件注入，bean的赋值，生命周期控制等。

比如，在bean初始化之前判断它是否有属性添加了某注解，如果添加了，则进行某些操作。

@EnableXXXX就是这么实现的。

BeanPostProcessor会经过所有的对象。

### 2.3 组件的赋值

> spring的赋值方式有多种，根据对象属性的不同使用不同的注解。

#### 2.3.1 @Value

>在xml配置中可以使用 <property>标签的name来指定属性名，用value属性来进行组件赋值。
>
>还可以使用@Value注解在实体类上直接注入属性值

```java
public class Employee implements Serializable{


    private static final long serialVersionUID = 1902835424187229669L;
    /**
     * 姓名
     */
    @Value("大刘")
    private String name;
    /**
     * 年龄
     */
    @Value("#{20-2}" )
    private Integer age;
    /**
     * 性别
     */
    @Value("${employee.gendar}")
    private String gender;
    
    }
```

> 1. @Value注解中可以直接写入基本数值
> 2. 也可以写入SPEL
> 3. 也可以使用${}取出配置文件中的值。配置文件在配置类上添加@PropertySource(value="")来导入

#### 2.3.2 @Autowired @Qualifier

> 基本属性可以使用@Value注解来进行赋值，对象属性则需要使用@Autowired @Qualifier来完成注入。
>
> 这些注解习惯于标注在成员变量上，**它们也可以标注在构造器，参数，方法上**。

##### 2.3.2.1 基本使用方法

> 在类的成员变量上添加@Autowired自动注入，容器会自动寻找与之对应类型的对象，并进行赋值。如下所示，容器默认会在容器中找到EmployeeDao类型的对象，并将这个对象赋值给dao。 如果没有找到这个类型的对象，则会报错，如果找到多个对象，则会根据属性名作为对象id来进行匹配。
>
> 而通过@Qualifier可以明确指定装配组件的id。
>
> 还可以通过给待装配的对象添加@Primary注解来表示默认注入这个对象。

```java
@Service
public class EmployeeService {
    @Qualifier("dao")
   @Autowired(required = false) //自动注入不是必须的 required
   private  EmployeeDao dao;
}
```

##### 2.3.2.2 @Autowired注入原理

> 下面我们来探索一下Autowired注入的原理，根据前面了解到的知识，肯定是通过BeanPostProcessor来进行处理那的，所以我们去找找会是那个BeanPostProcessor。

![1544501671210]({{site.url}}\imgs\springannotion\1544501671210.png)

> 可以看到BeanPostProcessor的实现类中有一个AutowiredAnnotationBeanPostProcessor，很明显就是这个了。我们看看它的

#### 2.3.3 @Resource(JSR250) @Inject(JSR330)

> spring还支持java规范实现对象属性的注入。
>
> @Resource：默认是按照对象的名称注入的，并不支持required = false和@Primary。
>
>  @Inject：需要导入javax.Inject，作用与Autowired一样，内部没有属性。
>
> 还是建议使用Autowired。

#### 2.3.4 注入spring底层类

> 在自定义组件中，想要使用spring底层的类可以通过实现XXXaware接口，接收回调函数注入的对象来获取。
>
> 比如我们要在自定义组件中使用spring容器类,实现 ApplicationContextAware，重写setApplicationContext方法，将set进来的对象存入applicationContext。

```java

@Controller
public class EmployeeController implements ApplicationContextAware {

    @Autowired
    private Appler appler;
    private ApplicationContext applicationContext;
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

实现的原理仍旧是通过BeanPostProcesser。

![1544512509412]({{site.url}}\imgs\springannotion\1544512509412.png)

```JAVA
@Override
	public Object postProcessBeforeInitialization(final Object bean, String beanName) throws BeansException {
		AccessControlContext acc = null;

		if (System.getSecurityManager() != null &&
				(bean instanceof EnvironmentAware || bean instanceof EmbeddedValueResolverAware ||
						bean instanceof ResourceLoaderAware || bean instanceof ApplicationEventPublisherAware ||
						bean instanceof MessageSourceAware || bean instanceof ApplicationContextAware)) {
			acc = this.applicationContext.getBeanFactory().getAccessControlContext();
		}

		if (acc != null) {
			AccessController.doPrivileged(new PrivilegedAction<Object>() {
				@Override
				public Object run() {
					invokeAwareInterfaces(bean);
					return null;
				}
			}, acc);
		}
		else {
			invokeAwareInterfaces(bean);
		}

		return bean;
	}
```

```JAVA
private void invokeAwareInterfaces(Object bean) {
		if (bean instanceof Aware) {
			if (bean instanceof EnvironmentAware) {
				((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
			}
			if (bean instanceof EmbeddedValueResolverAware) {
				((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
			}
			if (bean instanceof ResourceLoaderAware) {
				((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
			}
			if (bean instanceof ApplicationEventPublisherAware) {
				((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
			}
			if (bean instanceof MessageSourceAware) {
				((MessageSourceAware) bean).setMessageSource(this.applicationContext);
			}
			if (bean instanceof ApplicationContextAware) {
				((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
			}
		}
	}
```

如上所示，在类初始化完成后调用的postProcessBeforeInitialization，它对bean的类型进行了判断，如果是这其中Aware的一种，则根据类型的不同，将对象转为不同的Aware接口并调用相应的set方法，这是对象中重写的set方法生效，对应的底层类被获取到。

#### 2.3.5 @Profile

> 这个注解一般用于环境的切换，可以标注在类，方法上，只有当前的环境激活，标注当前Profile的配置才会生效

```JAVA
@Configuration
@EnableSwagger2
@Profile("dev")
public class Swagger2Config {
}
```

如上dev环境，只有在配置文件中激活的dev环境时此配置类才会生效。

```properties
spring.profiles.active='dev'
```

在dev激活的情况下所有没有标注@Profile的配置和标注了@Profile("dev")的配置会生效。

## 3. AOP

> springaop，面向切面编程，是对面向对象编程的补充，是一种将横向重复代码，进行纵向抽取的编程方式，是通过动态代理（存在接口时）或cglib（无接口则继承）实现的。 

### 3.1 spring aop如何使用

> 在使用aop前需要先创建自己的切面类，在切面类中标识好要对什么方法进行切面注入。同时要在配置类上开启aop自动代理。

#### 3.1.1 切面类的创建

```JAVA
@Aspect //切面类标识，添加这个注解，spring就会将这个类当做切面类
@Component //切面类也必须当做组件进行注入
public class ApplerPointCut {
    /**
     * 抽取公共切入点表达式，@Pointcut还可以将注解当做切入点，这样的方式更加灵活
     * @Pointcut("@annotation(com.pansky.entity.SysLog)") 凡是带有SysLog的注解均为切入点。
     */
    @Pointcut("execution(public String com.pansky.entity.Appler.*(..))")
    public void pointCut(){}
    /**
     * 切面类必须放入容器中
     *
     *  前置通知 方法执行之前执行
     */

    @Before("ApplerPointCut.pointCut()")
    public void applerCreatebefore(JoinPoint joinPoint){
        System.out.println("神说要有苹果");
    }

    /**
     * 后置通知 只要方法执行完就会进行，相当于finally中的代码
     */
    @After("pointCut()")
    public void applerCreateAfter(){
        System.out.println("创造苹果的魔法运行完毕");
    }

    /**
     * 异常通知 切入点抛出异常时执行。相当于catch内的代码，异常通知可以通过throwing来指定抛出的异常类型
     */
    @AfterThrowing(value = "pointCut()",throwing = "e")
    public void kk(Exception e){
        System.out.println("出事了"+e.getMessage());
        System.out.println("苹果产生失败了");
    }
    /**
     * 返回通知 切入点调用成功后执行，返回通知可以通过 returning来指定返回值
     */
    @AfterReturning(value = "pointCut()",returning ="o" )
    public void returngo(Object o){
        System.out.println(o);
        System.out.println("苹果获取成功");
    }

    /* 
     * 环绕通知 以动态代理方式，手动推进目标方法运行
     * ProceedingJoinPoint环绕通知必须使用此类，是JoinPoint的子类。
     * 可以在方法执行前后添加任意代码。
     */
    @Around("pointCut()")
    public void goAround(ProceedingJoinPoint joinPoint){
        //这里写执行前的
        //方法执行。
		joinPoint.proceed();
        //这里是执行后的。
    }
}
```

#### 3.1.2 AOP原理

##### 3.1.2.1@EnableAspectJAutoProxy 

>springAOP的生效需要在配置类上标注@EnableAspectJAutoProxy ，我们就从这个注解开始探索springAOP的原理。

```JAVA
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {
	
	boolean proxyTargetClass() default false;
	
	boolean exposeProxy() default false;

}

```

如上 EnableAspectJAutoProxy注解中Import了AspectJAutoProxyRegistrar.class，它继承了ImportBeanDefinitionRegistrar因而可以手动注入组件，我们看看它到底注入了什么组件。

```JAVA
class AspectJAutoProxyRegistrar implements ImportBeanDefinitionRegistrar {

	@Override
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        //①注册AspectJAnnotationAutoProxyCreator，如果需要的话
		AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);
        //获取注解EnableAspectJAutoProxy信息
		AnnotationAttributes enableAspectJAutoProxy =
				AnnotationConfigUtils.attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
        //根据注解值判断尽心操作，这里不详述了。
		if (enableAspectJAutoProxy.getBoolean("proxyTargetClass")) {
			AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
		}
		if (enableAspectJAutoProxy.getBoolean("exposeProxy")) {
			AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
		}
	}

}
```

>此处我们先跟进①的代码看看AspectJAnnotationAutoProxyCreator是如何注册的

```JAVA
//1先到这里
public static BeanDefinition registerAspectJAnnotationAutoProxyCreatorIfNecessary(BeanDefinitionRegistry registry) {
		return registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry, null);
	}
//2 在到这里
	public static BeanDefinition registerAspectJAnnotationAutoProxyCreatorIfNecessary(BeanDefinitionRegistry registry, Object source) {
		return registerOrEscalateApcAsRequired(AnnotationAwareAspectJAutoProxyCreator.class, registry, source);
	}
// 3 最终实际代码在这里
private static BeanDefinition registerOrEscalateApcAsRequired(Class<?> cls, BeanDefinitionRegistry registry, Object source) {
		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
    //首先判断在注册器中是否包含名字为AUTO_PROXY_CREATOR_BEAN_NAME的bean的定义信息
    //AUTO_PROXY_CREATOR_BEAN_NAME = org.springframework.aop.config.internalAutoProxyCreator
		if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
            //如果存在则获取这个bean的定义信息
			BeanDefinition apcDefinition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
			if (!cls.getName().equals(apcDefinition.getBeanClassName())) {
                //判断获取的bean的全类名与cls的全类名是否一致，如果不一致则获取这两个类的排序
				int currentPriority = findPriorityForClass(apcDefinition.getBeanClassName());
				int requiredPriority = findPriorityForClass(cls);
                //如果cls的排序靠前，则将bean定义信息的全类名更换为cls的
				if (currentPriority < requiredPriority) {
					apcDefinition.setBeanClassName(cls.getName());
				}
			}
			return null;
		}
    //如果注册器中不包含这个bean的定义信息则按cls创建bean并命名为AUTO_PROXY_CREATOR_BEAN_NAME
		RootBeanDefinition beanDefinition = new RootBeanDefinition(cls);
		beanDefinition.setSource(source);
		beanDefinition.getPropertyValues().add("order", Ordered.HIGHEST_PRECEDENCE);
		beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
		registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME, beanDefinition);
		return beanDefinition;
	}
```

> 可以看出最终AspectJAutoProxyRegistrar给容器中注入了一个名为  org.springframework.aop.config.internalAutoProxyCreator的类型为AnnotationAwareAspectJAutoProxyCreator.class的对象。（AspectJ自动代理创建器）



>同理的，对于很多EnableXXXX，我们先看它给容器中注入了什么组件，然后再通过分析这个组件的作用就可以知道这个EnableXXX到底干了什么。
>
>下面我们接着看AnnotationAwareAspectJAutoProxyCreator到底是什么作用。

![1544522523142]({{site.url}}\imgs\springannotion\1544522523142.png)

> 从类的继承图中可以看到它实现了BeanFactoryAwate和BeanPostProcessor接口。
>
> 通过这两个接口一个能够获取bean工厂，一个可以在bean初始化前后进行操作，因而我们重点关注这个问题。
>
> <u>写到这里我想到以前的一个疑惑，spring为什么每次都要自己添加EnableXXXXX，还有类似的xml配置<mvc:annoation-drivern>都要去写，为什么不写死。是因为这些东西底层 都是BeanPostProcessor，因为每个实例的创建都要经过所有的BeanPostProcessor，因而加入了多余的BeanPostProcessor，会影响性能，对象越多影响越大</u>。

##### 3.1.2.2 AnnotationAwareAspectJAutoProxyCreator

> 我们通过debug,系统的跟一下AnnotationAwareAspectJAutoProxyCreator代码的执行顺序，首先我们先找到它实现接口的回调方法，在上面打上断点。

![1544583725538]({{site.url}}\imgs\springannotion\1544583725538.png)

![1544583824962]({{site.url}}\imgs\springannotion\1544583824962.png)

> 断点打好后开始debug。

>  首先断点来到了setBeanFactory(BeanFactory)

```JAVA
@Override
	public void setBeanFactory(BeanFactory beanFactory) {
		super.setBeanFactory(beanFactory);
		if (!(beanFactory instanceof ConfigurableListableBeanFactory)) {
			throw new IllegalArgumentException(
					"AdvisorAutoProxyCreator requires a ConfigurableListableBeanFactory: " + beanFactory);
		}
		initBeanFactory((ConfigurableListableBeanFactory) beanFactory);
	}
```

> 通过断点回顾，我们看看是怎么来到这里的。首先创建容器类，在容器类的构造方法中执行了refresh();方法

```java
public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
		this();
		register(annotatedClasses);
		refresh();
	}
```

在refresh()方法中执行了如下方法。

```java
// 注册Bean的后置处理器，用来拦截bean的创建
 registerBeanPostProcessors(beanFactory);
```

然后到这里。

```JAVA
PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
```

在registerBeanPostProcessors中获取到所有的BeanPostProcessors，然后进行遍历区分，那些是实现PriorityOrdered.class接口的，那些是实现Ordered.class接口的，再把剩余的放在一起。

```java
public static void registerBeanPostProcessors(
			ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {
		//通过beanFactory获取所有BeanPostProcessor的名字（只是定义，未创建对象）
		String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);
   		 //创建BeanPostProcessorChecker
		int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
		beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));
    	//将BeanPostProcessor按照PriorityOrdered、Ordered来区分。分别放入不同的List
		List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<BeanPostProcessor>();
		List<BeanPostProcessor> internalPostProcessors = new ArrayList<BeanPostProcessor>();
		List<String> orderedPostProcessorNames = new ArrayList<String>();
		List<String> nonOrderedPostProcessorNames = new ArrayList<String>();
		for (String ppName : postProcessorNames) {
			if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
                //首先创建了实现PriorityOrdered接口的BeanPostProcessor
				BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
                //将对象放入集合中
				priorityOrderedPostProcessors.add(pp);
				if (pp instanceof MergedBeanDefinitionPostProcessor) {
					internalPostProcessors.add(pp);
				}
			}
			else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
                //实现order接口的没有创建对象，只是把全类名放入集合
				orderedPostProcessorNames.add(ppName);
			}
			else {
				nonOrderedPostProcessorNames.add(ppName);
			}
		}

		// 对实现PriorityOrdered接口的BeanPostProcessor进行排序，然后开始注册.
		sortPostProcessors(beanFactory, priorityOrderedPostProcessors);
		registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);
		// 对实现Ordered接口的BeanPostProcessor进行排序，然后开始注册. 
    	//二次遍历时会取出其中的 MergedBeanDefinitionPostProcessor。
		List<BeanPostProcessor> orderedPostProcessors = new ArrayList<BeanPostProcessor>();
		for (String ppName : orderedPostProcessorNames) {
			BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
			orderedPostProcessors.add(pp);
			if (pp instanceof MergedBeanDefinitionPostProcessor) {
				internalPostProcessors.add(pp);
			}
		}
		sortPostProcessors(beanFactory, orderedPostProcessors);
		registerBeanPostProcessors(beanFactory, orderedPostProcessors);

		// 注册剩余的BeanPostProcessor.
		List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList<BeanPostProcessor>();
		for (String ppName : nonOrderedPostProcessorNames) {
			BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
			nonOrderedPostProcessors.add(pp);
			if (pp instanceof MergedBeanDefinitionPostProcessor) {
				internalPostProcessors.add(pp);
			}
		}
		registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);

		// 最后注册内部的BeanPostProcessor.
		sortPostProcessors(beanFactory, internalPostProcessors);
		registerBeanPostProcessors(beanFactory, internalPostProcessors);

		// 重新注册ApplicationListenerDetector，将它添加进BeanPostProcessor的最末端。
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));
	}
```

> 上述代码中  BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);是获取对象的过程，其中调用了dogetBean=>getSingleton=>singletonFactory.getObject()=>createBean=>doCreateBean=>initializeBean...

```JAVA
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
		if (System.getSecurityManager() != null) {
			AccessController.doPrivileged(new PrivilegedAction<Object>() {
				@Override
				public Object run() {
					invokeAwareMethods(beanName, bean);
					return null;
				}
			}, getAccessControlContext());
		}
		else {
             //初始化Aware,判断bean是否实现了Aware接口，并调用相应的aware回调方法。
			invokeAwareMethods(beanName, bean);
		}

		Object wrappedBean = bean;
		if (mbd == null || !mbd.isSynthetic()) {
            //执行Bean后置处理器的初始化前方法
			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
		}

		try {
           //执行bean的初始化方法
			invokeInitMethods(beanName, wrappedBean, mbd);
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					(mbd != null ? mbd.getResourceDescription() : null),
					beanName, "Invocation of init method failed", ex);
		}

		if (mbd == null || !mbd.isSynthetic()) {
            //执行bean后置处理器初始化后的方法
			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
		}
		return wrappedBean;
	}
```

> invokeAwareMethods(beanName, bean); 回调函数调用了setBeanFactory();并执行了initBeanFactory()

```JAVA
protected void initBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		super.initBeanFactory(beanFactory);
		if (this.aspectJAdvisorFactory == null) {
			this.aspectJAdvisorFactory = new ReflectiveAspectJAdvisorFactory(beanFactory);
		}
		this.aspectJAdvisorsBuilder =
				new BeanFactoryAspectJAdvisorsBuilderAdapter(beanFactory, this.aspectJAdvisorFactory);
	}
```

> 最终给AnnotationAwareAspectJAutoProxyCreator 中添加了2个对象aspectJAdvisorFactory，aspectJAdvisorsBuilder























