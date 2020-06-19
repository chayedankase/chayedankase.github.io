---
layout: post
title: 1.用静态工厂方法代替构造器
categories: [effective java阅读]
description: effective java第一条
keywords: effective java
---

# effective java

## 1.用静态工厂方法代替构造器

​		对于类而言，为了让客户端获取它自身的一个实例，最传统的方法就是提供一个公有 的构造器。 还有一种方法，也应该在每个程序员的工具箱中占有一席之地。 类可以提供一个 公有的静态工厂 方法（ static factory method），它只是一个返回类的实例的静态方法。 下面是 一个来自 Boolean （基本类型 boolean 的装箱类）的简单示例。 这个方法将 boolean 基本 类型值转换成了一个 Boolean 对象引用：

```JAVA
public static Boolean valueOf(boolean b) { 
    return b? Boolean.TRUE : Boolean.FALSE; 
}
```

​		注意，静态工厂方法与设计模式［ Gamma95 ］中的工厂方法（ Factory Method） 模式不 同。 本条目中所指的静态工厂方法并不直接对应于设计模式（Design Pattern）中的工厂方法。 如果不通过公有的构造器， 或者说除了公有的构造器之外，类还可以给它的客户端提 供静态工厂方法。 提供静态工厂方法而不是公有的构造器，这样做既有优势，也有劣势。 

### 1.1 静态工厂的优势

#### 1.1.1 静态工厂方法与构造器不同的第一大优势在于，它们有名称。

