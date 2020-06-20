---
layout: post
title: 2.遇到多个构造器参数时要考虑使用构建器
categories: [effective java阅读]
description: effective java第二条
keywords: effective java
---

# effective java

## 2. 遇到多个构造器参数时要考虑使用构建器

​		静态工厂和构造器有个共同的局限性：它们都不能很好地扩展到大量的可选参数。 比如 用一个类表示包装食品外面显示的营养成分标签。 这些标签中有几个域是必需的：每份的含 量、每罐的含量以及每份的卡路里。 还有超过 20 个的可选域： 总脂肪量、饱和脂肪量、转 化脂肪、胆固醇、纳，等等。 大多数产品在某几个可选域中都会有非零的值。 

​		对于这样的类，应该用哪种构造器或者静态工厂来编写呢？程序员一向习惯采用重叠构造器（ telescoping constructor）模式，在这种模式下，提供的第一个构造器只有必要的参 数，第二个构造器有一个可选参数，第三个构造器有两个可选参数，依此类推，最后一个构造器包含所有可选的参数。下面有个示例，为了简单起见，它只显示四个可选域：

```java
//Telescoping constructor pattern - does not scale well! 
public class NutritionFacts { 
    Private final int servingSize; 	// (ml) 			required 
    Private final int servings; 	// (per container) 	required 
    Private final int calories;		// (per serving) 	optiona1 
    private final int fat; 			// (g/serving) 		optional 
    Private final int sodium; 		// (mg/serving)		optiona1 
    private final int carbohydrate; // (g/serving) 		optiona1 
    
    public NutritionFacts (int servingSize , int servings) { 
        this (servingSize, servings, 0);
    }
    public NutrftionFacts(int servingSize, int servings, int calories) { 
        this(servingSize , servings, calories, 0);
    }
    public NutritionFacts(int servingSize, int servings, int calortes, int fat) {
        this(servingSize, servings, calortes, fat , 0); 
    }
    public NutritionFacts(int servingSize, int servings, int calorfes, int fat, int sodium) { 
        this(servingSize, servings, calories, fat, sodium, 0); 
	}
    public NutritionFacts(int servingSize,int servings,int calories,int fat,int sodium, int carbohydrate) { 
        this.servingSize = servingSize; 
        this.servings = servings; 
        this.calories = calories; 
        this.fat = fat; 
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

当你想要创建实例的时候，就利用参数列表最短的构造器，但该列表中包含了要设置 的所有参数：

```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35 , 27); 
```

​		这个构造器调用通常需要许多你本不想设置的参数，但还是不得不为它们传递值。 在 这个例子中，我们给 fat 传递了一个值为 0。 如果“仅仅”是这 6 个参数，看起来还不算太 糟糕，问题是随着参数数目的增加，它很快就失去了控制。 

​		**简而言之，重叠构造器模式可行，但是当有许多参数的时候，客户端代码会很难缩写， 并且仍然较难以阅读。** 如果读者想知道那些值是什么意思，必须很仔细地数着这些参数来探 个究竟。 一长串类型相同的参数会导致一些微妙的错误。 如果客户端不小心颠倒了其中两个 参数的顺序，编译器也不会出错，但是程序在运行时会出现错误的行为（详见第 51 条）。 

​		遇到许多可选的构造器参数的时候，还有第二种代替办法，即 JavaBeans 模式，在这 种模式下，先调用一个无参构造器来创建对象，然后再调用 setter 方法来设置每个必要的参数，以及每个相关的可选参数：

```java
// JavaBeans Pattern - allows inconsistency, mandates mutability 
public class NutrftionFacts { 
    // Parameters initialized to default values (if any] 
    Private int servingSize  = -1; // Required; no defau1t va1ue 
    private int servings 	 = -1; // Required; no default value 
    private int calories 	 = 0; 
    Private int fat 		 = 0; 
    Prtvate int sodium 		 = 0; 
    Private int carbohydrate = 0; 
	public NutritionFacts() { }
    // Setters 
    public void setServingSize(int val)  { servingSize = val; }
    public void setServings(int val )    { servings = val; }
    public void setCalories(int val)	 { calories = val; }
    public void setFat(int val) 		 { fat=val; }
    public void setSodium(int val) 		 { sodium= val; }
    public void setcarbohydrate(int val) { carbohydrate = va1; } 
}
```

​		这种模式弥补了重叠构造器模式的不足。 说得明白一点，就是创建实例很容易，这样 产生的代码读起来也很容易：

```java
NutrftionFacts cocaCola = new NutrftionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8); 
cocaCola.setCalories(100);	
cocaCola.setSodium(35);
cocaCola.setcarbohydrate(27);
```

​		遗憾的是， JavaBeans 模式自身有着很严重的缺点。 因为构造过程被分到了几个调用中， **在构造过程中 JavaBean 可能处于不一致的状态。** 类无法仅仅通过检验构造器参数的有效性 来保证一致性。 试图使用处于不一致状态的对象将会导致失败，这种失败与包含错误的代码 大相径庭，因此调试起来十分困难。 与此相关的另一点不足在于， **JavaBeans 模式使得把 类做成不可变的可能性不复存在** （详见第 17 条），这就需要程序员付出额外的努力来确保它 的线程安全。 

​		当对象的构造完成，并且不允许在冻结之前使用时，通过手工“冻结”对象可以弥补这 些不足，但是这种方式十分笨拙，在实践中很少使用。 此外，它甚至会在运行时导致错误， 因为编译器无法确保程序员会在使用之前先调用对象上的 freeze 方法进行冻结。 

​		幸运的是，还有第三种替代方法，它既能保证像重叠构造器模式那样的安全性，也能 保证像 JavaBeans 模式那么好的可读性。 这就是建造者（ Builder）模式 ［ Gamma95 ］ 的一 种形式。 它不直接生成想要的对象，而是让客户端利用所有必要的参数调用构造器（或者静 态工厂），得到一个 builder 对象。 然后客户端在 builder 对象上调用类似于 setter 的方法，来设置每个相关的可选参数。 最后 客户端调用无参的 build 方法来生成通常是不可变的对象。 这个buiider通常是它构建的类的静态成员类（详见第 24 条）。 下面就是它的示例 ：

```java
// Builder Pattern 
public class NutritionFacts { 
    Private final int servingSize; 
    Private final int servings; 
    Private final int calortes; 
    Private final int fat; 
    Private final int sodium; 
    private final int carbohydrate;
    public static class Builder｛ 
        // Required parametes 
        private final int servingSize; 
        Prtvate final int sevings;
        // Optional parametes - initialized to default values 
        Private int calories 	 = 0; 
        Prtvate int fat 		 = 0; 
        Private int sodium		 = 0; 
        Private int carbohydrate = 0; 
    	//builer构造器，保证必传的两个参数
        public Builder(int servingSize, int servings) { 
            this.servingSize = servingSize; 
            this.servings = servings; 
        } 
        public Builder calories(int val) { calories = val ;return this; } 
        public Builder fat(int val) { fat = val ;return this; } 
        public Builder sodium(int val) { sodium = val ;return this; } 
        public Builder carbohydrate(int val) { carbohydrate = val;return this; } 
        public NutritionFacts build() { return new NutritionFacts(this);} 
	}
	Private NutrtionFacts(Builder builder）｛ 
        servingSize = builder.servingSize; 
        servings = builder.servings; 
        calories = bui1der.calories; 
        fat = builder.fat; 
        sodium = builder.sodium; 
        carbohydrate = builder.carbohydrate;
   }
}                              
```

​		注意 NutritionFacts 是不可变的，所有的默认参数值都单独放在）个地方。builder 的设值方法返回 builder 本身，以便把调用链接起来，得到一个流式的 API。 下面就是其客 户端代码 ：

```JAVA
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).calortes(100).sodium(35).carbohydrate(27). bui1d();
```

​		这样的客户端代码很容易编写，更为重要的是易于阅读。 **BuiIder模式模拟了具名的可选参数**，就像 Python 和 Scala 编程语言中的一样。 

​		为了简洁起见，示例中省略了有效性检查。 要想尽快侦测到无效的参数，可以在 builder 的构造器和方法中检查参数的有效性。 查看不可变量，包括build方法调用的构造器中的多个参数。 为了确保这些不变量免受攻击， 从builder复制完参数之后，要检查对象域（详见第 50 条） 。 如果检查失败，就抛出 illegalArgumentException （详见第 72 条），其中的详细信息会说明哪些参数是无效的（详见第 75 条）。

​		Builder 模式也适用于类层次结构。 使用平行层次结构的 builder 时，各自嵌套在相应的 类中。抽象类有抽象的 builder，具体类有具体的 builder。假设用类层次根部的一个抽象类 表示各式各样的比萨：

```JAVA
// Builder pattern for class hierarchies 
public abstract class Pizza { 
    public enum Topping { HAM , MUSHROOM, ONION, PEPPER, SAUSAGE } 
    final Set<Topping> toppings; 
	abstract static class Builder<T extends Builder<T>>{ 
        EnumSet<Topping> toppings= EnumSet.noneOf(Topping.class); 
        public T addTopping(Topping topping) { 
            toppings.add(Objects.requireNonNull(topping)); 
            return self(); 
		} 
		abstract Pizza build(); 
        //Subclasses must override this method to return ” this" 
        Protected abstract T self(); 
	} 
    Pizza(Builder<?> builder）{
        toppings = builder.toppings.clone(); // 详见50条
    }
}
```

​		注意，Pizza.Builder 的类型是泛型（generic type），带有一个递归类型参数（recursive type parameter），详见第 30 条。 它和抽象的 self 方法一样，允许在子类中适当地进行方法链接，不需要转换类型。 这个针对 Java 缺乏 self类型的解决方案，被称作模拟的 self 类 型（simulated self-type）。 

​		这里有两个具体的 Pizza 子类，其中一个表示经典纽约风味的比萨，另一个表示馅料内置的半月型（ calzone）比萨。 前者需要一个尺寸参数，后者则要你指定酱汁应该内置还是外置：

```JAVA
public class NyPizza extends Pizza { 
    public enum Size { SMALL ，MEDIUM, LARGE } 
    private final Size size; 
    