​		如果构造器的参数本 身没有确切地描述正被返回的对象，那么具有适当名称的静态工厂会更容易使用，产生的 客户端代码也更易于阅读。 例如，构造器 Biginteger (int, int, Random）返回的 Biginteger 可能为素数，如果用名为 Biginteger.probablePrime 的静态工厂方法 来表示，显然更为清楚。 (Java 4 版本中增加了这个方法。）

​		一个类只能有一个带有指定签名的构造器。 编程人员通常知道如何避开这一限制 ： 通过提供两个构造器，它们的参数列表只在参数类型的顺序上有所不同。 实际上这并不是个好主 意。面对这样的 API， 用户永远也记不住该用哪个构造器， 结果常常会调用错误的构造器。 并且在读到使用了这些构造器的代码时，如果没有参考类的文档，往往不知所云。 

​		由于静态工厂方法有名称，所以它们不受上述限制。 当一个类需要多个带有相同签名 的构造器时，就用静态工厂方法代替构造器，并且仔细地选择名称以便突出静态工厂方法之 间的区别。

>类的构造器同一签名的情况下只能有一个，比如
>
>```JAVA
>public class Desk {
>    
>        private String color;
>        private String height;
>        private String width;
>        //这两个构造器同时只能存在一个
>        public Desk(String color) {
>            this.color = color;
>        }
>        //上面有了下面的就会编译错误
>        public Desk(String width){
>            this.width = width;
>        }
>        public Desk(String color, String height, String width) {
>            this.color = color;
>            this.height = height;
>            this.width = width;
>        }
>}
>```
>
>通过静态工厂方法返回对象，可以避免这种问题，并且通过良好的方法命名，静态工厂方法使用起来更加见名知意。  (原谅名字确实起的很烂)
>
>```JAVA
>public class Desk {
>
>        private String color;
>        private String height;
>        private String width;
>        //构造器私有化
>        private Desk(String color, String height, String width) {
>            this.color = color;
>            this.height = height;
>            this.width = width;
>        }
>        //获得一个有颜色的桌子
>        public static Desk colorDesk(String color){
>            return new Desk(color, null, null);
>        }
>        //获得一个有宽度的桌子
>        public static Desk widthDesk(String width){
>            return new Desk(null, null, width);
>        }
>    
>}
>```

#### 1.1.2 静态工厂方法与构造器不同的第二大优势在于，不必在每次调用它们的时候都创建一个新对象。

​		这使得不可变类（详见第 17 条）可以使用预先构建好的实例，或者将 构建好的实例缓存起来， 进行重复利用，从而避免创建不必要的重复对象。 Boolean. valueOf (boolean ）方法说明了这项技术 ： 它从来不创建对象。 这种方法类似于享元 (Flyweight）模式［ Gamma95 ］ 。 如果程序经常请求创建相同的对象，并且创建对象的代价 很高，则这项技术可以极大地提升性能。

​		静态工厂方法能够为重复的调用返回相同对象，这样有助于类总能严格控制在某个时刻哪些实例应该存在。 这种类被称作实例受控的类（ instance-controlled） 。 编写实例受控的 类有几个原因 。 实例受控使得类可以确保它是一个 Singleton （详见第 3 条）或者是不可实例 化的（详见第 4 条）。 它还使得不可变的值类（详见第 17 条）可以确保不会存在两个相等的 实例， 即当且仅当 a==b 时， a . equals(b ）才为 true。 这是享元模式［ Gamma95 ］ 的基 础。 枚举（enum）类型（详见第 34 条）保证了这一点。   

> 这个就不多解释了，参考单例模式，利用枚举类实现单例。

#### 1.1.3 静态工厂方法与构造器不同的第三大优势在于，它们可以返回原返回类型的任何子类型的对象

​		这样我们在选择返回对象的类时就有了更大的灵活性。 这种灵活性的一种应用是， API 可以返回对象，同时又不会使对象的类变成公有的。以 这种方式隐藏实现类会使 API 变得非常简洁。 这项技术适用于基于接口的框架（ interface-based framework) （详见第 20 条），因为在这种框架中 ，接口为静态工厂方法提供了自然返回 类型。 

​		在 Java 8 之前，接口不能有静态方法，因此按照惯例，接口 Type 的静态工厂方法被放在 一个名为 Types 的不可实例化的伴生类（详见第 4 条）中。 例如 Java Collections Framework 的集合接口有 45 个工具实现，分别提供了不可修改的集合、 同步集合，等等。 几乎所有这 些实现都通过静态工厂方法在一个不可实例化的类（ java . util. Collections ） 中导出。 所有返回对象的类都是非公有的。

​		现在的 Collections Framework API 比导出 45 个独立公有类的那种实现方式要小得多，每种便利实现都对应一个类。 这不仅仅是指 API 数量上的减少，也是概念意义上的减少： 为了使用这个 API，用户必须掌握的概念在数量和难度上都减少了。 程序员知道，被返回的对象是由相关的接口精确指定的，所以他们不需要阅读有关的类文档。 此外，使用这种静态 工厂方法时，甚至要求客户端通过接口来引用被返回的对象， 而不是通过它的实现类来引用 被返回的对象，这是一种良好的习惯（详见第 64 条）。 

​		从 Java 8 版本开始，接口中不能包含静态方法的这一限制成为历史，因此一般没有任何理由给接口提供一个不可实例化的伴生类。 已经被放在这种类中的许多公有的静态成员， 应该被放到接口中去。但是要注意，仍然有必要将这些静态方法背后的大部分实现代码， 单独放进一个包级私有的类中。 这是因为在 Java 8 中仍要求接口的所有静态成员都必须是 公有的。 在 Java 9 中允许接口有私有的静态方法，但是静态域和静态成员类仍然需要是公 有的。 


> 合理使用这种方式，能够实现一些很不错的写法。创建对象时通过不同的参数，获取父类引用子类实现。
>
> 感觉配合枚举更合适。
>
> ```JAVA
> /**
>  * 检测的接口
>  *
>  * @author Xue-Pan
>  * @date 2020/2/8
>  */
> public interface ValidateHandler {
>     //用于检测的方法
>     String valid(String info);
> }
> /**
>  * 效验的父类
>  *
>  * @author Xue-Pan
>  * @date 2020/2/8
>  */
> public class FatherValidateHandler implements ValidateHandler{
>     //因为要被继承所以不能私有化，这是静态工厂方法的劣势之一，不过可以用复合来解决。
>     protected FatherValidateHandler(){
>     }
>     //不同的类型获取不同的实现子类，适用性好很多。
>     public static FatherValidateHandler createValidateHandler(String type){
>         if("phone".equals(type)){
>             return new PhoneValidateHandler();
>         }else if("card".equals(type)){
>             return new CardValidateHandler();
>         }else{
>             return null;
>         }
>     }
>     @Override
>     public String valid(String info) {
>         return "这是父类的实现";
>     }
>     //测试的方法
>     public static void main(String[] args) {
>         FatherValidateHandler card = 	FatherValidateHandler.createValidateHandler("card");
>         String v = card.valid("111");
>         System.out.println(v);
>     }
> }
> public class CardValidateHandler extends FatherValidateHandler {
>     @Override
>     public String valid(String info) {
>         return "通过卡来认证";
>     }
> }
> public class PhoneValidateHandler extends FatherValidateHandler {
> 
>     @Override
>     public String valid(String info) {
>         return "获取到了手机号，进行了认证";
>     }
> }
> ```
> 

#### 1.1.4 静态工厂的第四大优势在于，所返回的对象的类可以随着每次调用而发生变化，这取 决于静态工厂方法的参数值。

 		只要是已声明的返回类型的子类型，都是允许的。 返回对象的 类也可能随着发行版本的不同而不同。EnumSet （详见第 36 条）没有公有的构造器，只有静态工厂方法。 在 OpenJDK 实现中， 它们返回两种子类之一的一个实例，具体则取决于底层枚举类型的大小：如果它的元素有 64 个或者更少，就像大多数枚举类型一样，静态工厂方法就会返回一个 RegalarEumSet 实例， 用单个 long 进行支持；如果枚举类型有 65 个或者更多元素，工厂就返回 JumboEnumSet 实例，用一个 long 数组进行支持。

​		这两个实现类的存在对于客户端来说是不可见的。 如果 RegularEnumSet 不能再给 小的枚举类型提供性能优势，就可能从未来的发行版本中将它删除，不会造成任何负面的影 H向。 同样地，如果事实证明对性能有好处，也可能在未来的发行版本中添加第三甚至第四个 EnumSet 实现。 客户端永远不知道也不关心它们从工厂方法中得到的对象的类，它们只关 心它是 EnumSet 的某个子类。  
> 这一点跟第三点有点接近。相当于对参数进行判断来返回不同的实现类。
> ```JAVA
>public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
>   Enum<?>[] universe = getUniverse(elementType);
>   if (universe == null)
>          throw new ClassCastException(elementType + " not an enum");
>      if (universe.length <= 64)
>          return new RegularEnumSet<>(elementType, universe);
>      else
>          return new JumboEnumSet<>(elementType, universe);  
>    }
>    ```
>  

#### 1.1.5 静态工厂的第五大优势在于，方法返回的对象所属的类，在编写包含该静态工厂方法的类时可以不存在。

​		这种灵活的静态工厂方法构成了服务提供者框架（ Service Provider Framework）的基础，例如 JDBC(Java 数据库连接）API。 服务提供者框架是指这样一个系统：多个服务提供者实现一个服务，系统为服务提供者的客户端提供多个实现，并把它们从多个实现中解耦出来。 

​		服务提供者框架中有三个重要的组件：服务接口 （ Service Interface），这是提供者实现的；提供者注册 API ( Provider Registration API），这是提供者用来注册实现的；服务访问 API (Service Access API)，这是客户端用来获取服务的实例。 服务访问 API 是客户端用来指定某种选择实现的条件。 如果没有这样的规定，API 就会返回默认实现的一个实例，或者允许客户端遍历所有可用的实现。服务访问 API 是“灵活的静态工厂”，它构成了服务提供者 框架的基础。 

​		服务提供者框架的第四个组件服务提供者接口（Service Provider Interface）是可选的， 它表示产生服务接口之实例的工厂对象。 如果没有服务提供者接口，实现就通过反射方式进行实例化（详见第 65 条）。 对于 JDBC 来说， Connection就是其服务接口的一部分， DriverManager.registerDriver 是提供者注册 API,DriverManager.getConnection 是服务访问 API, Driver 是服务提供者接口 。

​		服务提供者框架模式有着无数种变体。 例如，服务访问 API 可以返回比提供者需要的更丰富的服务接口 。 这就是桥接（ Bridge）模式 ［ Gamma95 ］ 。 依赖、注入框架（详见第 5 条） 可以被看作是一个强大的服务提供者。 从 Java 6 版本开始， Java 平台就提供了一个通用的 服务提供者框架 java . util . ServiceLoader ，因此你不需要（一般来说也不应该）再自己编写了（详见第 59 条）。 JDBC 不用 ServiceLoader ，因为前者出现得比后者早。

> 这是指通过静态工厂获取对象，然后用提供的接口来进行接收。实际上获取的对象是由提供者来实现的。
>
> 感觉在spring的环境下，更易于使用。比如：我会从容器中获取到A，然后调用A的do方法，A是一个接口。你写了一个实现了A的B接口，然后创建对象注入容器。结果就是我用你的对象调用了do方法。如果有变化，你可以重新注入一个实现了A的C对象，而我这里就变成了调用C的do方法。

### 1.2静态工厂的劣势

#### 1.2.1 静态工厂方法的主要缺点在子，类如果不含公有的或者受保护的构造器，就不能被子类化。 

​		例如，要想将 Collections Framework 中的任何便利的实现类子类化， 这是不可能的。 但是这样也许会因祸得福，因为它鼓励程序员使用复合（ composition），而不是继承（详见 第四条），这正是不可变类型所需要的（详见第 17 条）。

> 构造器私有化的类无法被继承，而通过静态工厂方法返回实例的对象，最好是将构造器私有化，防止通过构造器创建对象，但这样一来，该类便无法被继承。但可以通过复合来实现继承的效果（策略模式），即不使用继承的 是什么概念，而是使用 能干什么的感念。

#### 1.2.2 静态工厂方法的第二个缺点在于，程序员很难发现它们。

​		在 API 文档中，它们没有像 构造器那样在 API 文档中明确标识出来， 因此 对于提供了静态工厂方法而不是构造器的 类来说，要想查明如何实例化一个类是非常困难的。 Javadoc 工具总有一天会注意到静态工 厂方法。 同时，通过在类或者接口注释中关注静态工厂， 并遵守标准的命名习惯，也可以弥 补这一劣势。 下面是静态工厂方法的一些惯用名称。 这里只列出了其中的一小部分： 

- from一一类型转换方法，它只有单个参数，返回该类型的一个相对应的实例，例如：
  
```JAVA
  Dated= Date.from(instant) ; 
```

- of 聚合方法，带有多个参数，返回该类型的一个实例，把它们合并起来，例如 ：
  
```java
  Set<Rank> faceCards = EnumSet.of(JACK,QUEEN,KING);
```

- valueOf一一比 from 和 of 更烦琐的一种替代方法，例如 ：
  
```JAVA
  BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
```

- instance 或者 getInstance一返回的实例是通过方法的（如有）参数来描述 的，但是不能说与参数具有同样的值，例如 ：

  ```JAVA
  StackWalker luke = StackWalker.getInstance(options);
  ```

- create 或者 newInstance一一像instance 或者 getInstance 一样，但 create 或者 newInstance 能够确保每次调用都返回一个新的实例 ，例如：

  ```JAVA
  Object newArray = Array.newInstance(classObject, arrayLen);
  ```

- get Type一一像 getInstance 一样，但是在工厂方法处于不同的类中的时候使用。Type 表示工厂方法所返回的对象类型，例如：

  ```JAVA
  FileStore fs = Files.getFileStore(path);
  ```

- newType一一像newInstance 一样，但是在工厂方法处于不同的类中的时候使用。Type 表示工厂方法所返回的对象类型，例如：

  ```JAVA
  BufferedReader br＝ Files.newBufferedReader(path);
  ```

- type一一getType 和 newType 的简版，例如：

  ```JAVA
  List<Complaint> litany ＝ Collections.list(legacylitany);
  ```

​		简而言之，静态工厂方法和公有构造器都各有用处，我们需要理解它们各自的长处。 静态工厂经常更加合适，因此切忌第一反应就是提供公有的构造器， 而不先考虑静态工厂。