	public static class Builder extends Pizza.Builder<Builder>{
        private final Size size; 
        public Builder(Size size){ 
            this.size = Objects.requireNonNull(size);
        }
        @Override 
        public NyPizza build() { 
            return new NyPizza(this); 
        }
        @Override 
        Protected Builder self() {return this; }
 	}
	Private NyPizza(Builder builder){ 
        super(builder); 
        size = builder.size;
    }
}

public class Calzone extends Pizza { 
    Private final boolean sauceInside; 
	public static class Builder extends Pizza.Builder<Builder>{ 
        Private boolean sauceinside = false; // Default 
		public Builder sauceInside() { 
            saucelnside = true; 
            return this;
        }
		@Override
        public Calzone build() { 
           return new Calzone(this);
        }
        @Override 
        protected Builder self() {return this; } 
    }
	Private Calzone(Builder builder){
        super(builder); 
        sauceInside = builder sauceInside;
    }
}
```

​		注意，每个子类的构建器中的 build 方法，都声明返回正确的子类： NyPizza.Builder 的 build 方法返回 NyPizza ，而 Calzone.Builder 中的则返回 Calzone。 在该 方法中，子类方法声明返回超级类中声明的返回类型的子类型，这被称作协变返回类型 (covariant return type）。 它允许客户端无须转换类型就能使用这些构建器。 

​		这些“层次化构建器” 的客户端代码本质上与简单的 NutritionFacts 构建器一样。 为了简洁起见，下列客户端代码示例假设是在枚举常量上静态导人：

```JAVA
NyPizza pizza= new NyPizza.Builder(SMALL).addTopping(SAUSAGE).addTopping(ONION).build(); 
Calzone calzone =new Calzone.Builder().addTopping(HAM).sauceInside().build(); 
```

​		与构造器相比， builder 的微略优势在于，它可 以有多个可变（ varargs） 参数。 因为 builder 是利用单独的方法来设置每一个参数。 此外，构造器还可以将多次调用某一个方法 而传人的参数集中到一个域中，如前面的调用了两次 addTopping 方法的代码所示。

​		Builder 模式十分灵活，可以利用单个 builder 构建多个对象。 buildr的参数可以在调用 build 方法来创建对象期间进行调整，也可以随着不同的对象而改变。 builder 可以自动填充某些域，例如每次创建对象时自动增加序列号。 

​		Builder 模式的确也有它自身的不足。 为了创建对象，必须先创建它的构建器。 虽然创 建这个构建器的开销在实践中可能不那么明显 但是在某些十分注重性能的情况下，可能就 成问题了。 Builder 模式还比重叠构造器模式更加冗长，因此它只在有很多参数的时候才使 用，比如 4 个或者更多个参数。 但是记住，将来你可能需要添加参数。 如果一开始就使用构 造器或者静态工厂，等到类需要多个参数时才添加构造器，就会无法控制，那些过时的构造器或者静态工厂显得十分不协调。 因此，通常最好一开始就使用构建器。 

​		简而言之， **如果类的构造器或者静态工厂中具有多个参数，设计这种类时， Builder模式就是一种不错的选择**， 特别是当大多数参数都是可选或者类型相同的时候。 与使用 重叠构造器模式相比，使用 Builder 模式的客户端代码将更易于阅读和编写，构建器也比 JavaBeans 更加安全。

## 3.用私有构造器或者枚举类型强化Singleton属性

​		Singleton 是指仅仅被实例化一次的类 ［ Gamma95 ］。 Singleton 通常被用来代表一个无状态的对象，如函数（详见第 24 条），或者那些本质上唯一的系统组件。 **使类成为 Singleton 会使它的客户端测试变得十分困难**，因为不可能给 Singleton 替换模拟实现，除非实现一个充当其类型的接口 。 

​		实现 Singleton 有两种常见的方法。 这两种方法都要保持构造器为私有的，并导出公有的静态成员，以便允许客户端能够访问该类的唯一实例。 在第一种方法中，公有静态成员是 个 final 域：

```JAVA
// Singleton with public final field 
public class Elvis { 
    public static final Elvis INSTANCE= new Elvis(); 
    Private Elvis() { .. . } 
	public void leaveTheBuilding() { ... } 
}
```

​		私有构造器仅被调用一次，用来实例化公有的静态 final 域 Elvis . INSTANCE。 由于 缺少公有的或者受保护的构造器，所以保证了 Elvis 的全局唯一性 ： 一旦 Elvis 类被实例 化，将只会存在一个 Elvis 实例 ，不多也不少。 客户端的任何行为都不会改变这一点，但 要提醒一点：享有特权的客户端可以借助 AccessibleObject.setAccessible 方法， 通过反射机制（详见第 65 条）调用私有构造器。 如果需要抵御这种攻击，可以修改构造器， 让它在被要求创建第二个实例的时候抛出异常。 

> 修改构造器为：
>
> ```JAVA
> public class Elvis { 
>     public static final Elvis INSTANCE= new Elvis(); 
>     Private Elvis() {
>     	if(INSTANCE!=NULL){
>             throw new RuntimeException("已经创建过了");
>         }
>         ...
>     } 
> 	public void leaveTheBuilding() { ... } 
> }
> ```

​		在实现 Singleton 的第二种方法中，公有的成员是个静态工厂方法：

```JAVA
// Singleton with static factory 
public class Elvis { 
    private static final Elvis INSTANCE = new Elvis(); 
    prvate Elvis() { ... } 
    public static Elvis getInstance() {return INSTANCE; } 
	public void leaveTheBuilding() { ... } 
}
```

​		对于静态方法 Elvis . getinstance 的所有调用，都会返回同一个对象引用，所以，永远不会创建其他的 Elvis 实例（上述提醒依然适用）。

​		公有域方法的主要优势在于， API 很清楚地表明了这个类是一个 Singleton ： 公有的静态域是 final 的，所以该域总是包含相同的对象引用。 第二个优势在于它更简单。 

​		静态工厂方法的优势之一在于，它提供了灵活性： 在不改变其 API 的前提下 ， 我们可 以改变该类是否应该为 Singleton 的想法。 工厂方法返回该类的唯一实例，但是，它很容易 被修改， 比如改成为每个调用该方法的线程返回一个唯一的实例。 第二个优势是， 如果应用 程序需要，可以编写一个泛型 Singleton 工厂 （ generic singleton factory) （详见第 30 条）。 使 用静态工厂的最后一个优势是，可以通过方法引用（ method reference）作为提供者，比如 Elvis : : instance 就是一个 Supplier<Elvis＞。 除非满足以上任意一种优势 ， 否则还 是优先考虑公有域（public-field）的方法。 

​		为了将利用上述方法实现的 Singleton 类变成是可序列化的（Serializable）(详见第 12 章），仅仅在声明中加上 implements Serializable 是不够的。为了维护并保证 Singleton, 必须声明所有实例域都是瞬时（ transient ）的，并提供一个 readResolve 方法（详见 第四条）。否则 ，每次反序列化一个序列化的实例时，都会创建一个新的实例，比如，在我 们的例子中，会导致“假冒的Elvis ” 。 为了防止发生这种情况，要在 Elvis 类中加入如 下 readResolve 方法：

```JAVA
// readResolve method to preserve singleton property 
private Object readResolve () { 
    // Return the one true Elvis and let the garbage coll ector 
    // take care of the Elvis impersonatar.
    return INSTANCE; 
}
```

​		实现 Singleton 的第三种方法是声明一个包含单个元素的枚举类型：

```java
// Enum singleton - the preferred approach 
public enum Elvis { 
    INSTANCE; 
	public void leaveTheBuilding() { .. . } 
}
```

这种方法在功能上与公有域方法相似，但更加简洁，无偿地提供了序列化机制，绝对 防止多次实例化，即使是在面对复杂的序列化或者反射攻击的时候。 虽然这种方法还没有广 泛采用，但是单元素的枚举类型经常成为实现 Singleton 的最佳方法。 注意，如果 Singleton 必须扩展一个超类，而不是扩展 Enum 的时候，则不宜使用这个方法（虽然可以声明枚举去实现接口）。

> 这一点的很多内容，跟单例模式相关，深入学习单例模式后，对这一点能更好的理解。