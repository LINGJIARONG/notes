# 项目

## RPC 框架

![image-20200909234437445](/Users/ano/Desktop/notes/image/image-20200909234437445.png)





# 设计模式

## 创建型模式

- 单例（Singleton）模式：某个类只能生成一个实例，该类提供了一个全局访问点供外部获取该实例，其拓展是有限多例模式。
- 原型（Prototype）模式：将一个对象作为原型，通过对其进行复制而克隆出多个和原型类似的新实例。
- 工厂方法（FactoryMethod）模式：定义一个用于创建产品的接口，由子类决定生产什么产品。
- 抽象工厂（AbstractFactory）模式：提供一个创建产品族的接口，其每个子类可以生产一系列相关的产品。
- 建造者（Builder）模式：将一个复杂对象分解成多个相对简单的部分，然后根据不同需要分别创建它们，最后构建成该复杂对象。

### 单例模式

DCL :  

```java
public class Singleton {  
      private volatile static Singleton instance;  
      private Singleton (){
      }   
      public static Singleton getInstance() {  
      if (instance== null) {  
          synchronized (Singleton.class) {  
          if (instance== null) {  
              instance= new Singleton();  
          }  
         }  
     }  
     return singleton;  
     }  
 }  
```

**静态内部类单例模式**

```java
public class Singleton { 
    private Singleton(){
    }
      public static Singleton getInstance(){  
        return SingletonHolder.sInstance;  
    }  
    private static class SingletonHolder {  
        private static final Singleton sInstance = new Singleton();  
    }  
} 
```

反序列化操作提供了readResolve方法，这个方法可以让开发人员控制对象的反序列化。在上述的几个方法示例中如果要杜绝单例对象被反序列化是重新生成对象，就必须加入如下方法：

```java
private Object readResolve() throws ObjectStreamException{
		return singleton;
}
```



### 原型模式

浅拷贝：将一个对象复制后，基本类型会被重新创建，引用类型的对象会把引用拷贝过去，实际上还是指向的同一个对象

深拷贝：将一个对象复制后，基本类型和引用类型的对象都会被重新创建

##### 深拷贝

深拷贝的实现方案主要有两种

1. 引用类型也使用clone()，进行clone的时候，对引用类型在调用一次clone()方法
2. 使用序列化，将对象序列化后在反序列化回来，得到新的对象实例

### builder模式

```java
public final class Car {
    /**
     * 必需属性
     */
    final String carBody;//车身
    final String tyre;//轮胎
    final String engine;//发动机
    final String aimingCircle;//方向盘
    final String safetyBelt;//安全带
    /**
     * 可选属性
     */
    final String decoration;//车内装饰品
    /**
     * car 的构造器 持有 Builder,将builder制造的组件赋值给 car 完成构建
     * @param builder
     */
    public Car(Builder builder) {
        this.carBody = builder.carBody;
        this.tyre = builder.tyre;
        this.engine = builder.engine;
        this.aimingCircle = builder.aimingCircle;
        this.decoration = builder.decoration;
        this.safetyBelt = builder.safetyBelt;
    }
    //...省略一些get方法
    public static final class Builder {
        String carBody;
        String tyre;
        String engine;
        String aimingCircle;
        String decoration;
        String safetyBelt;

        public Builder() {
            this.carBody = "宝马";
            this.tyre = "宝马";
            this.engine = "宝马";
            this.aimingCircle = "宝马";
            this.decoration = "宝马";
        }
         /**
         * 实际属性配置方法
         * @param carBody
         * @return
         */
        public Builder carBody(String carBody) {
            this.carBody = carBody;
            return this;
        }

        public Builder tyre(String tyre) {
            this.tyre = tyre;
            return this;
        }
        public Builder safetyBelt(String safetyBelt) {
          if (safetyBelt == null) throw new NullPointerException("没系安全带，你开个毛车啊");
            this.safetyBelt = safetyBelt;
            return this;
        }
        public Builder engine(String engine) {
            this.engine = engine;
            return this;
        }

        public Builder aimingCircle(String aimingCircle) {
            this.aimingCircle = aimingCircle;
            return this;
        }

        public Builder decoration(String decoration) {
            this.decoration = decoration;
            return this;
        }
        /**
         * 最后创造出实体car
         * @return
         */
        public Car build() {
            return new Car(this);
        }
    }
}
//调用。。
 Car car = new Car.Builder()
                .build();

作者：张少林同学
链接：https://juejin.im/post/6844903746124644365
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

**优点**：

- 解耦，逻辑清晰。统一交由 Builder 类构造，Car 类不用关心内部实现细节，只注重结果。
- 链式调用，使用灵活，易于扩展。相对于方法一中的构造器方法，配置对象属性灵活度大大提高，支持链式调用使得逻辑清晰不少，而且我们需要扩展的时候，也只需要添加对应扩展属性即可，十分方便。

**缺点**：

- 硬要说缺点的话 就是前期需要编写更多的代码，每次构建需要先创建对应的 Builder 对象。但是这点开销几乎可以忽略吧，前期编写更多的代码是为了以后更好的扩展，这不是优秀程序员应该要考虑的事么



## 结构型模式分为以下 7 种：

1. 代理（Proxy）模式：为某对象提供一种代理以控制对该对象的访问。即客户端通过代理间接地访问该对象，从而限制、增强或修改该对象的一些特性。
2. 适配器（Adapter）模式：将一个类的接口转换成客户希望的另外一个接口，使得原本由于接口不兼容而不能一起工作的那些类能一起工作。
3. 桥接（Bridge）模式：将抽象与实现分离，使它们可以独立变化。它是用组合关系代替继承关系来实现的，从而降低了抽象和实现这两个可变维度的耦合度。
4. 装饰（Decorator）模式：动态地给对象增加一些职责，即增加其额外的功能。
5. 外观（Facade）模式：为多个复杂的子系统提供一个一致的接口，使这些子系统更加容易被访问。
6. 享元（Flyweight）模式：运用共享技术来有效地支持大量细粒度对象的复用。
7. 组合（Composite）模式：将对象组合成树状层次结构，使用户对单个对象和组合对象具有一致的访问性。



### 适配器模式（Adapter模式）

#### 适配器模式分类

有三种分类：

- `类适配器` (通过引用适配者进行`组合`实现)
- `对象适配器`(通过`继承`适配者进行实现)
- `接口适配器` （通过`抽象类`来实现适配）



- 在适配器模式中可以定义一个包装类，包装不兼容接口的对象，这个包装类指的就是适配器(Adapter)，它所包装的对象就是适配者(Adaptee)，即被适配的类。

```java
public class Adapter extends Adaptee implements Target {
    public void request() {
        super.doSomething();
    }
}

public class Client {
    public static void main(String[] args) {
        //原有的业务逻辑
        Target target = new ConcreteTarget();
        target.request();
        //现在增加了适配器角色后的业务逻辑
        Target target2 = new Adapter();
        target2.request();
    }
}
```

#### 优点

适配器模式可以让两个没有任何关系的类在一起运行，只要适配器这个角色能够搞定他们就成。

- 增加了类的透明性 : 我们访问的Target目标角色，但是具体的实现都委托给了源角色，而这些对高层次模块是透明的，也是它不需要关心的。

- 提高了类的复用度: 源角色在原有的系统中还是可以正常使用，而在目标角色中也可以充当新的演员

- 灵活性非常好 : 某一天，突然不想要适配器，没问题，删除掉这个适配器就可以了，其他的代码都不用修改，基本上就类似一个灵活的构件，想用就用，不想就卸载。

#### 适用场景：

1、已经存在的类的接口不符合我们的需求；

2、创建一个可以复用的类，使得该类可以与其他不相关的类或不可预见的类（即那些接口可能不一定兼容的类）协同工作；

3、在不对每一个都进行子类化以匹配它们的接口的情况下，使用一些已经存在的子类。



#### **SpringMvc中的适配器（HandlerAdapter）**

首先我们找到前端控制器DispatcherServlet可以把它理解为适配器模式中的Client，它的主要作用在于通过处理映射器（HandlerMapper）来找到相应的Handler（即Controller（宽泛的概念Controller，以及HttpRequestHandler，Servlet，等等）），并执行Controller中相应的方法并返回ModelAndView，

mappedHandler.getHandler()其实就是通过Spring容器中获取到的（宽泛的概念Controller，以及HttpRequestHandler，Servlet，等等）Controller

处理器（宽泛的概念Controller，以及HttpRequestHandler，Servlet，等等）的类型不同，有多重实现方式，那么调用方式就不是确定的，如果需要直接调用Controller方法，需要调用的时候就得不断是使用if else来进行判断是哪一种子类然后执行。那么如果后面要扩展（宽泛的概念Controller，以及HttpRequestHandler，Servlet，等等）Controller，就得修改原来的代码，这样违背了开闭原则（对修改关闭，对扩展开放）。

#### **SpringMvc 是如何使用适配器模式来解决以上问题的呢？**

Spring创建了一个适配器接口（HandlerAdapter）使得每一种处理器（宽泛的概念Controller，以及HttpRequestHandler，Servlet，等等）有一种对应的适配器实现类，让适配器代替（宽泛的概念Controller，以及HttpRequestHandler，Servlet，等等）执行相应的方法。这样在扩展Controller 时，只需要增加一个适配器类就完成了SpringMVC的扩展了



```java

public interface HandlerAdapter {
 
	
	boolean supports(Object handler);
 
	
	ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
 
	
 

```

supports()方法传入处理器（宽泛的概念Controller，以及HttpRequestHandler，Servlet，等等）判断是否与当前适配器支持如果支持则从DispatcherServlet中的HandlerAdapter实现类中返回支持的适配器实现类。handler方法就是代理Controller来执行请求的方法并返回结果。

在DispatchServlert中的doDispatch方法中

HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

此代码通过调用DispatchServlert 中getHandlerAdapter传入Controller（宽泛的概念Controller，以及HttpRequestHandler，Servlet，等等），来获取对应的HandlerAdapter 的实现子类，从而做到使得每一种Controller有一种对应的适配器实现类

![image-20200907204532120](/Users/ano/Desktop/notes/image/image-20200907204532120.png)

mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

其中一个处理器适配器就是我们常说的最熟悉的Controller类适配器

### 桥接模式

桥接模式，是结构型的设计模式之一。桥接模式基于类的最小设计原则，通过使用封装，聚合以及继承等行为来让不同的类承担不同的责任。它的主要特点是把抽象（abstraction）与行为实现（implementation）分离开来，从而可以保持各部分的独立性以及应对它们的功能扩展。

![image-20200907214916981](/Users/ano/Desktop/notes/image/image-20200907214916981.png)

优点：

- 分离抽象接口及其实现部分。桥接模式使用“对象间的关联关系”解耦了抽象和实现之间固有的绑定关系，使得抽象和实现可以沿着各自的维度来变化。所谓抽象和实现沿着各自维度的变化，也就是说抽象和实现不再在同一个继承层次结构中，而是“子类化”它们，使它们各自都具有自己的子类，以便任何组合子类，从而获得多维度组合对象。
- 在很多情况下，桥接模式可以取代多层继承方案，多层继承方案违背了“单一职责原则”，复用性较差，且类的个数非常多，桥接模式是比多层继承方案更好的解决方法，它极大减少了子类的个数。
- 桥接模式提高了系统的可扩展性，在两个变化维度中任意扩展一个维度，都不需要修改原有系统，符合“开闭原则”。

缺点：

- 桥接模式的使用会增加系统的理解与设计难度，由于关联关系建立在抽象层，要求开发者一开始就针对抽象层进行设计与编程。

### 组合模式

组合模式（Composite Pattern），又叫部分整体模式，是用于把一组相似的对象当作一个单一的对象。组合模式依据树形结构来组合对象，用来表示部分以及整体层次。这种类型的设计模式属于结构型模式，它创建了对象组的树形结构。

![image-20200907215145128](/Users/ano/Desktop/notes/image/image-20200907215145128.png)

### 装饰器模式

装饰器模式（Decorator Pattern）允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。

![image-20200907215239767](/Users/ano/Desktop/notes/image/image-20200907215239767.png)



### 外观模式

外观模式（Facade Pattern）隐藏系统的复杂性，并向客户端提供了一个客户端可以访问系统的接口。这种类型的设计模式属于结构型模式，它向现有的系统添加一个接口，来隐藏系统的复杂性。

![image-20200907215325597](/Users/ano/Desktop/notes/image/image-20200907215325597.png)



### 享元模式

享元模式（Flyweight Pattern）主要用于减少创建对象的数量，以减少内存占用和提高性能。这种类型的设计模式属于结构型模式，它提供了减少对象数量从而改善应用所需的对象结构的方式。

![image-20200907215507440](/Users/ano/Desktop/notes/image/image-20200907215507440.png)

### 代理模式

在代理模式（Proxy Pattern）中，一个类代表另一个类的功能。这种类型的设计模式属于结构型模式。

#### jdk动态代理

```java
	
	  public static void main(String[] args) throws IllegalAccessException, InstantiationException {
	        // 1. 创建被代理的对象，UserService接口的实现类
	        UserServiceImpl userServiceImpl = new UserServiceImpl();
	        // 2. 获取对应的 ClassLoader
	        ClassLoader classLoader = userServiceImpl.getClass().getClassLoader();
	        // 3. 获取所有接口的Class，这里的UserServiceImpl只实现了一个接口UserService，
	        Class[] interfaces = userServiceImpl.getClass().getInterfaces();
	        // 4. 创建一个将传给代理类的调用请求处理器，处理所有的代理对象上的方法调用
	        //     这里创建的是一个自定义的日志处理器，须传入实际的执行对象 userServiceImpl
	        InvocationHandler logHandler = new LogHandler(userServiceImpl);
	        /*
			   5.根据上面提供的信息，创建代理对象 在这个过程中，
	               a.JDK会通过根据传入的参数信息动态地在内存中创建和.class 文件等同的字节码
	               b.然后根据相应的字节码转换成对应的class，
	               c.然后调用newInstance()创建代理实例
			 */
	        UserService proxy = (UserService) Proxy.newProxyInstance(classLoader, interfaces, logHandler);
	        // 调用代理的方法
	        proxy.select();
	        proxy.update();
	    }
```



```java
public class LogHandler implements InvocationHandler{
    Object target;  // 被代理的对象，实际的方法执行者
    public LogHandler(Object target) {
        this.target = target;
    }
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
			    before();
	        Object result = method.invoke(target, args);  // 调用 target 的 method 方法
	        after();
	        return result;  // 返回方法的执行结果
	}
	 // 调用invoke方法之前执行
    private void before() {
        System.out.println(String.format("log start time [%s] ", new Date()));
    }
    // 调用invoke方法之后执行
    private void after() {
        System.out.println(String.format("log end time [%s] ", new Date()) );
    }
}

```



生成的xxxxProxy 继承了 Proxy 类，并且实现了被代理的所有接口，以及equals、hashCode、toString等方法，由于 xxxProxy 继承了 Proxy 类，所以每个代理类都会关联一个 InvocationHandler 方法调用处理器

类和所有方法都被 `public final` 修饰，所以代理类只可被使用，不可以再被继承，每个方法都有一个 Method 对象来描述，Method 对象在static静态代码块中创建，以 `m + 数字` 的格式命名

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;
import proxy.UserService;

public final class UserServiceProxy extends Proxy implements UserService {
    private static Method m1;
    private static Method m2;
    private static Method m4;
    private static Method m0;
    private static Method m3;

    public UserServiceProxy(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        // 省略...
    }

    public final String toString() throws  {
        // 省略...
    }

    public final void select() throws  {
        try {
            super.h.invoke(this, m4, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws  {
        // 省略...
    }

    public final void update() throws  {
        try {
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m4 = Class.forName("proxy.UserService").getMethod("select");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
            m3 = Class.forName("proxy.UserService").getMethod("update");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}

作者：小旋锋
链接：https://juejin.im/post/6844903744954433544
```



调用方法的时候通过 `super.h.invoke(this, m1, (Object[])null);` 调用，其中的 `super.h.invoke` 实际上是在创建代理的时候传递给 `Proxy.newProxyInstance` 的 xxxHandler 对象，它继承 InvocationHandler 类，负责实际的调用处理逻辑。

​      ![image-20200705231529903](/Users/ano/Desktop/notes/image/image-20200705231529903.png)



#### CGLIB动态代理

```java
public class Intercepter implements MethodInterceptor{
	@Override
	public Object intercept(Object object, Method arg1, Object[] objects, MethodProxy methodProxy) throws Throwable {
		before();
        Object result = methodProxy.invokeSuper(object, objects);   // 注意这里是调用 invokeSuper 而不是 invoke，否则死循环，methodProxy.invokesuper执行的是原始类的方法，method.invoke执行的是子类的方法
        after();
        return result;
	}
	private void before() {
		System.out.println(String.format("log start time [%s] ", new Date()));
	}
	private void after() {
		System.out.println(String.format("log end time [%s] ", new Date()));
	}
}
```

```java
public class Client3 {
	public static void main(String[] args) {
		Intercepter proxy = new Intercepter();
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(UserServiceImpl.class); // 设置超类，cglib是通过继承来实现的
		enhancer.setCallback(proxy); //设置拦截
		UserService dao = (UserService) enhancer.create(); // 创建代理类
		dao.update();
		dao.select();
	}
}
```

#### CGLIB 创建动态代理类的模式是：

1. 查找目标类上的所有非final 的public类型的方法定义；
2. 将这些方法的定义转换成字节码；
3. 将组成的字节码转换成相应的代理的class对象；
4. 实现 MethodInterceptor接口，用来处理对代理类上所有方法的请求

Diff : 

* cglib动态代理：基于ASM机制实现，通过生成业务类的子类作为代理类。

* 基于类似 cglib 框架的优势：

  - 无需实现接口，达到代理类无侵入

  - 只操作我们关心的类，而不必为其他相关类增加工作量。

  - 高性能

    

* JDK动态代理：基于Java反射机制实现，必须要实现了接口的业务类才能用这种办法生成代理对象。

* JDK Proxy 的优势：

  - 最小化依赖关系，减少依赖意味着简化开发和维护，JDK 本身的支持，可能比 cglib 更加可靠。
  - 平滑进行 JDK 版本升级，而字节码类库通常需要进行更新以保证在新版 Java 上能够使用。
  - 代码实现简单。

  

  

#### 描述动态代理的几种实现方式？分别说出相应的优缺点

代理可以分为 "静态代理" 和 "动态代理"，动态代理又分为 "JDK动态代理" 和 "CGLIB动态代理" 实现。

**静态代理**：代理对象和实际对象都继承了同一个接口，在代理对象中指向的是实际对象的实例，这样对外暴露的是代理对象而真正调用的是 Real Object

- **优点**：可以很好的保护实际对象的业务逻辑对外暴露，从而提高安全性。
- **缺点**：不同的接口要有不同的代理类实现，会很冗余

**JDK 动态代理**：

- 为了解决静态代理中，生成大量的代理类造成的冗余；
- JDK 动态代理只需要实现 InvocationHandler 接口，重写 invoke 方法便可以完成代理的实现，
- jdk的代理是利用反射生成代理类 Proxyxx.class 代理类字节码，并生成对象
- jdk动态代理之所以**只能代理接口**是因为**代理类本身已经extends了Proxy，而java是不允许多重继承的**，但是允许实现多个接口
- **优点**：解决了静态代理中冗余的代理实现类问题。
- **缺点**：JDK 动态代理是基于接口设计实现的，如果没有接口，会抛异常。

**CGLIB 代理**：

- 由于 JDK 动态代理限制了只能基于接口设计，而对于没有接口的情况，JDK方式解决不了；
- CGLib 采用了非常底层的字节码技术，其原理是通过字节码技术为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑，来完成动态代理的实现。
- 实现方式实现 MethodInterceptor 接口，重写 intercept 方法，通过 Enhancer 类的回调方法来实现。
- 但是CGLib在创建代理对象时所花费的时间却比JDK多得多，所以对于单例的对象，因为无需频繁创建对象，用CGLib合适，反之，使用JDK方式要更为合适一些。
- 同时，由于CGLib由于是采用动态创建子类的方法，对于final方法，无法进行代理。
- **优点**：没有接口也能实现动态代理，而且采用字节码增强技术，性能也不错。
- **缺点**：技术实现相对难理解

## 行为型模式

行为型模式(Behavioral Pattern)是对在不同的对象之间划分责任和算法的抽象化。

行为型模式不仅仅关注类和对象的结构，而且重点关注它们之间的相互作用。

通过行为型模式，可以更加清晰地划分类与对象的职责，并研究系统在运行时实例对象 之间的交互。在系统运行时，对象并不是孤立的，它们可以通过相互通信与协作完成某些复杂功能，一个对象在运行时也将影响到其他对象的运行。

行为型模式分为类行为型模式和对象行为型模式两种：

- 类行为型模式：类的行为型模式使用继承关系在几个类之间分配行为，类行为型模式主要通过多态等方式来分配父类与子类的职责。
- 对象行为型模式：对象的行为型模式则使用对象的聚合关联关系来分配行为，对象行为型模式主要是通过对象关联等方式来分配两个或多个类的职责。根据“合成复用原则”，系统中要尽量使用关联关系来取代继承关系，因此大部分行为型设计模式都属于对象行为型设计模式。

### 责任链模式

责任链模式(Chain of Responsibility Pattern)是一种是行为型设计模式，它为请求创建了一个处理对象的链。其链中每一个节点都看作是一个对象，每个节点处理的请求均不同，且内部自动维护一个下一节点对象。当一个请求从链式的首端发出时，会沿着链的路径依次传递给每一个节点对象，直至有对象处理这个请求为止。责任链模式主要解决了发起请求和具体处理请求的过程解耦，职责链上的处理者负责处理请求，用户只需将请求发送到职责链上即可，无需关心请求的处理细节和请求的传递。


链接：https://www.jianshu.com/p/11274d8f8870

### 命令模式

命令模式（Command Pattern）是一种数据驱动的设计模式，它属于行为型模式。请求以命令的形式包裹在对象中，并传给调用对象。调用对象寻找可以处理该命令的合适的对象，并把该命令传给相应的对象，该对象执行命令。

### 观察者模式

当对象间存在一对多关系时，则使用观察者模式（Observer Pattern）。比如，当一个对象被修改时，则会自动通知依赖它的对象。观察者模式属于行为型模式。

### 策略模式

在策略模式（Strategy Pattern）中，一个类的行为或其算法可以在运行时更改。这种类型的设计模式属于行为型模式。

在策略模式中，我们创建表示各种策略的对象和一个行为随着策略对象改变而改变的 context 对象。策略对象改变 context 对象的执行算法。





# Spring

## springMVC

doDispatch src 

```java

protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;
 
		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
 
		try {
			ModelAndView mv = null;
			Exception dispatchException = null;
 
			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);
 
				// 此处通过HandlerMapping来映射Controller
				mappedHandler = getHandler(processedRequest);
				if (mappedHandler == null || mappedHandler.getHandler() == null) {
					noHandlerFound(processedRequest, response);
					return;
				}
 
				// 获取适配器
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
 
				// Process last-modified header, if supported by the handler.
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (logger.isDebugEnabled()) {
						logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
					}
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}
 
				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}
 
				// 通过适配器调用controller的方法并返回ModelAndView
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
 
				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}
 
				applyDefaultViewName(processedRequest, mv);
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				// As of 4.3, we're processing Errors thrown from handler methods as well,
				// making them available for @ExceptionHandler methods and other scenarios.
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
			triggerAfterCompletion(processedRequest, response, mappedHandler,
					new NestedServletException("Handler processing failed", err));
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// Clean up any resources used by a multipart request.
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
	}
```



![image-20200907205128821](/Users/ano/Desktop/notes/image/image-20200907205128821.png) 

1.DispatcherServlet：前端控制器。用户请求到达前端控制器，它就相当于mvc模式中的c，dispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求，dispatcherServlet的存在降低了组件之间的耦合性,系统扩展性提高。由框架实现
 2.HandlerMapping：处理器映射器。HandlerMapping负责根据用户请求的url找到Handler即处理器，springmvc提供了不同的映射器实现不同的映射方式，根据一定的规则去查找,例如：xml配置方式，实现接口方式，注解方式等。由框架实现
 3.Handler：处理器。Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。由于Handler涉及到具体的用户业务请求，所以一般情况需要程序员根据业务需求开发Handler。
 4.HandlAdapter：处理器适配器。通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。由框架实现。
 5.ModelAndView是springmvc的封装对象，将model和view封装在一起。
 6.ViewResolver：视图解析器。ViewResolver负责将处理结果生成View视图，ViewResolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。
 7View:是springmvc的封装对象，是一个接口, springmvc框架提供了很多的View视图类型，包括：jspview，pdfview,jstlView、freemarkerView、pdfView等。一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由程序员根据业务需求开发具体的页面。



作者：CoderZS
链接：https://www.jianshu.com/p/8a20c547e245
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。







# 背问题

## 从URL输入到页面展现的过程

从URL的输入到页面的展现，大概会经过以下几个过程：

1. 在浏览器中输入url
2. DNS域名解析
3. 服务器处理请求
4. 网站处理流程
5. 浏览器显示页面信息

### 一 在浏览器中输入url

#### **URL（Uniform Resource Locator,统一资源定位符）**

由一串简单的文本字符组成，他的作用是用于定位互联网上资源的地址。URL通常由以下几部分组成：**传输协议**、**域名**、**端口**、**文件路径**。

### 二 DNS域名解析**

**DNS（Domain Name System，域名系统）**

主要进行将主机名和域名转换为IP地址的工作。把便于用户记忆的特定域名转换成为能够被机器直接读取的IP数串。IP地址与域名并不是一对一的映射关系，一个域名后面可以对应多台设备的IP地址，一个IP地址也可以绑定多个域名。如果直接使用IP地址的话，可能无法准确的定位你想要访问的网站。

在同样的局域网下，如果知道对方的IP就可以进行访问，公网IP是处于整个互联网可访问的状态中。

**域名解析的流程：**

1. **查找浏览器缓存**——我们日常浏览网站时，浏览器会缓存DNS记录一段时间。如果以前我们访问过该网站，那么在浏览器中就会有相应的缓存记录。因此，我们输入网址后，浏览器会首先检查缓存中是否有该域名对应的IP信息。如果有，则直接返回该信息供用户访问网站，如果查询失败，会从系统缓存中进行查找。
2. **查找系统缓存**——从hosts文件中查找是否有存储的DNS信息（MAC端，可在“终端”中输入命令cat etc/hosts找到hosts文件位置），如果查询失败，可从路由器缓存中继续查找。
3. **查找路由器缓存**——如果之前访问过相应的网站，一般路由器也会缓存信息。如果查询失败，可继续从 ISP DNS 缓存查找。
4. **查找ISP DNS缓存**——从网络服务商（例如电信）的DNS缓存信息中查找。
5. 如果经由以上方式都没找到对应IP的话，则向**根域名服务器查找**域名对应的IP地址，根域名服务器把请求转发到下一级，逐层查找该域名的对应数据，直到获得最终解析结果或失败的相应。

**根域名服务器，**根服务器主要用来管理互联网的主目录。是互联网域名解析系统（DNS）中最高级别的域名服务器。

如果本地DNS服务器本地区域文件与缓存解析都失效，则根据本地DNS服务器的设置（是否设置转发器）进行查询，如果未用转发模式，本地DNS就把请求发至13台根DNS，根DNS服务器收到请求后会判断这个域名(.com)是谁来授权管理，并会返回一个负责该顶级域名服务器的一个IP。本地DNS服务器收到IP信息后，将会联系负责.com域的这台服务器。这台负责.com域的服务器收到请求后，如果自己无法解析，它就会找一个管理.com域的下一级DNS服务器地址([http://qq.com](https://link.zhihu.com/?target=http%3A//qq.com))给本地DNS服务器。当本地DNS服务器收到这个地址后，就会找[http://qq.com](https://link.zhihu.com/?target=http%3A//qq.com)域服务器，重复上面的动作，进行查询，直至找到www  . qq  .com主机。



作者：Marlous
链接：https://www.zhihu.com/question/23042131/answer/24922954
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



**1）** **浏览器缓存**

　　当用户通过浏览器访问某域名时，浏览器首先会在自己的缓存中查找是否有该域名对应的IP地址（若曾经访问过该域名且没有清空缓存便存在）；

**2）** **系统缓存**

　　当浏览器缓存中无域名对应IP则会自动检查用户计算机系统Hosts文件DNS缓存是否有该域名对应IP；

**3）** **路由器缓存**

　　当浏览器及系统缓存中均无域名对应IP则进入路由器缓存中检查，以上三步均为客服端的DNS缓存；

**4）** **ISP****（互联网服务提供商）DNS****缓存**

　　当在用户客服端查找不到域名对应IP地址，则将进入ISP DNS缓存中进行查询。比如你用的是电信的网络，则会进入电信的DNS缓存服务器中进行查找；

**5）** **根域名服务器**

　　当以上均未完成，则进入根服务器进行查询。全球仅有13台根域名服务器，1个主根域名服务器，其余12为辅根域名服务器。根域名收到请求后会查看区域文件记录，若无则将其管辖范围内顶级域名（如.com）服务器IP告诉本地DNS服务器；

**6）** **顶级域名服务器**

　　顶级域名服务器收到请求后查看区域文件记录，若无则将其管辖范围内主域名服务器的IP地址告诉本地DNS服务器；

**7）** **主域名服务器**

　　主域名服务器接受到请求后查询自己的缓存，如果没有则进入下一级域名服务器进行查找，并重复该步骤直至找到正确纪录；

**8****）保存结果至缓存**

　　本地域名服务器把返回的结果保存到缓存，以备下一次使用，同时将该结果反馈给客户端，客户端通过这个IP地址与web服务器建立链接。



#### **什么是DNS劫持？**

DNS劫持（Domain Name System，域名劫持），是指在劫持的网络范围内拦截域名解析的请求，分析请求的域名，把审查范围以外的请求放行，返回假的IP地址或者什么都不做使请求失去响应，其效果就是对特定的网络不能访问或访问的是假网址。简单来说，就是你输入的是知乎的网址，但是却跳转到了百度的页面。

#### 怎么办？

手动修改DNS，自己手动调整dns服务器，

修改路由器默认密码 

### 三 服务器处理请求**

浏览器通过IP地址找到对应的服务器，要求建立TCP链接，此时服务器开始处理用户请求。



#### **服务器是什么？**

服务器是一台安装系统的机器。常见的系统有Linux、Windows Server2012等。系统里安装的处理请求的应用叫Web server。



#### **服务器如何处理请求？**

由服务器上安装的处理请求的应用（Web Server）来处理。
常见的Web服务器有：Apache、Nginx、IIS、Lighttpd。
Web服务器接收用户的Requset交给网站代码或反向代理到其他服务器。



#### **TCP是什么？**

TCP是互联网中的传输层协议，提供可靠的链接服务，采用三次握手确认一个连接：

1. 浏览器向服务器发送建立连接的请求。
2. 服务器接收到浏览器发送的请求后，想浏览器发送统一连接的信号。
3. 浏览器接受到服务器发出的同意连接的信号后，再次向服务器发出确认连接的信号。

当三次握手返程，TCP客户端和服务端成功的建立连接，就可以开始传输数据了。



### **四、网站处理流程**

用户输入网址后向服务器发送内容请求，服务器接收到请求后触发Controller（控制器），控制器从Model（模型）和视图（View）中获取各种数据信息进行处理，最后视图（View）将数据渲染为HTML使得页面完整的呈献给用户。

#### **MVC是什么**

MVC是一种设计模式，全名是Model View Controller，是模型（Model）- 视图（View）- 控制器（Controller）的缩写。

- **Model（模型）**
  是应用程序中用于处理应用程序数据逻辑的部分。
  通常模型对象负责在数据苦衷存取数据。
- **View（视图）**
  是应用程序中处理数据显示的部分。
  通常视图是一句模型数据创建的。
  而这一部分也是我们前端工作中很重要的一项内容。
- **Controller（控制器）**
  是应用程序中处理用户交互的部分。
  通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据。



### **五、浏览器显示页面信息**

- HTML字符串被浏览器接收后被一句句读取解析。
- 解析到link标签后重新发送请求获取CSS。
- 解析到script标签后发送请求获取JS，并执行代码。
- 解析到img标签后发送请求获取图片资源。
- 浏览器根据HTML和CSS计算得到渲染书，绘制到屏幕上。
- JS会被执行，页面展现。



# MYBATIS

https://juejin.im/entry/59127ad4da2f6000536f64d



## 

# JVM

### 内存区域划分 8

#### Q1：运行时数据区是什么？

虚拟机在执行 Java 程序的过程中会把它所管理的内存划分为若干不同的数据区，这些区域有各自的用途、创建和销毁时间。

线程私有：程序计数器、Java 虚拟机栈、本地方法栈。

线程共享：Java 堆、方法区。

------

#### Q2：程序计数器是什么？

**程序计数器**是一块较小的内存空间，可以看作当前线程所执行字节码的行号指示器。字节码解释器工作时通过改变计数器的值选取下一条执行指令。分支、循环、跳转、线程恢复等功能都需要依赖计数器完成。是唯一在虚拟机规范中没有规定内存溢出情况的区域。

如果线程正在执行 Java 方法，计数器记录正在执行的虚拟机字节码指令地址。如果是本地方法，计数器值为 Undefined。

```java
public class Test {
    private void run() {
        List<String> list = new ArrayList<>(); // invokespecial 构造器调用
        list.add("a"); // invokeinterface 接口调用 
        ArrayList<String> arrayList = new ArrayList<>(); // invokespecial 构造器调用
        arrayList.add("b"); // invokevirtual 虚函数调用
    }
    public static void main(String[] args) {
        Test test = new Test(); // invokespecial 构造器调用
        test.run(); // invokespecial 私有函数调用
    }
}
————————————————
版权声明：本文为CSDN博主「TellH」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/TellH/java/article/details/77370223
```

垃圾回收

4种算法： 

好多垃圾回收器

方法区

![image-20200809220615632](/Users/ano/Desktop/notes/image/image-20200809220615632.png)



![image-20200809220822476](/Users/ano/Desktop/notes/image/image-20200809220822476.png)



![image-20200809220839717](/Users/ano/Desktop/notes/image/image-20200809220839717.png)

JVM 深入了解java 虚拟机 Page 39 -- 49 jvm结构

![image-20200809170725658](/Users/ano/Desktop/notes/image/image-20200809170725658.png)

![image-20200809170812538](/Users/ano/Desktop/notes/image/image-20200809170812538.png)

# JAVA 

## ClassNotFoundException 和NoClassDefFoundError的区别

ClassNotFoundException是一个检查异常。从类继承层次上来看，ClassNotFoundException是从Exception继承的，所以ClassNotFoundException是一个检查异常。当应用程序运行的过程中尝试使用类加载器去加载Class文件的时候，如果没有在classpath中查找到指定的类，就会抛出ClassNotFoundException。一般情况下，当我们使用Class.forName()或者ClassLoader.loadClass以及使用ClassLoader.findSystemClass()在运行时加载类的时候，如果类没有被找到，那么就会导致JVM抛出ClassNotFoundException。

　　最简单的，当我们使用JDBC去连接数据库的时候，我们一般会使用Class.forName()的方式去加载JDBC的驱动，如果我们没有将驱动放到应用的classpath下，那么会导致运行时找不到类，所以运行Class.forName()会抛出ClassNotFoundException。



## NoClassDefFoundError

　　NoClassDefFoundError异常，看命名后缀是一个Error。从类继承层次上看，NoClassDefFoundError是从Error继承的。和ClassNotFoundException相比，明显的一个区别是，NoClassDefFoundError并不需要应用程序去关心catch的问题。

　　当JVM在加载一个类的时候，如果这个类在编译时是可用的，但是在运行时找不到这个类的定义的时候，JVM就会抛出一个NoClassDefFoundError错误。比如当我们在new一个类的实例的时候，如果在运行是类找不到，则会抛出一个NoClassDefFoundError的错误

<img src="/Users/ano/Desktop/notes/image/image-20200912192729397.png" alt="image-20200912192729397" style="zoom:50%;" />

### Synchronized同步静态方法和非静态方法？

**Synchronized修饰非静态方法，实际上是对调用该方法的对象加锁，俗称“对象锁”。**

**Synchronized修饰静态方法，实际上是对该类对象加锁，俗称“类锁”。**



```
1.对象锁钥匙只能有一把才能互斥，才能保证共享变量的唯一性

2.在静态方法上的锁，和 实例方法上的锁，默认不是同样的，如果同步需要制定两把锁一样。

3.关于同一个类的方法上的锁，来自于调用该方法的对象，如果调用该方法的对象是相同的，那么锁必然相同，否则就不相同。比如 new A().x() 和 new A().x(),对象不同，锁不同，如果A的单利的，就能互斥。

4.静态方法加锁，能和所有其他静态方法加锁的 进行互斥

5.静态方法加锁，和xx.class 锁效果一样，直接属于类的
```

### JAVA 中有以下几个『元注解』：

- @Target：注解的作用目标

  ElementType.TYPE：允许被修饰的注解作用在类、接口和枚举上

  ElementType.FIELD：允许作用在属性字段上

  ElementType.METHOD：允许作用在方法上

  ElementType.PARAMETER：允许作用在方法参数上

  ElementType.CONSTRUCTOR：允许作用在构造器上

  ElementType.LOCAL_VARIABLE：允许作用在本地局部变量上

  ElementType.ANNOTATION_TYPE：允许作用在注解上

  ElementType.PACKAGE：允许作用在包上

  

- @Retention：注解的生命周期

  RetentionPolicy.SOURCE：当前注解编译期可见，不会写入 class 文件

  RetentionPolicy.CLASS：类加载阶段丢弃，会写入 class 文件

  RetentionPolicy.RUNTIME：永久保存，可以反射获取

  ​	

- @Documented：注解是否应当被包含在 JavaDoc 文档中

- @Inherited：是否允许子类继承该注解

### ConcurrentHashMap put 方法

1. 检查key ， value是否为null ， null 直接抛出异常
2. 计算hashcode，
3. 检查数组是否初始化，没有就调用initTable
4. 检查table【0】 是否为null， null 则直接用 cas 设置为new Node（k，v）
5. 如果当前位置的 `hashcode == MOVED == -1`,则需要进行扩容。
6. 如果都不满足，则利用 synchronized 锁写入数据
7. 如果数量大于 `TREEIFY_THRESHOLD` 则要转换为红黑树。



### get 方法

- 根据计算出来的 hashcode 寻址，如果就在桶上那么直接返回值。
- 如果是红黑树那就按照树的方式获取值。
- 就不满足那就按照链表的方式遍历获取值。
- *//hash值为负值表示正在扩容，这个时候查的是ForwardingNode的find方法来定位到nextTable来*        *//eh=-1，说明该节点是一个ForwardingNode，正在迁移，此时调用ForwardingNode的find方法去nextTable里找。*        *//eh=-2，说明该节点是一个TreeBin，此时调用TreeBin的find方法遍历红黑树，由于红黑树有可能正在旋转变色，所以find里会有读写锁。*        *//eh>=0，说明该节点下挂的是一个链表，直接遍历该链表即可。*

### 多态的原理

多态允许具体访问时实现方法的动态绑定。Java对于动态绑定的实现主要依赖于方法表，通过继承和接口的多态实现有所不同。



### Executor生命周期

关闭线程池可能出现一下几种情况：

1、平缓关闭：已经启动的任务全部执行完毕，同时不再接受新的任务

2、立即关闭：取消所有正在执行和未执行的任务

shutdown()是一个平缓的关闭过程，线程池停止接受新的任务，同时等待已经提交的任务执行完毕，包括那些进入队列还没有开始的任务，这时候线程池处于SHUTDOWN状态；shutdownNow()是一个立即关闭过程，线程池停止接受新的任务，同时线程池取消所有执行的任务和已经进入队列但是还没有执行的任务，这时候线程池处于STOP状态。

一旦shutdown()或者shutdownNow()执行完毕，线程池就进入TERMINATED状态，此时线程池就结束了。



```
这几个状态的转化关系为：

1、调用shundown()方法线程池的状态由RUNNING——>SHUTDOWN

2、调用shutdowNow()方法线程池的状态由RUNNING——>STOP

3、当任务队列和线程池均为空的时候 线程池的状态由STOP/SHUTDOWN——–>TIDYING

4、当terminated()方法被调用完成之后，线程池的状态由TIDYING———->TERMINATED状态
```



![image-20200903210041831](/Users/ano/Desktop/notes/image/image-20200903210041831.png)



# JUC

# AQS

https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html

何时出队列？”和“如何出队列？

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

final boolean acquireQueued(final Node node, int arg) {
	// 标记是否成功拿到资源
	boolean failed = true;
	try {
		// 标记等待过程中是否中断过
		boolean interrupted = false;
		// 开始自旋，要么获取锁，要么中断
		for (;;) {
			// 获取当前节点的前驱节点
			final Node p = node.predecessor();
			// 如果p是头结点，说明当前节点在真实数据队列的首部，就尝试获取锁（别忘了头结点是虚节点）
			if (p == head && tryAcquire(arg)) {
				// 获取锁成功，头指针移动到当前node
				setHead(node);
				p.next = null; // help GC
				failed = false;
				return interrupted;
			}
			// 说明p为头节点且当前没有获取到锁（可能是非公平锁被抢占了）或者是p不为头结点，这个时候就要判断当前node是否要被阻塞（被阻塞条件：前驱节点的waitStatus为-1），防止无限循环浪费资源。具体两个方法下面细细分析
			if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
				interrupted = true;
		}
	} finally {
		if (failed)
			cancelAcquire(node);
	}
}
```

```java
private void setHead(Node node) {
	head = node;
	node.thread = null;
	node.prev = null;
}

// java.util.concurrent.locks.AbstractQueuedSynchronizer

// 靠前驱节点判断当前线程是否应该被阻塞
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
	// 获取头结点的节点状态
	int ws = pred.waitStatus;
	// 说明头结点处于唤醒状态
	if (ws == Node.SIGNAL)
		return true; 
	// 通过枚举值我们知道waitStatus>0是取消状态
	if (ws > 0) {
		do {
			// 循环向前查找取消节点，把取消节点从队列中剔除
			node.prev = pred = pred.prev;
		} while (pred.waitStatus > 0);
		pred.next = node;
	} else {
		// 设置前任节点等待状态为SIGNAL
		compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
	}
	return false;
}
```



```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

public final boolean release(int arg) {
	// 上边自定义的tryRelease如果返回true，说明该锁没有被任何线程持有
	if (tryRelease(arg)) {
		// 获取头结点
		Node h = head;
		// 头结点不为空并且头结点的waitStatus不是初始化节点情况，解除线程挂起状态
		if (h != null && h.waitStatus != 0)
			unparkSuccessor(h);
		return true;
	}
	return false;
}
```

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private void unparkSuccessor(Node node) {
	// 获取头结点waitStatus
	int ws = node.waitStatus;
	if (ws < 0)
		compareAndSetWaitStatus(node, ws, 0);
	// 获取当前节点的下一个节点
	Node s = node.next;
	// 如果下个节点是null或者下个节点被cancelled，就找到队列最开始的非cancelled的节点
	if (s == null || s.waitStatus > 0) {
		s = null;
		// 就从尾部节点开始找，到队首，找到队列第一个waitStatus<0的节点。
		for (Node t = tail; t != null && t != node; t = t.prev)
			if (t.waitStatus <= 0)
				s = t;
	}
	// 如果当前节点的下个节点不为空，而且状态<=0，就把当前节点unpark
	if (s != null)
		LockSupport.unpark(s.thread);
}
```



1. ReentrantReadWriteLock 

   **同步状态的低16位用来表示写锁的获取次数**

   **同步状态的高16位用来表示读锁被获取的次数**

   ​      Write Lock  加锁： 

   1. 有共享锁/有独占锁： 直接返还错误

   2. 独占是自己： 重入
   3. CAS 设置同步状态

   ```java
   protected final boolean tryAcquire(int acquires) {
       /*
        * Walkthrough:
        * 1. If read count nonzero or write count nonzero
        *    and owner is a different thread, fail.
        * 2. If count would saturate, fail. (This can only
        *    happen if count is already nonzero.)
        * 3. Otherwise, this thread is eligible for lock if
        *    it is either a reentrant acquire or
        *    queue policy allows it. If so, update state
        *    and set owner.
        */
       Thread current = Thread.currentThread();
   	// 1. 获取写锁当前的同步状态
       int c = getState();
   	// 2. 获取写锁获取的次数
       int w = exclusiveCount(c);
       if (c != 0) {
           // (Note: if c != 0 and w == 0 then shared count != 0)
   		// 3.1 当读锁已被读线程获取或者当前线程不是已经获取写锁的线程的话
   		// 当前线程获取写锁失败
           if (w == 0 || current != getExclusiveOwnerThread())
               return false;
           if (w + exclusiveCount(acquires) > MAX_COUNT)
               throw new Error("Maximum lock count exceeded");
           // Reentrant acquire
   		// 3.2 当前线程获取写锁，支持可重复加锁
           setState(c + acquires);
           return true;
       }
   	// 3.3 写锁未被任何线程获取，当前线程可获取写锁
       if (writerShouldBlock() ||
           !compareAndSetState(c, c + acquires))
           return false;
       setExclusiveOwnerThread(current);
       return true;
   }
   ```

   释放

   ```java
   protected final boolean tryRelease(int releases) {
       if (!isHeldExclusively())
           throw new IllegalMonitorStateException();
   	//1. 同步状态减去写状态
       int nextc = getState() - releases;
   	//2. 当前写状态是否为0，为0则释放写锁
       boolean free = exclusiveCount(nextc) == 0;
       if (free)
           setExclusiveOwnerThread(null);
   	//3. 不为0则更新同步状态
       setState(nextc);
       return free;
   }
   
   ```

   只需要用**当前同步状态直接减去写状态的原因正是我们刚才所说的写状态是由同步状态的低16位表示的**。

   ## 读锁的获取

   \1. If write lock held by another thread, fail. *

   \ 2. Otherwise, this thread is eligible for * lock wrt state, so ask if it should block * because of queue policy. If not, try * to grant by CASing state and updating count. Note that step does not check for reentrant * acquires, which is postponed to full version * to avoid having to check hold count in * the more typical non-reentrant case. * 

   \3. If step 2 fails either because thread * apparently not eligible or CAS fails or count * saturated, chain to version with full retry loop.

   

2. 

3. 

https://zhuanlan.zhihu.com/p/137624149

https://juejin.im/post/5ae755606fb9a07ab97942a4

# 算法

1. 洗牌算法，**设计一个公平的洗牌算法**

    

   ```pseudocode
   -- To shuffle an array a of n elements (indices 0..n-1):
   for i from n − 1 downto 1 do
        j ← random integer such that 0 ≤ j ≤ i
        exchange a[j] and a[i]
   ```

2. 哈夫曼编码

   https://blog.csdn.net/FX677588/article/details/70767446





# 操作系统

上下文切换： https://juejin.im/post/5c872a255188257e3f1afaef



## 以下主要说明Swap机制：

在Linux下，SWAP的作用类似Windows系统下的“虚拟内存”。当物理内存不足时，拿出部分硬盘空间当SWAP分区（虚拟成内存）使用，从而解决内存容量不足的情况。

SWAP意思是交换，顾名思义，**当某进程向OS请求内存发现不足时，OS会把内存中暂时不用的数据交换出去，放在SWAP分区中，这个过程称为SWAP OUT**。**当某进程又需要这些数据且OS发现还有空闲物理内存时，又会把SWAP分区中的数据交换回物理内存中，这个过程称为SWAP IN**。

当然，swap大小是有上限的，一旦swap使用完，操作系统会触发OOM-Killer机制，把消耗内存最多的进程kill掉以释放内存。





## kill pid与kill -9 pid的区别

​        kill pid的作用是向进程号为pid的进程发送SIGTERM（这是kill默认发送的信号），该信号是一个结束进程的信号且可以被应用程序捕获。若应用程序没有捕获并响应该信号的逻辑代码，则该信号的默认动作是kill掉进程。这是终止指定进程的推荐做法。
​        kill -9 pid则是向进程号为pid的进程发送SIGKILL（该信号的编号为9），从本文上面的说明可知，SIGKILL既不能被应用程序捕获，也不能被阻塞或忽略，其动作是立即结束指定进程。通俗地说，应用程序根本无法“感知”SIGKILL信号，它在完全无准备的情况下，就被收到SIGKILL信号的操作系统给干掉了，显然，在这种“暴力”情况下，应用程序完全没有释放当前占用资源的机会。事实上，SIGKILL信号是直接发给init进程的，它收到该信号后，负责终止pid指定的进程。

---



## **进程上下文**

​    所谓的进程上下文，就是一个进程在执行的时候，CPU的所有寄存器中的值、进程的状态以及堆栈上的内容，当内核需要切换到另一个进程时，它需要保存当前进程的所有状态，即保存当前进程的进程上下文，以便再次执行该进程时，能够恢复切换时的状态，继续执行。

​    一个进程的上下文可以分为三个部分:**用户级上下文**、**寄存器上下文**以及**系统级上下文**。

* 用户级上下文: 正文、数据、用户堆栈以及共享存储区；

* 寄存器上下文: 通用寄存器、程序寄存器(IP)、处理器状态寄存器(EFLAGS)、栈指针(ESP)；

* 系统级上下文: 进程控制块task_struct、内存管理信息(mm_struct、vm_area_struct、pgd、pte)、内核栈。

​    当发生进程调度时，进行进程切换就是上下文切换(context switch)。

​    操作系统必须对上面提到的全部信息进行切换，新调度的进程才能运行。

而**系统调用进行的是模式切换(mode switch)**。模式切换与进程切换比较起来，容易很多，而且节省时间，因为**模式切换最主要的任务只是切换进程寄存器上下文的切换**。进程上下文主要是异常处理程序和内核线程。内核之所以进入进程上下文是因为进程自身的一些工作需要在内核中做。例如，系统调用是为当前进程服务的，异常通常是处理进程导致的错误状态等。所以在进程上下文中引用current是有意义的。

## 中断上下文具体包括：

（1）硬件传递过来的参数

因此上下文切换可以分为以下几类：

（1）进程之间的上下文切换：A进程切换到B进程

（2）进程和中断之间的上下文切换：进程A被中断打断

（3）中断之间的上下文切换：低级别中断被高级别中断打断

### **模式切换**

这是要说一种特殊的上下文切换：模式切换，即进程A从用户态因为系统调用进入内核态，这种切换之所以特殊，是因为它并没有经过完整的上下文切换，只是寄存器上下文进行了切换，所以模式切换的耗时相对完整进程上下文更低。

https://blog.csdn.net/zqixiao_09/article/details/50877756

## 进程切换分两步

1.切换**页目录以使用新的地址空间**

2.切换**内核栈和硬件上下文**。

对于linux来说，线程和进程的最大区别就在于地址空间。
对于线程切换，第1步是不需要做的，第2是进程和线程切换都要做的。所以明显是进程切换代价大

线程上下文切换和进程上下文切换一个最主要的区别是线程的切换虚拟内存空间依然是相同的，但是进程切换是不同的。这两种上下文切换的处理都是通过操作系统内核来完成的。内核的这种切换过程伴随的最显著的性能损耗是将寄存器中的内容切换出.

另外一个隐藏的损耗是上下文的切换会扰乱处理器的缓存机制。简单的说，一旦去切换上下文，处理器中所有已经缓存的内存地址一瞬间都作废了。还有一个显著的区别是当你改变虚拟内存空间的时候，处理的页表缓（processor’s Translation Lookaside Buffer (TLB)）或者相当的神马东西会被全部刷新，这将导致内存的访问在一段时间内相当的低效。但是在线程的切换中，不会出现这个问题。

## 进程上下文切换跟系统调用又有什么区别呢

首先，**进程是由内核来管理和调度的，进程的切换只能发生在内核态**。所以，进程的上下文不仅包括了虚拟内存、栈、全局变量等用户空间的资源，还包括了内核堆栈、寄存器等内核空间的状态。

因此，**进程的上下文切换就比系统调用时多了一步：在保存内核态资源（当前进程的内核状态和 CPU 寄存器）之前，需要先把该进程的用户态资源（虚拟内存、栈等）保存下来；而加载了下一进程的内核态后，还需要刷新进程的虚拟内存和用户栈**。

如下图所示，保存上下文和恢复上下文的过程并不是“免费”的，需要内核在 CPU 上运行才能完成。

 Linux 通过 TLB（Translation Lookaside Buffer）来管理虚拟内存到物理内存的映射关系。当虚拟内存更新后，TLB 也需要刷新，内存的访问也会随之变慢。特别是在多处理器系统上，缓存是被多个处理器共享的，刷新缓存不仅会影响当前处理器的进程，还会影响共享缓存的其他处理器的进程。



![img](/Users/ano/Desktop/notes/image/v2-440bb1699b2fa0f0340b38eabcbd7452_1440w.jpg)

## 进程上下文切换的场景

1. 为了保证所有进程可以得到公平调度，CPU 时间被划分为一段段的时间片，这些时间片再被轮流分配给各个进程。这样，当某个进程的时间片耗尽了，就会被系统挂起，切换到其它正在等待 CPU 的进程运行。
2. 进程在系统资源不足（比如内存不足）时，要等到资源满足后才可以运行，这个时候进程也会被挂起，并由系统调度其他进程运行。
3. 当进程通过睡眠函数 sleep 这样的方法将自己主动挂起时，自然也会重新调度。
4. 当有优先级更高的进程运行时，为了保证高优先级进程的运行，当前进程会被挂起，由高优先级进程来运行
5. 发生硬件中断时，CPU 上的进程会被中断挂起，转而执行内核中的中断服务程序。

## 线程上下文切换

线程与进程最大的区别在于：线程是调度的基本单位，而进程则是资源拥有的基本单位。说白了，所谓内核中的任务调度，实际上的调度对象是线程；而进程只是给线程提供了虚拟内存、全局变量等资源。

所以，对于线程和进程，我们可以这么理解：

- 当进程只有一个线程时，可以认为进程就等于线程。
- 当进程拥有多个线程时，这些线程会共享相同的虚拟内存和全局变量等资源。这些资源在上下文切换时是不需要修改的。
- 另外，线程也有自己的私有数据，比如栈和寄存器等，这些在上下文切换时也是需要保存的。。







## INODE

stat -x file 

![image-20200724230508284](/Users/ano/Desktop/notes/image/image-20200724230508284.png)

## fork的实现分为以下两步

\1. 复制进程资源
\2. 执行该进程

复制进程的资源包括以下几步
\1. 进程pcb
\2. 程序体，即代码段数据段等
\3. 用户栈
\4. 内核栈
\5. 虚拟内存池
\6. 页表

- 子进程用于独立且唯一的进程 ID；
- 子进程的父进程 ID 与父进程 ID 完全相同；
- 子进程不会继承父进程的内存锁；
- 子进程会重新设置进程资源利用率和 CPU 计时器；



fork后子进程和父进程共享的资源还包括：

* 打开的文件

* 实际用户ID、实际组ID、有效用户ID、有效组ID

* 添加组ID

* 进程组ID

* 会话期ID

* 控制终端

* 设置-用户-ID标志和设置-组-ID标志

* 当前工作目录

* 根目录

* 文件方式创建屏蔽字

* 信号屏蔽和排列

* 对任一打开文件描述符的在执行时关闭标志

* 环境

* 连接的共享存储段（共享内存）

* 资源限制

  父子进程之间的区别是：

* fork的返回值

* 进程ID

* 不同的父进程ID

* 子进程的tms_utime，tms_stime，tms-cutime以及tms_ustime设置为0

* 父进程设置的锁，子进程不继承

* 子进程的未决告警被清除

* 子进程的未决信号集设置为空集

  

## 进程的调度

1 先来先服务调度算法
先来先服务(FCFS)调度算法是一种最简单的调度算法，该算法既可用于作业调度，也可用于进程调度。当在作业调度中采用该算法时，每次调度都是从后备作业队列中选择一个或多个最先进入该队列的作业，将它们调入内存，为它们分配资源、创建进程，然后放入就绪队列。在进程调度中采用FCFS算法时，则每次调度是从就绪队列中选择一个最先进入该队列的进程，为之分配处理机，使之投入运行。该进程一直运行到完成或发生某事件而阻塞后才放弃处理机。

 

2、短作业(进程)优先调度算法
短作业(进程)优先调度算法，是指对短作业或短进程优先调度的算法。它们可以分别用于作业调度和进程调度。短作业优先(SJF)的调度算法是从后备队列中选择一个或若干个估计运行时间最短的作业，将它们调入内存运行。而短进程优先(SPF)调度算法则是从就绪队列中选出一个估计运行时间最短的进程，将处理机分配给它，使它立即执行并一直执行到完成，或发生某事件而被阻塞放弃处理机时再重新调度。

 

3、时间片轮转法
在早期的时间片轮转法中，系统将所有的就绪进程按先来先服务的原则排成一个队列，每次调度时，把CPU分配给队首进程，并令其执行一个时间片。时间片的大小从几ms到几百ms。当执行的时间片用完时，由一个计时器发出时钟中断请求，调度程序便据此信号来停止该进程的执行，并将它送往就绪队列的末尾；然后，再把处理机分配给就绪队列中新的队首进程，同时也让它执行一个时间片。这样就可以保证就绪队列中的所有进程在一给定的时间内均能获得一时间片的处理机执行时间。换言之，系统能在给定的时间内响应所有用户的请求。

 

4、多级反馈队列调度算法
前面介绍的各种用作进程调度的算法都有一定的局限性。如短进程优先的调度算法，仅照顾了短进程而忽略了长进程，而且如果并未指明进程的长度，则短进程优先和基于进程长度的抢占式调度算法都将无法使用。而多级反馈队列调度算法则不必事先知道各种进程所需的执行时间，而且还可以满足各种类型进程的需要，因而它是目前被公认的一种较好的进程调度算法。在采用多级反馈队列调度算法的系统中，调度算法的实施过程如下所述：

1）应设置多个就绪队列，并为各个队列赋予不同的优先级。第一个队列的优先级最高，第二个队列次之，其余各队列的优先权逐个降低。该算法赋予各个队列中进程执行时间片的大小也各不相同，在优先权愈高的队列中，为每个进程所规定的执行时间片就愈小。例如，第二个队列的时间片要比第一个队列的时间片长一倍，第i+1个队列的时间片要比第i个队列的时间片长一倍。

2）当一个新进程进入内存后，首先将它放入第一队列的末尾，按FCFS原则排队等待调度。当轮到该进程执行时，如它能在该时间片内完成，便可准备撤离系统；如果它在一个时间片结束时尚未完成，调度程序便将该进程转入第二队列的末尾，再同样地按FCFS原则等待调度执行；如果它在第二队列中运行一个时间片后仍未完成，再依次将它放入第三队列，……，如此下去，当一个长作业(进程)从第一队列依次降到第n队列后，在第n队列便采取按时间片轮转的方式运行。

3）仅当第一队列空闲时，调度程序才调度第二队列中的进程运行；仅当第1～(i-1)队列均空时，才会调度第i队列中的进程运行。如果处理机正在第i队列中为某进程服务时，又有新进程进入优先权较高的队列(第1～(i-1)中的任何一个队列)，则此时新进程将抢占正在运行进程的处理机，即第i队列中某个正在运行的进程的时间片用完后，由调度程序选择优先权较高的队列中的那一个进程，把处理机分配给它。

优先权调度算法
为了照顾紧迫型作业，使之在进入系统后便获得优先处理，引入了最高优先权优先(FPF)调度算法。此算法常被用于批处理系统中，作为作业调度算法，也作为多种操作系统中的进程调度算法，还可用于实时系统中。当把该算法用于作业调度时，系统将从后备队列中选择若干个优先权最高的作业装入内存。当用于进程调度时，该算法是把处理机分配给就绪队列中优先权最高的进程，这时，又可进一步把该算法分成如下两种。

1) 非抢占式优先权算法

在这种方式下，系统一旦把处理机分配给就绪队列中优先权最高的进程后，该进程便一直执行下去，直至完成；或因发生某事件使该进程放弃处理机时，系统方可再将处理机重新分配给另一优先权最高的进程。这种调度算法主要用于批处理系统中；也可用于某些对实时性要求不严的实时系统中。

2) 抢占式优先权调度算法

在这种方式下，系统同样是把处理机分配给优先权最高的进程，使之执行。但在其执行期间，只要又出现了另一个其优先权更高的进程，进程调度程序就立即停止当前进程(原优先权最高的进程)的执行，重新将处理机分配给新到的优先权最高的进程。因此，在采用这种调度算法时，是每当系统中出现一个新的就绪进程i时，就将其优先权Pi与正在执行的进程j的优先权Pj进行比较。如果Pi≤Pj，原进程Pj便继续执行；但如果是Pi>Pj，则立即停止Pj的执行，做进程切换，使i进程投入执行。显然，这种抢占式的优先权调度算法能更好地满足紧迫作业的要求，故而常用于要求比较严格的实时系统中，以及对性能要求较高的批处理和分时系统中。

Linux 从整体上区分实时进程和普通进程，因为实时进程和普通进程度调度是不同的，它们两者之间，实时进程应该先于普通进程而运行，然后，对于同一类型的不同进程，采用不同的标准来选择进程。对普通进程的调度策略是动态优先调度，对于实时进程采用了两种调度策略，FIFO(先来先服务调度)和RR（时间片轮转调度）。

UNIX系统是单纯的分时系统，所以没有设置作业调度。UNIX系统的进程调度采用的算法是，多级反馈队列调度法。其核心思想是先从最高休先级就绪队列中取出排在队列最前面的进程，当进程执行完一个时间片仍未完成则剥夺它的执行，将它放入到相应的队列中，取出下一个就绪进程投入运行，对于同一个队列中的各个进程，按照时间片轮转法调度。多级反馈队列调度算法即能使高优先级的作业得到响应又能使短作业（进程）迅速完成。但是它还是存在某些方面的不足，当不断有新进程到来时，则长进程可能饥饿。

## 完全公平调度算法CFS


原文链接：https://blog.csdn.net/fuzhongmin05/java/article/details/55802925

https://blog.csdn.net/XD_hebuters/article/details/79623130

## 进程的组成

 **进程是由程序控制块（PCB）、程序段、数据段组成。**
  操作系统是通过PCB来管理进程，因此PCB中应该包含操作系统对其进行管理所需的各种信息，如进程描述信息、进程控制和管理信息、资源分配清单和处理机相关信息。
  程序段：程序代码存放的位置。
  数据段：程序运行时使用、产生的运算数据。如全局变量、局部变量、宏定义的常量就存放在数据段内。

PCB 里面 task—Struct

标识相关：pid，ppid等等
文件相关：进程需要记录打开的文件信息，于是需要文件描述符表
内存相关：内存指针，指向进程的虚拟地址空间（用户空间）信息
优先级相关：进程相对于其他进程的调度优先级
上下文信息相关：CPU的所有寄存器中的值、进程的状态以及堆栈上的内容，当内核需要切换到另一个进程时，需要保存当前进程的所有状态，即保存当前进程的进程上下文，以便再次执行该进程时，能够恢复切换时的状态，继续执行。
状态相关：进程当前的状态，说明该进程处于什么状态
信号相关：进程的信号处理函数，以及记录当前进程是否还有待处理的信号
I/O相关：记录进程与各种I/O设备之间的交互



## float浮点数

![image-20200629130913247](/Users/ano/Desktop/notes/image/image-20200629130913247.png)

![image-20200629130931296](/Users/ano/Desktop/notes/image/image-20200629130931296.png)



![image-20200629130947046](/Users/ano/Desktop/notes/image/image-20200629130947046.png)

1. double 编码

   ![image-20200629131016685](/Users/ano/Desktop/notes/image/image-20200629131016685.png)

   ![image-20200629131050228](/Users/ano/Desktop/notes/image/image-20200629131050228.png)

2. ## 进程与线程

   **进程是程序在计算机上的一次执行活动。**当你运行一个程序，你就启动了一个进程。显然，程序只是一组指令的有序集合，它本身没有任何运行的含义，只是一个静态实体。而进程则不同，它是程序在某个数据集上的执行，是一个动态实体。它因创建而产生，因调度而运行，因等待资源或事件而被处于等待状态，因完成任务而被撤消，反映了一个程序在一定的数据集上运行的全部动态过程。

   **进程是操作系统分配资源的单位。**在Windows下，进程又被细化为线程，也就是一个进程下有多个能独立运行的更小的单位。

   **线程(Thread)是进程的一个实体，是CPU调度和分派的基本单位。**线程不能够独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制

   

   

3. ## PCB一般包括：

   1.程序ID（PID、进程句柄）：它是唯一的，一个进程都必须对应一个PID。PID一般是整形数字

   2.特征信息：一般分系统进程、用户进程、或者内核进程等

   3.进程状态：运行、就绪、阻塞，表示进程现的运行情况

   4.优先级：表示获得CPU控制权的优先级大小

   5.通信信息：进程之间的通信关系的反映，由于操作系统会提供通信信道

   6.现场保护区：保护阻塞的进程用

   7.资源需求、分配控制信息

   8.进程实体信息，指明程序路径和名称，进程数据在物理内存还是在交换分区（分页）中

   9.其他信息：工作单位，工作区，文件信息等 [1]

4. ## 逻辑地址、相对地址与物理地址

    

 逻辑地址：与当前数据在内存中的物理分配地址无关的访问地址，在执行对内存的访问之前必须转化为物理地址。

相对地址：特殊的逻辑地址，相对于某些已知点的存储单元。

物理地址：数据在主存中的实际位置

5. ## 分段分页

   **分页**逻辑地址到物理地址的映射过程，分页系统中，允许将进程的每一页离散地存储在内存的任一物理块中，为了能在内存中找到每个页面对应的物理块，系统为每个进程建立一张页表，用于记录进程逻辑页面与内存物理页面之间的对应关系。页表的作用是实现从页号到物理块号的地址映射，地址空间有多少页，该页表里就登记多少行，且按逻辑页的顺序排列，形如：

   

   分段存储管理

   1.基本思想

   页面是主存物理空间中划分出来的等长的固定区域。分页方式的优点是页长固定，因而便于构造页表、易于管理，且不存在外碎片。但分页方式的缺点是页长与程序的逻辑大小不相关。例如，某个时刻一个子程序可能有一部分在主存中，另一部分则在辅存中。这不利于编程时的独立性，并给换入换出处理、存储保护和存储共享等操作造成麻烦。

   另一种划分可寻址的存储空间的方法称为分段。段是按照程序的自然分界划分的长度可以动态改变的区域。通常，程序员把子程序、操作数和常数等不同类型的数据划分到不同的段中，并且每个程序可以有多个相同类型的段。

    段表本身也是一个段，可以存在辅存中，但一般是驻留在主存中。将用户程序地址空间分成若干个大小不等的段，每段可以定义一组相对完整的逻辑信息。存储分配时，以段为单位，段与段在内存中可以不相邻接，也实现了离散分配。

   2. 分段地址结构

   作业的地址空间被划分为若干个段，每个段定义了一组逻辑信息。例程序段、数据段等。每个段都从0开始编址，并采用一段连续的地址空间。段的长度由相应的逻辑信息组的长度决定，因而各段长度不等。整个作业的地址空间是二维的。

   在段式虚拟存储系统中，虚拟地址由段号和段内地址组成，虚拟地址到实存地址的变换通过段表来实现。每个程序设置一个段表，段表的每一个表项对应一个段，每个表项至少包括三个字段：有效位（指明该段是否已经调入主存）、段起址(该段在实存中的首地址)和段长（记录该段的实际长度）。

   ***分段存储方式的优缺点***

   分页对程序员而言是不可见的，而分段通常对程序员而言是可见的，因而分段为组织程序和数据提供了方便。与页式虚拟存储器相比，段式虚拟存储器有许多优点：

   (1)    段的逻辑独立性使其易于编译、管理、修改和保护，也便于多道程序共享。

   (2)    段长可以根据需要动态改变，允许自由调度，以便有效利用主存空间。

   (3)    方便编程，分段共享，分段保护，动态链接，动态增长

    因为段的长度不固定，段式虚拟存储器也有一些缺点：

   (1)    主存空间分配比较麻烦。

   (2)    容易在段间留下许多碎片，造成存储空间利用率降低。

   (3)    由于段长不一定是2的整数次幂，因而不能简单地像分页方式那样用虚拟地址和实存地址的最低若干二进制位作为段内地址，并与段号进行直接拼接，必须用加法操作通过段起址与段内地址的求和运算得到物理地址。因此，段式存储管理比页式存储管理方式需要更多的硬件支持。






   段页式结合 对于每一个虚拟地址，处理器使用段号检索进程段表来寻找该段的页表；页号用来检索页表并查找到帧号。

   段页式存储管理的基本思想

   段页式存储组织是分段式和分页式结合的存储组织方法，这样可充分利用分段管理和分页管理的优点。

   　  (1) 用分段方法来分配和管理虚拟存储器。程序的地址空间按逻辑单位分成基本独立的段，而每一段有自己的段名，再把每段分成固定大小的若干页。

   (2) 用分页方法来分配和管理实存。即把整个主存分成与上述页大小相等的存储块，可装入作业的任何一页。程序对内存的调入或调出是按页进行的。但它又可按段实现共享和保护。

   

   

   段页式存储管理的优缺点

    优点

   (1) 它提供了大量的虚拟存储空间。

   (2) 能有效地利用主存，为组织多道程序运行提供了方便。

   缺点：

   (1) 增加了硬件成本、系统的复杂性和管理上的开消。

   (2) 存在着系统发生抖动的危险。

   (3) 存在着内碎片。

   (4) 还有各种表格要占用主存空间。

   　段页式存储管理技术对当前的大、中型计算机系统来说，算是最通用、最灵活的一种方案。
   ————————————————

   原文链接：https://blog.csdn.net/zephyr_be_brave/java/article/details/8944967

   

5. ### 进程间通信IPC (InterProcess Communication)

   **1.** **管道/** 匿名管道(pipe) 只能用于父子进程或者兄弟进程之间(具有亲缘关系的进程

6. ### 虚拟内存

   虚拟内存的目的是为了让物理内存扩充成更大的逻辑内存，从而让程序获得更多的可用内存，为了更好的管理内存，操作系统将内存抽象成地址空间。每个程序拥有自己的地址空间，这个地址空间被分割成多个块，每一块称为一页。这些页被映射到物理内存，但不需要映射到连续的物理内存，也不需要所有页都必须在物理内存中。当程序引用到不在物理内存中的页时，由硬件执行必要的映射，将缺失的部分装入物理内存并重新执行失败的指令。

   内存管理单元（MMU）管理着地址空间和物理内存的转换，其中的页表（Page table）存储着页（程序地址空间）和页框（物理内存空间）的映射表。

   一个虚拟地址分成两个部分，一部分存储页面号，一部分存储偏移量。

7. ### 死锁必要条件和处理方法

   - 互斥：每个资源要么已经分配给了一个进程，要么就是可用的。
   - 占有和等待：已经得到了某个资源的进程可以再请求新的资源。
   - 不可抢占：已经分配给一个进程的资源不能强制性地被抢占，它只能被占有它的进程显式地释放。
   - 环路等待：有两个或者两个以上的进程组成一条环路，该环路中的每个进程都在等待下一个进程所占有的资源。

8. ### 硬盘有什么组成

   （磁头、磁道、扇区、柱面）

   

9. ## linux启动方式


   原文链接：https://blog.csdn.net/a624731186/java/article/details/22691049

   1. BIOS自检,BIOS的第一个步骤是加电自检,BIOS的第二个步骤是检测本地设备。,侦测电脑周边配套设备是否工作正  常,如cpu的类型,速度,缓存等;主板类型,内存的速度,容量,硬盘的大小,类型和工作模式,风扇速度等,主要是为了检查这些设备在开机的时候是否能正常的工作.
   2. 载入启动程序
      主板的BIOS会读取硬盘的主引导记录(MBR),MBR中存放的是一段很小的程序,他的功能是从硬盘读取操作系统核心文件并运行,因为这个小程序太小了,因此通常这个小程序不具备直接引导系统内核的能力,他先去引导另一个稍微大一点的小程序,再由这个大一点的小程序去引导系统内核.
   3. 加载内核
      LINUX内核一般是压缩保存的，因此，它首先要进行自身的解压缩。内核映象前面的一些代码完成解压缩。解压后将其放入高端内存中，如果有初始RAM磁盘映像，就会将它移动到内存中，并标明以后使用，然后内核映象前面的代码调用内核，并开始启动内核引导的过程 
   4. 启动init服务
           这里的Init程序，一般放在/sbin下,(到这里会出现很多不同的启动方式，主要有：SystemV,BSD,upstart和systemd).
            这里主要说SystemV,init进程是所有进程的起点，也是Linux内核启动后的第一个动作，所以这个程序的PID是永远是1，init进程是所有进程的发起者和控制者
   5. 执行run level对应目录中的脚本，例如：等级为5，则执行/etc/rc.d/rc5.d下面的脚本
      执行时按脚本的文件名 串行执行，这样就造成开机比较慢。目前systemd是以并行执行（号称最快2秒开机）

10. ## 用户态到内核态切换可以通过三种方式：

    1. 系统调用，这个上面已经讲解过了。其实系统调用本身就是中断，但是软件中断，跟硬中断不同。
    2. 异常：如果当前进程运行在用户态，如果这个时候发生了异常事件，就会触发切换。例如：缺页异常。
    3. 外设中断：当外设完成用户的请求时，会向CPU发送中断信号。

11. ## c/c++ struct的大小

    3条原则:(在没有#pragma pack宏的情况下)

    １:数据成员对齐规则：结构(struct)(或联合(union))的数据成员，第一个数据成员放在offset为0的地方，以后每个数据成员存储的起始位置要从该成员大小的整数倍开始(比如int在３２位机为４字节,则要从４的整数倍地址开始存储。

    ２:结构体作为成员:如果一个结构里有某些结构体成员,则结构体成员要从其内部最大元素大小的整数倍地址开始存储.(struct a里存有struct b,b里有char,int ,double等元素,那b应该从8的整数倍开始存储.)

    ３:收尾工作:结构体的总大小,也就是sizeof的结果,.必须是其内部最大成员的整数倍.不足的要补齐

12. ## Linux下 文件描述符（fd）与 文件指针（FILE*）

    ![image-20200719165144942](/Users/ano/Desktop/notes/image/image-20200719165144942.png)

13. ### 磁盘调度算法  （SCAN：电梯调度算法）

    https://blog.csdn.net/Jaster_wisdom/article/details/52345674



![image-20200719192434420](/Users/ano/Desktop/notes/image/image-20200719192434420.png)

# Mysql 

## 第一二三范式 ：

**1NF的定义为：符合1NF的关系中的每个属性都不可再分**

**2NF在1NF的基础之上，消除了非主属性对于码的部分函数依赖**。

>  成绩表 （学号，系，课，分数）， 码 = （学号+课） 
>
>  系只依赖学号

 **3NF在2NF的基础之上，消除了非主属性对于码的传递函数依赖**

> （学号，系，系主任）， 码 = 学号
>
> 学号-》系-》系主任

**BCNF再3NF 的基础上消除主属性对于码的部分与传递函数依赖。**

> （仓库，管理员，物品，数额）
>
> 主属性：仓库名、管理员、物品名
> 非主属性：数量
>
> 

https://www.zhihu.com/question/24696366



## 索引

### 索引是啥？优缺点

```
索引是帮助MySQL高效获取数据的数据结构（有序）。



提高数据检索的效率，降低数据库的IO成本，同时却也降低更新表的速度，如对表进行INSERT、UPDATE、DELETE。因为更新表时，MySQL 不仅要保存数据，还要保存一下索引文件每次更新添加了索引列的字段，都会调整因为更新所带来的键值变化后的索引信息。
```



### 索引类型，结构

```
https://www.jianshu.com/p/f74feb7dbdc1

- 1. BTree 索引 ： 最常见的索引类型，大部分索引都支持 BTree索引。

  2. Hash 索引：Memory引擎支持 ， 使用场景简单 。

  3. R-tree 索引（空间索引）：空间索引是MyISAM引擎的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少。

  4. Full-text （全文索引） ：全文索引也是MyISAM的一个特殊索引类型，主要用于全文索引，InnoDB从Mysql5.6版本开始支持全文索引。

  

- 1） 单值索引 ：即一个索引只包含单个列，一个表可以有多个单列索引。
  2） 唯一索引 ：索引列的值必须唯一，但允许有空值。
  3） 复合索引 ：即一个索引包含多个列，例如用身份证号和考生号两列作为复合索引
  4） 主键索引,也就是在唯一索引的基础上相应的列必须为主键
```



### 什么时候建议/不建议+索引

#### 索引建立原则

```
（1）. 尽量减少like，但不是绝对不可用，”xxxx%” 是可以用到索引的
（2）. 表的主键、外键必须有索引
（3）. 谁的区分度更高（同值的最少），谁建索引，区分度的公式是count(distinct（字段）)/count(*)
（4）. 单表数据太少，不适合建索引
（5）. where，order by ,group by 等过滤时，后面的字段最好加上索引
（6）. 如果既有单字段索引，又有这几个字段上的联合索引，一般可以删除联合索引；
（7）. 联合索引的建立需要进行仔细分析；尽量考虑用单字段索引代替：
（8）. 联合索引: mysql 从左到右的使用索引中的字段，一个查询可以只使用索引中的一部份，但只能是最左侧部分。例如索引是key index(a,b,c). 可以支持 a|a,b|a,b,c 3种组合进行查找，但不支持 b,c 进行查找.当最左侧字段是常量引用时，索引就十分有效。
（9）. 前缀索引: 有时候需要索引很长的字符列，这会让索引变得大且慢。通常可以索引开始的部分字符，这样可以大大节约索引空间，从而提高索引效率。其缺点是不能用于ORDER BY和GROUP BY操作，也不能用于覆盖索引 Covering index（即当索引本身包含查询所需全部数据时，不再访问数据文件本身）
10）. NULL会导致索引形同虚设
```

#### 索引什么时候失效

1. ```
   1. where条件中有or，即使其中有条件带索引也不会使用；如果要想使用or又想让索引生效，只能将or条件中的每个列都加上索引；
   2. 对于多列索引，不是使用的第一部分，则不会使用索引（即不符合最左前缀原则）；
   3. like查询是以%开头；
   4. 如果列类型是字符串，那一定要在条件中将数据使用引号引用起来，否则不使用索引；
   5. 如果mysql估计使用全表扫描要比使用索引快，则不使用索引
   ```



### 索引下推是什么  (Index Condition Pushdown) ICP

```
在Mysql5.6以后引入了索引下推优化，可以在索引遍历过程中对索引所包含的字段先做判断，直接过滤不满足的条件，旨在减少回表次数和减少MySQL server层和引擎层的交互次数；
```



## 什么是游标/光标

```
游标是用来存储查询结果集的数据类型 , 在存储过程和函数中可以使用光标对结果集进行循环的处理
```

## 触发器是什么？

```
触发器是与表有关的数据库对象，指在 insert/update/delete 之前或之后，触发并执行触发器中定义的SQL语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性 , 日志记录 , 数据校验等操作 。
```



## 存储引擎有哪些，特点是啥？InnoDB主要特性？

https://blog.csdn.net/zgrgfr/article/details/74455547

## innode 和MyISAM 区别

* InnoDB 中不保存表的具体行数，也就是说，执行select count(*) from table时，InnoDB要扫描一遍整个表来计算有多少行，但是MyISAM只要简单的* 读出保存好的行数即可。注意的是，当count(*)语句包含 where条件时，两种表的操作是一样的。
* ***\*两种类型最主要的差别就是Innodb 支持事务处理与外键和行级锁。\****而MyISAM不支持.所以MyISAM往往就容易被人认为只适合在小项目中使用。
* InnoDB 把数据和索引存放在表空间里面， MyISAM 表被存放在三个文件 。frm 文件存放表格定义。 数据文件是MYD (MYData) 。 索引文件是MYI (MYIndex)引伸
* **InnoDB支持事务，MyISAM不支持，对于InnoDB每一条SQL语言都默认封装成事务，自动提交，这样会影响速度，所以最好把多条SQL语言放在begin和commit之间，组成一个事务；**
* **InnoDB表必须有主键（用户没有指定的话会自己找或生产一个主键），而Myisam可以没有**

## mysql 日志

   ```
   https://www.jianshu.com/p/db19a1d384bc
   
   Mysql 有4种类型的日志：Error Log、Genaral Query Log、 Binary Log 和 Slow Query Log
   
   Error Log 记录Mysql运行过程中的Error、Warning、Note等信息，系统出错或者某条记录出问题可以查看Error日志。
   
   General Query Log 记录mysql的日常日志，包括查询、修改、更新等的每条sql。
   
   Binary Log 二进制日志，包含一些事件，这些事件描述了数据库的改动，如建表、数据改动等，主要用于备份恢复、回滚操作等，需要配置一个server-id
   ```

   

## InnoDB 重做日志(redo log) 回滚日志(undo log）是什么

- ```
  InnoDB 有两块非常重要的日志，一个是undo log，另外一个是redo log，前者用来保证事务的原子性以及InnoDB的MVCC，后者用来保证事务的持久性。
  
  - RedoLog
  
  事务将数据的变更写到内存页中同时记录到redolog里面，然后提交就算这个事务已经结束，期间只有一次磁盘IO，效率远比比事务将数据直接写到磁盘里面要高的多，并且redolog写到磁盘可以看作是顺序IO，所以redolog能提升性能的一个方面就在于将对磁盘的随机IO写转换成顺序IO；且事务因为宕机可能会丢失，redolog还有一个作用就是保证事务的持久性；
  
  
  为什么需要Redo log？
  Innodb buffer pool内部储存的是数据页，当修改innodb表上某行数据时，如果该行不在内存中则需要将该行所在的数据页从磁盘读入到内存去，然后在内存中更新该行，现在内存中的数据页与磁盘中就不一致了，把这种内存数据页与磁盘数据页不一致的数据页称为脏页（dirty page），DB需要把脏页数据写入磁盘，但是如果每一次更新数据就会带来一次磁盘操作的话那么机器肯定撑不住；所以说mysql会把标记为脏页的数据页储存在一个flush list里面，用一个专门的后台线程定时刷脏；那么如果在mysql还没有刷脏的时候数据库挂了怎么办呢，所以就需要redo log；
  
  - UndoLog
  
  Undo log的存在保证了事务的原子性，MVCC就是依赖它来实现，当对任何行做了修改的时候都会在undo log里面记录，大量的undo log构成行的历史版本记录，在需要的时候可以回退(rollback)到任何版本；
  
   
  二、不同的DML操作，UNDO BLOCK中保存的前映像内容
  INSERT操作，UNDO中只需要保存插入记录的rowid，如果需要回退，通过保存的rowid进行删除即可（后面有案例）
  UPDATE操作，UNDO中只需要记录被更新字段的旧值，如果需要回退，只需要通过旧值覆盖更新后的值即可。
  DELETE操作，UNDO中必须记录整行的数据，如果需要回退，只需要将这整行的数据重新插入至表中即可。
  
  
     
  https://www.jianshu.com/p/c5a3c3a0508f   
  ```

  ## redo log

  https://juejin.im/entry/6844903681091977229

  redo log是innodb层产生的，只记录该存储引擎中表的修改，redo log包括两部分：一是内存中的日志缓冲(redo log buffer)，该部分日志是易失性的；二是磁盘上的重做日志文件(redo log file)，该部分日志是持久的。

  

  

  在概念上，innodb通过***force log at commit\***机制实现事务的持久性，即在事务提交的时候，必须先将该事务的所有事务日志写入到磁盘上的redo log file和undo log file中进行持久化。

  

  ![image-20200822001337259](/Users/ano/Desktop/notes/image/image-20200822001337259.png)

  

  ### 写入机制

  `redo log`写入前是先写入到`redo log buffer`，buffer中的内容是不会持久化到磁盘中，如果写buffer的过程中异常退出了会丢失这部分日志，但是这部分的事务是没有提交的所有丢失的影响也不大；

  

  `innodb_flush_log_at_trx_commit`参数控制着redo log的写入策略：

  - 0： 表示每次事务提交时都只是把 redo log 留在 redo log buffer中，然后每秒后台线程使用fsync写入磁盘；
  - 1：表示每次事务提交时都使用fsync将 redo log 直接持久化到磁盘；
  - 2：表示每次事务提交时都只是把 redo log 写到 page cache中，然后每秒后台线程使用fsync写入磁盘；

  Innodb中有一个后台线程每隔1s会把redo log buffer中的日志write到文件系统的page cache中，然后使用fsync持久化到磁盘里面；

  

  ![image-20200822001604290](/Users/ano/Desktop/notes/image/image-20200822001604290.png)

  在主从复制结构中，要保证事务的持久性和一致性，需要对日志相关变量设置为如下：

  - **如果启用了二进制日志，则设置sync_binlog=1，即每提交一次事务同步写到磁盘中。**
  - **总是设置innodb_flush_log_at_trx_commit=1，即每提交一次事务都写到磁盘中。**

  ==**如果MySQL宕机，而Buffer Pool的数据没有完全刷新到磁盘，就会导致数据丢失，无法保证持久性。** 因此引入Redo Log解决这个问题。==

  - 当数据修改时，首先写入Redo Log，再更新到Buffer Pool，保证数据不会因为宕机而丢失，保证持久性。
  - 当事务提交时会调用fsync将redo log刷至磁盘持久化。MySQL宕机时，通过读取Redo Log中的数据，对数据库进行恢复。

  ==Redo Log也是记录在磁盘中，为什么会比直接将Buffer Pool写入磁盘更快？==

  - Buffer Pool刷入脏页至磁盘是随机IO，每次修改的数据位置随机，而Redo Log永远在页中追加，属于顺序IO。
  - Buffer Pool刷入磁盘是以数据页为单位，每次都需要整页写入。而Redo Log只需要写入真正**物理修改**的部分，IO数据量大大减少。

## undo log

undo log有两个作用：提供回滚和多版本并发控制(MVCC)。

Undo Log中基于回滚指针(DB_ROLL_PT)维护数据行的所有历史版本。

 

### Insert/update

InnoDB中，undo log分为：

- Insert Undo Log
- Update Undo Log

1. Insert Undo Log

Insert Undo Log是INSERT操作产生的undo log。

INSERT操作的记录由于是该数据的第一个记录，对其他事务不可见，该Undo Log可以在事务提交后直接删除。

![image-20200822004014716](/Users/ano/Desktop/notes/image/image-20200822004014716.png)



- type_cmpl：undo的类型
- undo_no：事务的ID
- table_id：undo log对应的表对象 接着的部分记录了所有主键的列和值。

Update Undo Log

Update Undo Log记录对DELETE和UPDATE操作产生的Undo Log。

Update Undo Log会提供MVCC机制，因此不能在事务提交时就删除，而是放入undo log链表，等待purge线程进行最后的删除。

![image-20200822004101341](/Users/ano/Desktop/notes/image/image-20200822004101341.png)

## binlog记录格式 ,主从复制三种方式

```
MySQL 主从复制有三种方式：基于SQL语句的复制（statement-based  replication，SBR），基于行的复制（row-based replication，RBR)，混合模式复制（mixed-based  replication,MBR)。对应的binlog文件的格式也有三种：STATEMENT,ROW,MIXED。
```



## 两阶段提交 https://zhuanlan.zhihu.com/p/98778890

最后，那么当我执行一条 update 语句时，redo log 和 binlog 是在什么时候被写入的呢？这就有了我们常说的「两阶段提交」：

- 写入：redo log（prepare）
- 写入：binlog
- 写入：redo log（commit）

为什么 redo log 要分两个阶段： prepare 和 commit ？redo log 就不能一次写入吗？

我们分两种情况讨论：

- 先写 redo log，再写 binlog
- 先写 binlog，再写 redo log

1、先写 redo log，再写 binlog

这样会出现 redo log 写入到磁盘了，但是 binlog 还没写入磁盘，于是当发生 crash recovery 时，恢复后，主库会应用 redo log，恢复数据，但是由于没有 binlog，从库就不会同步这些数据，主库比从库“新”，造成主从不一致

2、先写 binlog，再写 redo log

跟上一种情况类似，很容易知道，这样会反过来，造成从库比主库“新”，也会造成主从不一致

而两阶段提交，就解决这个问题，crash recovery 时：

- 如果 redo log 已经 commit，那毫不犹豫的，把事务提交

- 如果 redo log 处于 prepare，则去判断事务对应的 binlog 是不是完整的

- - 是，则把事务提交
  - 否，则事务回滚

两阶段提交，其实是为了保证 redo log 和 binlog 的逻辑一致性。



## DML 

DML语句的处理过程描述
update undotest set object_type='VIEW' where object_type='PROCEDURE';

* 检查shared pool中是否存在相同的语句，如果存在，重用执行计划，执行扫描运算，如果不存在，执行硬解析生成执行计划
  * 根据执行计划中的扫描运算，检查undotest表中的相关数据块是否存在buffer cache中，如果不存在则读取到内存中
  * 检查数据块中符合object_type='PROCEDURE'条件的记录，如果没有符合条件的行记录，则结束语句，如果存在则进入下一步
* 以当前模式（current）获取符合object_type='PROCEDURE'条件的数据块，准备进行更新
  * 在回滚表空间的相应回滚段头的事务表上分配事务槽，这个动作需要记录redo日志
* 从回滚段数据块上创建object_type='PROCEDURE'的前映像数据，这个动作也要记录redo日志
  * 修改object_type='VIEW' ，这是DML操作的数据变更，而需要记录redo日志
* 用户提交时，在redo日志中记录提交信息，将回滚段头上的事务表和回滚段数据块标记为非活动，清除修改数据块上的事务信息（也可能延迟清除）。同时必须确保整个事务的redo日志写到磁盘上的日志文件
  * 注意：如果最后用户回滚了事务，oracle从回滚段中将前映像数据提取出来，覆盖被更新的数据块。这个回滚动作本身也需要产生redo日志，因此，我们要知道回滚的代价非常昂贵。

## MVCC

### 隐藏的列

MVCC 在每行记录后面都保存着两个隐藏的列，用来存储两个版本号：

- 创建版本号：创建一行数据时，将当前系统版本号作为创建版本号赋值。
- 删除版本号：删除一行数据时，将当前系统版本号作为删除版本号赋值。如果该快照的删除版本号大于当前事务版本号表示该快照有效，否则表示该快照已经被删除了。

#### REPEATABLE READ（可重复读）隔离级别下MVCC如何工作：

当开始新一个事务时，该事务的版本号肯定会大于当前所有数据行快照的创建版本号，理解这一点很关键。

### **1. SELECT**

InnoDB会根据以下条件检查每一行记录：

\1. InnoDB只查找版本早于当前事务版本的数据行，这样可以**确保事务读取的行要么是在开始事务之前已经存在要么是事务自身插入或者修改过的**，在事务开始之后才插入的行，事务不会看到。

\2. 行的删除版本号要么未定义，要么大于当前事务版本号，这样可以**确保事务读取到的行在事务开始之前未被删除**，在事务开始之前就已经过期的数据行，该事务也不会看到。

只有符合上述两个条件的才会被查询出来

### **2. INSERT**

将当前系统版本号作为数据行快照的创建版本号。

### **3. DELETE**

将当前系统版本号作为数据行快照的删除版本号。

### 4. UPDATE

将当前系统版本号作为更新前的数据行快照的删除版本号，并将当前系统版本号作为更新后的数据行快照的创建版本号。**可以理解为先执行 DELETE 后执行 INSERT**。

保存这两个版本号，使大多数操作都不用加锁。使数据操作简单，性能很好，并且能保证只会读取到复合要求的行。不足之处是每行记录都需要额外的存储空间，需要做更多的行检查工作和一些额外的维护工作。



**MVCC只在COMMITTED READ（读提交）和REPEATABLE READ（可重复读）两种隔离级别下工作。**

可以认为MVCC是行级锁一个变种，但是他很多情况下避免了加锁操作，开销更低。虽然不同数据库的实现机制有所不同，但大都实现了非阻塞的读操作（读不用加锁，且能避免出现不可重复读和幻读），写操作也只锁定必要的行（写必须加锁，否则不同事务并发写会导致数据不一致）。

- **“快照”不是全量拷贝，而是利用了数据多版本的特性，也就是MVCC**
- **MVCC的核心在于每个事务自己维护的一个事务ID数组**
- **可以用“等价原则”来判断数据版本的可见性**
- **可重复读的关键，来自于“快照”**
- **“快照”并不是在begin后就生成，而是在第一条“快照读”语句后才生成**

https://www.jianshu.com/p/0d1d14ea84e6

### MVCC  Mysql到底是怎么实现MVCC的？

https://blog.csdn.net/chen77716/article/details/6742128

Read Committed隔离级别：每次select都生成一个快照读。

Read Repeatable隔离级别：开启事务后第一个select语句才是快照读的地方，而不是一开启事务就快照读。



## 间隙锁(Gap Lock) 是啥

```
间隙锁是为了解决在RR(可重复读)隔离级别下出现的幻读问题，虽然RR可以保证当前事务的不受其他事务的`update、delete`的影响，但是会因为`insert`而出现幻读；只有RR隔离级别下才会有Gap锁；

间隙锁和行锁合称Next-key lock(临键锁)

Gap Lock和Next-key lock只是在RR隔离级别下存在的，并遵守下面几个规则
    加锁的基本单位是Next-key lock；
    查找过程中访问到的对象才会加锁；
    索引上的等值查询，给唯一索引加锁的时候，Next-key lock退化成行锁；
    索引上的等值查询，向右遍历时最后一个值不满足等值条件的时候，Next-key lock退化成间隙锁；

```







## 事务的隔离级别

```
**读未提交(Read uncommitted)**
 即一个事务做出的改变还未提交就能被其他事务所看到

**读提交(Read committed)**
 即一个事务做出的改变需要提交后才能被其他事务看到

**可重复读(Repeatable read)**
 即一个事务执行的过程中看到的数据总是和这个事务启动时看到数据是一致的；

**串行化(Serializable)**
 对同一行数据，写会加写锁，读会加读锁，当读写锁出现冲突时后访问的事务会等待占用锁事务执行完成才会继续完成；
```

MVCC只会在`读提交`和`可重复读`这两个隔离级别下才会存在，读未提交是直接返回该行最新的数据，而串行化是直接采用加锁的方式避免并行访问。



## MySQL的语句执行顺序

开始->FROM子句->WHERE子句->GROUP BY子句->HAVING子句->ORDER BY子句->SELECT子句->LIMIT子句->最终结果 



(1)from 

(2) on 

(3) join 

(4) where 

(5)group by(开始使用select中的别名，后面的语句中都可以使用)
(6) avg,sum.... 
(7)having 
(8) select 
(9) distinct 

(10) order by
(11) limit 
————————————————

1. **FORM**: 首先对from子句中的前两个表执行一个笛卡尔乘积，此时生成虚拟表 vt1（选择相对小的表做基础表） 

2. **ON**: 对虚表VT1进行ON筛选，只有那些符合<join-condition>的行才会被记录在虚表VT2中。

3. **JOIN**： 如果指定了OUTER JOIN（比如left join、 right  join），那么保留表中未匹配的行就会作为外部行添加到虚拟表VT2中，产生虚拟表VT3, rug  from子句中包含两个以上的表的话，那么就会对上一个join连接产生的结果VT3和下一个表重复执行步骤1~3这三个步骤，一直到处理完所有的表为止。

4. **WHERE**： 对虚拟表VT3进行WHERE条件过滤。只有符合<where-condition>的记录才会被插入到虚拟表VT4中。

5. **GROUP BY**: 根据group by子句中的列，对VT4中的记录进行分组操作，产生VT5.

6. **CUBE | ROLLUP**: 对表VT5进行cube或者rollup操作，产生表VT6.

7. **HAVING**： 对虚拟表VT6应用having过滤，只有符合<having-condition>的记录才会被 插入到虚拟表VT7中。

8. **SELECT**： 执行select操作，选择指定的列，插入到虚拟表VT8中。

9. **DISTINCT**： 对VT8中的记录进行去重。产生虚拟表VT9.

10. **ORDER BY**: 将虚拟表VT9中的记录按照<order_by_list>进行排序操作，产生虚拟表VT10.

11. **LIMIT**：取出指定行的记录，产生虚拟表VT11, 并将结果返回。

     



## 主从复制模式，同步异步半同步

[https://zhuanlan.zhihu.com/p/50597960

## 方案

### mysql 分页

https://www.jianshu.com/p/59a28a0a88aa

###### 一、基于偏移的分页

> 例如： `http://XXXXXXXlist?page=1&count=20`
>
> **缺点：** 1、数据重复 2、数据缺失 3、效率低
>
> 使用limit在数据量小的时候并不会有效率问题，但是当数据偏移量很大时性能会开始急剧下降，查询性能比对会在接下来提到。
>
> 综上所述，流式分页不需要也不适合使用传统分页，而是使用一种更合适的方法：游标分页。
>
> 所以这种分页方法 适用于 数据部经常变化的情况。或者拿到数据后做去重处理。

###### 二、基于游标分页方案

>基于游标的分页是最有效的分页方法，应尽量使用这种方法。游标是指标记数据列表中特定项目的一个随机字符串。该项目未被删除时，游标将始终指向列表的同一部分，项目被删除时游标即失效。因此，您的应用不应存储任何旧的游标，也不能假定它们仍然有效。
>
>游标分页的连线支持以下参数：
>
>- **before**：这是指向已返回的数据页面开头的游标。
>- **after**：这是指向已返回的数据页面末尾的游标。
>- **limit**：这是每个页面返回的单独对象的数量。请注意：这是上限。如果数据列表中没有足够的剩余对象，那么返回的数量将小于这个数。为提升性能，将为某些连线的 limit 值设置上限。在调用案例中，API 将返回正确的分页链接。
>- **next**：将返回下一页数据的图谱 API 端点。如果未包含，则显示的是最后一页数据。根据分页的可见性和隐私，页面可能为空，但包含“next”分页链接。“next”链接不再出现时，应停止分页。
>- **previous**：将返回上一页数据的图谱 API 端点。如果未包含，则显示的是第一页数据。
>
>

##### 三、基于时间的分页

>  时间分页使用指向数据列表中的特定时间的 Unix 时间戳在结果数据中导航。
>
>  基于时间的连线支持以下参数：

- until：指向基于时间的数据范围末尾的 Unix 时间戳或 `strtotime`
  数据值。
- since：指向基于时间的数据范围开头的 Unix 时间戳或 `strtotime`
  数据值。
- limit：这是每个页面返回的单独对象的数量。限值 0 将不返回结果。为提升性能，将为某些连线的 `limit`值设置上限。在调用案例中，API 将返回正确的分页链接。
- next：将返回下一页数据的图谱 API 端点。
- previous：将返回上一页数据的图谱 API 端点。

![image-20200706190650629](/Users/ano/Desktop/notes/image/image-20200706190650629.png)



https://juejin.im/entry/5b5eb7f2e51d4519700f7d3c

## 分页

```sql
SELECT 查询列表 FROM 表名 WHERE 筛选条件
GROUP BY 分组条件 HAVING 分组后筛选
ORDER BY 排序列表
LIMIT 起始索引，显示的条目数
```

要显示的是第p页，每页显示s条数据， 则limit后参数为(p-1)*s



## 优化分页查询

**优化思路一**
在索引上完成排序分页操作，最后根据主键关联回原表查询所需要的其他列内容。

**优化思路二**
该方案适用于主键自增的表，可以把Limit 查询转换成某个位置的查询 。

```sql
SELECT * FROM tb_user_1 WHERE id> 900000 LIMIT 10;
相当于
SELECT * FROM tb_user_1 WHERE id BETWEEN 900001 AND 900010;
```



## 视图作用

```
是一种虚拟存在的表，行和列的数据来自定义视图的查询中使用的表，并且是在使用视图时动态生成的，只保存了sql逻辑，不保存查询结果。
实现了sql语句的重用
简化了复杂的sql操作，不必知道其细节
保护数据，提高安全性
```

不可更新的视图

- 包含以下关键字: 分组函数,DISTINCT,GROUP BY,HAVING,UNION,UNION ALL

- 常量视图，例: 

  ```sql
  CREATE VIEW v AS SELECT 'abc' NAME;
  ```

- SELECT 中包含子查询，例: 	

  ```sql
  CREATE VIEW v AS SELECT(SELECT 1) AS number;
  ```

- JOIN，例: 

  ```sql
  CREATE VIEW v AS 
  SELECT * FROM employees e JOIN departments d
  ON e.department_id=d.department_id;
  ```

- FROM 一个不能更新的视图，例：

  ```sql
  CREATE VIEW v2 AS 
  SELECT * FROM v;
  ```

- WHERE 子句的子查询引用了 FROM 子句中的表，例：

  ```sql
  CREATE VIEW v AS 
  SELECT * FROM employees
  WHERE employees_id IN (SELECT DISTINCT manager_id FROM employees);
  ```



## MYSQL优化

写日志的操作是磁盘上一小块区域内的顺序I/O，而不像随机I/O需要在磁盘的多个地方移动磁头，所以采用事务日志的方式相对来说要快得多

### undo log ： https://blog.csdn.net/harryptter/article/details/87388164

**插入**: 但通过事务**插入**新的数据时候，数据行版本号（DB_TRX_ID）为当前事务版本号，删除版本号设置为NULL。

**删除**：在删除版本号增加当前系统全局事务ID号

**修改** : 先做命中的数据行的复制,然后间原行数据的删除版本号设置为当前全局事务ID，新的行数据的数据行版本号也设置为当前全局事务ID，删除版本号为NULL

**查找** ：1.寻找数据行版本号小于或者等于当前全局事务ID（这样可以确认事务读取的行在开始前已经存在，或者是事务自身插入或者修改的），2.查找删除版本号为NULL，或者大于当前事务版本号的ID（即确保读取出来的行在此事务开启之前没有被删除）

## JDBC 流程

（1）导入相应的jar包
（2）加载、注册sql驱动
（3）获取Connection连接对象
（4）创建Statement对象并执行SQL语句
（5）使用ResultSet对象获取查询结果集
（6）依次关闭ResultSet、Statement、Connection对象

```java
public class JDBCUtils {
    //获取数据库连接
    public static Connection getConnection() throws Exception {
        //读取配置文件信息
        InputStream in = ClassLoader.getSystemClassLoader().getResourceAsStream("jdbc.properties");
        Properties properties=new Properties();
        properties.load(in);
        String driver = properties.getProperty("driver");
        String url = properties.getProperty("url");
        String user = properties.getProperty("user");
        String password = properties.getProperty("password");
        //注册驱动
        Class.forName(driver);
        //获取数据库连接
        return DriverManager.getConnection(url, user, password);
    }
```

```java
//通用的增删改操作
    public void commonUpdate(String sql,Object...args){
        Connection connect=null;
        PreparedStatement ps=null;
        try{
            connect = JDBCUtils.getConnection();
            ps = connect.prepareStatement(sql);
            for(int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);//数据库中索引从1开始
            }
            ps.execute();//返回值true代表查询 false代表增删改
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            JDBCUtils.closeConnection(connect,ps);
        }
    }
```

```java
//通用查询操作
    @Test
    public <T>List<T> query(Class<T> clazz,String sql,Object...args){
        Connection connect=null;
        PreparedStatement ps=null;
        ResultSet rs=null;
        List<T> result=new ArrayList<>();
        try{
            connect = JDBCUtils.getConnection();
            ps = connect.prepareStatement(sql);
            for(int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
            ResultSet resultSet = ps.executeQuery();
            ResultSetMetaData metaData = resultSet.getMetaData();
            int columnCount = metaData.getColumnCount();
            while (resultSet.next()){
                T t=clazz.newInstance();
               for(int i=0;i<columnCount;i++){
                   Object value = resultSet.getObject(i + 1);
                   String columnName = metaData.getColumnLabel(i + 1);
                   Field field = clazz.getDeclaredField(columnName);
                   field.setAccessible(true);
                   field.set(t,value);
               }
               result.add(t);
            }
            return result;
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            JDBCUtils.closeConnection(connect,ps,rs);
        }
        return null;
    }
```



-  PreparedStatement 和Statement的区别

> - PreparedStatement代码的可读性和可维护性更强
>
> - PreparedStatement 能实现更高效的批量操作
>
>   - DBServer会对预编译语句提供性能优化。因为预编译语句有可能被重复调用，所以语句在被DBServer的编译器编译后的执行代码被缓存下来，那么下次调用时只要是相同的预编译语句就不需要编译，只要将参数直接传入编译过的语句执行代码中就会得到执行。
>   - 在statement语句中,每执行一次都要对传入的语句编译一次。
>
> - PreparedStatement 可以防止 SQL 注入
>
> - PreparedStatement  可操作Blob类数据
>
>   

- Java与SQL对应数据类型转换表

| Java类型           | SQL类型                  |
| ------------------ | ------------------------ |
| boolean            | BIT                      |
| byte               | TINYINT                  |
| short              | SMALLINT                 |
| int                | INTEGER                  |
| long               | BIGINT                   |
| String             | CHAR,VARCHAR,LONGVARCHAR |
| byte   array       | BINARY  ,    VAR BINARY  |
| java.sql.Date      | DATE                     |
| java.sql.Time      | TIME                     |
| java.sql.Timestamp | TIMESTAMP                |

###  ResultSet

> - PreparedStatement 的 `executeQuery() `方法，查询结果是一个ResultSet 对象
> - ResultSet 对象以逻辑表格的形式封装了执行数据库操作的结果集，ResultSet 接口由数据库厂商提供实现
> - ResultSet 返回的实际上就是一张数据表，有一个指针指向数据表的第一条记录的前面。
> - ResultSet 对象维护了一个指向当前数据行的**游标**，初始的时候，游标在第一行之前，可以通过 ResultSet 对象的` next() `方法移动到下一行。调用 next()方法检测下一行是否存在。若存在，该方法返回
>   true，且指针下移，相当于Iterator对象的 `hasNext()` 和 `next() `方法的结合体。
> - 可以通过调用 getXxx()获取每一列的值。

   **注：Java与数据库交互涉及到的相关Java API中的索引都从1开始。**

### ResultSetMetaData

> - 可用于获取关于 ResultSet 对象中列的类型和属性信息的对象
> - 通过调用ResultSet对象的`getMetaData()`方法获得ResultSetMetaData对象
>   - **getColumnName**(int column)：获取指定列的名称
>   - **getColumnLabel**(int column)：获取指定列的别名
>   - **getColumnCount**()：返回当前 ResultSet 对象中的列数。

### 资源的释放

> - 释放ResultSet, Statement,Connection。
> - 数据库连接（Connection）是非常稀有的资源，用完后必须马上释放，如果Connection不能及时正确的关闭将导致系统问题。Connection的使用原则是**尽量晚创建，尽量早的释放。**
> - 可以在finally中关闭，保证及时其他代码出现异常，资源也一定能被关闭。

### JDBC API小结

- 两种思想

>   - 面向接口编程的思想
>   - ORM思想(object relational mapping)
>     - 一个数据表对应一个java类
>     - 表中的一条记录对应java类的一个对象
>     - 表中的一个字段对应java类的一个属性

- 两种技术

>   - JDBC结果集的元数据：ResultSetMetaData
>
>     - 获取列数：getColumnCount()
>     - 获取列的别名：getColumnLabel()
>
>   - 通过反射，创建指定类的对象，获取指定的属性并赋值
>
>     

在jdbc中有3种执行sql的语句分别是execute，executeQuery和executeUpdate

> execute执行增删改查操作
>
> execute返回的结果是个boolean型，当返回的是true的时候，表明有ResultSet结果集，通常是执行了select操作，当返回的是false时，通常是执行了insert、update、delete等操作。execute通常用于执行不明确的sql语句。
>
> 
>
> executeQuery执行查询操作
>
> executeQuery返回的是ResultSet结果集，通常是执行了select操作。
>
> 
>
> executeUpdate执行增删改操作
>
> executeUpdate返回的是int型，表明受影响的行数，通常是执行了insert、update、delete等操作。




## mysql 死锁，**什么情况下会造成死锁**？



## 如果在创建表时没有显式地定义主键,则InnoDB存储引擎会按如下方式选择或创建主键？

> 如果表没有主键，甚至都没有唯一键索引的话，InnoDB内部会基于一个包含了ROW_ID值的列生成一个隐式的聚簇索引，行都会根据这个ROW_ID排序。ROW_ID是一个6个字节，即48位的单调递增字段。有新数据插入时，就会生成一个新的递增的ROW_ID。所以，根据ROW_ID排序的行，本质上是按照插入顺序排序。

需要说明的是，这个单调递增的ROW_ID，和我们平时申明主键时指定AUTO_INREMENT的实现逻辑是**完全不一样**的。

- 隐式ROW_ID实现

这个在既没有主键，也没有一个非空唯一键的InnoDB表中自动添加的被称为ROW_ID的列，既不能被任何查询访问，也不能被内部（例如基于行的复制）使用。

、、、

##  MySQL主从复制原理的是啥？



https://blog.csdn.net/jaryle/article/details/52002727

MySQL里有一个概念，叫binlog日志，就是每个增删改类的操作，会改变数据的操作，除了更新数据以外，对这个增删改操作还会写入一个日志文件，记录这个操作的日志。

主库将变更写binlog日志，然后从库连接到主库之后，从库有一个IO线程，将主库的binlog日志拷贝到自己本地，写入一个中继日志中。接着从库中有一个SQL线程会从中继日志读取binlog，然后执行binlog日志中的内容，也就是在自己本地再次执行一遍SQL，这样就可以保证自己跟主库的数据是一样的。

这里有一个非常重要的一点，就是从库同步主库数据的过程是串行化的，也就是说主库上并行的操作，在从库上会串行执行。所以这就是一个非常重要的点了，由于从库从主库拷贝日志以及串行执行SQL的特点，在高并发场景下，从库的数据一定会比主库慢一些，是有延时的。所以经常出现，刚写入主库的数据可能是读不到的，要过几十毫秒，甚至几百毫秒才能读取到。而且这里还有另外一个问题，就是如果主库突然宕机，然后恰好数据还没同步到从库，那么有些数据可能在从库上是没有的，有些数据可能就丢失了。

所以mysql实际上在这一块有两个机制，一个是**半同步复制**，用来解决主库数据丢失问题；一个是**并行复制**，用来解决主从同步延时问题。

这个所谓半同步复制，semi-sync复制，指的就是主库写入binlog日志之后，就会将强制此时立即将数据同步到从库，从库将日志写入自己本地的relay log之后，接着会返回一个ack给主库，主库接收到至少一个从库的ack之后才会认为写操作完成了。

所谓并行复制，指的是从库开启多个线程，并行读取relay log中不同库的日志，然后并行重放不同库的日志，这是库级别的并行。

- 主从复制的原理
- 主从延迟问题产生的原因
- 主从复制的数据丢失问题，以及半同步复制的原理
- 并行复制的原理，多库并发重放relay日志，缓解主从延迟问题

![image-20200730005610522](/Users/ano/Desktop/notes/image/image-20200730005610522.png)



**半同步模式(mysql semi-sync)**
这种模式下主节点只需要接收到**其中一台**从节点的返回信息，就会commit；否则需要等待直到超时时间然后切换成异步模式再提交；这样做的目的可以使主从数据库的数据延迟缩小，可以提高数据安全性，确保了事务提交后，binlog至少传输到了一个从节点上，不能保证从节点将此事务更新到db中。性能上会有一定的降低，响应时间会变长。如下图所示：

![img](/Users/ano/Desktop/notes/image/v2-d9ac9c5493d1d772f5bf57ede089f0d5_1440w.jpg)



**l 全同步模式**
全同步模式是指主节点和从节点全部执行了commit并确认才会向客户端返回成功。

l Statement-base Replication (SBR)就是记录sql语句在bin log中，Mysql 5.1.4 及之前的版本都是使用的这种复制格式。优点是只需要记录会修改数据的sql语句到binlog中，减少了binlog日质量，节约I/O，提高性能。缺点是在某些情况下，会导致主从节点中数据不一致（比如sleep(),now()等）。


l Row-based Relication(RBR)是mysql master将SQL语句分解为基于Row更改的语句并记录在bin log中，也就是只记录哪条数据被修改了，修改成什么样。优点是不会出现某些特定情况下的**存储过程、或者函数、或者trigger的调用或者触发无法被正确复制的问题**。缺点是会产生大量的日志，尤其是修改table的时候会让日志暴增,同时增加bin log同步时间。也不能通过bin log解析获取执行过的sql语句，只能看到发生的data变更。

l Mixed-format Replication(MBR)，MySQL NDB cluster 7.3 和7.4 使用的MBR。是以上两种模式的混合，对于一般的复制使用STATEMENT模式保存到binlog，对于STATEMENT模式无法复制的操作则使用ROW模式来保存，MySQL会根据执行的SQL语句选择日志保存方式。

## InnoDB结构

InnoDB存储引擎由内存池和一些后台线程组成，其各自主要的工作是：

**内存池主要工作**

- 维护所有进程/线程需要访问的多个内部数据结构

- 缓存磁盘上的数据，方便快速读取，同时在对磁盘文件修改之前进行缓存

- 缓存重做日志（redo log）

  ![image-20200731135311965](/Users/ano/Desktop/notes/image/image-20200731135311965.png)

 **缓冲池**
InnoDB缓冲池是为了通过内存的速度来弥补磁盘速度慢对数据库性能造成的影响。其工作方式总是将数据库文件按页（每页16K）读取到缓冲池，然后按最近最少使用（LRU）的算法来保留在缓冲池中的缓存数据。在数据库中进行读操作时，首先将从磁盘读到的页存放在缓冲池中，下一次读取相同的页时，首先判定是否存在缓冲池中，如果有就是被命中直接读取，没有的话就从磁盘中读取。在数据库进行改操作时，首先修改缓冲池中的页（修改后，该页即为脏页），然后在以一定的频率刷新到磁盘上。这里的刷新机制不是每页在发生变更时触发。而是通过一种checkpoint机制刷新到磁盘的。
所以缓冲池的大小直接影响着数据库的整体性能，可以通过配置参数innodb_buffer_pool_size来设置。
从架构图中可以看出缓冲池中缓存的数据页类型有:索引页、数据页、 undo 页、插入缓冲、自适应哈希索引、 InnoDB 的锁信息、数据字典信息等。索引页和数据页占缓冲池的很大一部分。

- 数据页和索引页: Page是Innodb存储的最基本结构，也是Innodb磁盘管理的最小单位，与数据库相关的所有内容都存储在Page结构里。Page分为几种类型，数据页和索引页就是其中最为重要的两种类型。
- 插入缓存: 在InnoDB引擎上进行插入操作时，一般需要按照主键顺序进行插入，这样才能获得较高的插入性能。当一张表中存在非聚簇的且不唯一的索引时，在插入时，数据页的存放还是按照主键进行顺序存放，但是对于非聚簇索引叶节点的插入不再是顺序的了，这时就需要离散的访问非聚簇索引页，由于随机读取的存在导致插入操作性能下降。
  InnoDB为此设计了Insert Buffer来进行插入优化。对于非聚簇索引的插入或者更新操作，不是每一次都直接插入到索引页中，而是先判断插入的非聚集索引是否在缓冲池中，若在，则直接插入；若不在，则先放入到一个Insert Buffer中。看似数据库这个非聚集的索引已经查到叶节点，而实际没有，这时存放在另外一个位置。然后再以一定的频率和情况进行Insert Buffer和非聚簇索引页子节点的合并操作。这时通常能够将多个插入合并到一个操作中，这样就大大提高了对于非聚簇索引的插入性能。
- 自适应哈希索引: InnoDB会根据访问的频率和模式，为热点页建立哈希索引，来提高查询效率。InnoDB存储引擎会监控对表上各个索引页的查询，如果观察到建立哈希索引可以带来速度上的提升，则建立哈希索引，所以叫做自适应哈希索引。
  自适应哈希索引是通过缓冲池的B+树页构建而来，因此建立速度很快，而且不需要对整张数据表建立哈希索引。其 有一个要求，即对这个页的连续访问模式必须是一样的，也就是说其查询的条件(WHERE)必须完全一样，而且必须是连续的。
- 锁信息 : nnoDB存储引擎会在行级别上对表数据进行上锁。不过InnoDB也会在数据库内部其他很多地方使用锁，从而允许对多种不同资源提供并发访问。数据库系统使用锁是为了支持对共享资源进行并发访问，提供数据的完整性和一致性。关于锁的具体知识我们之后再进行详细学习。
- 数据字典信息 : InnoDB有自己的表缓存，可以称为表定义缓存或者数据字典。当InnoDB打开一张表，就增加一个对应的对象到数据字典。
  数据字典是对数据库中的数据、库对象、表对象等的元信息的集合。在MySQL中，数据字典信息内容就包括表结构、数据库名或表名、字段的数据类型、视图、索引、表字段信息、存储过程、触发器等内容。MySQL INFORMATION_SCHEMA库提供了对数据局元数据、统计信息、以及有关MySQL server的访问信息（例如：数据库名或表名，字段的数据类型和访问权限等）。该库中保存的信息也可以称为MySQL的数据字典。

**重做日志冲池**

InnoDB有buffer pool（简称bp）。bp是数据库页面的缓存，对InnoDB的任何修改操作都会首先在bp的page上进行，然后这样的页面将被标记为dirty并被放到专门的flush list上，后续将由master thread或专门的刷脏线程阶段性的将这些页面写入磁盘（disk or ssd）。这样的好处是避免每次写操作都操作磁盘导致大量的随机IO，阶段性的刷脏可以将多次对页面的修改merge成一次IO操作，同时异步写入也降低了访问的时延。然而，如果在dirty page还未刷入磁盘时，server非正常关闭，这些修改操作将会丢失，如果写入操作正在进行，甚至会由于损坏数据文件导致数据库不可用。为了避免上述问题的发生，Innodb将所有对页面的修改操作写入一个专门的文件，并在数据库启动时从此文件进行恢复操作，这个文件就是redo log file。这样的技术推迟了bp页面的刷新，从而提升了数据库的吞吐，有效的降低了访问时延。带来的问题是额外的写redo log操作的开销（顺序IO，当然很快），以及数据库启动时恢复操作所需的时间。

redo日志由两部分构成：redo log buffer、redo log file。innodb是支持事务的存储引擎，在事务提交时，必须先将该事务的所有日志写入到redo日志文件中，待事务的commit操作完成才算整个事务操作完成。在每次将redo log buffer写入redo log file后，都需要调用一次fsync操作，因为重做日志缓冲只是把内容先写入操作系统的缓冲系统中，并没有确保直接写入到磁盘上，所以必须进行一次fsync操作。因此，磁盘的性能在一定程度上也决定了事务提交的性能。

InnoDB 存储引擎先将重做日志信息放入这个缓冲区,然后以一定频率将其刷新到重做日志文件。重做日志文件一般不需要设置得很大,因为在下列三种情况下重做日志缓冲中的内容会刷新到磁盘的重做日志文件中。

- Master Thread 每一秒将重做日志缓冲刷新到重做日志文件
- 每个事物提交时会将重做日志缓冲刷新到重做日志文件
- 当重做日志缓冲剩余空间小于1/2时,重做日志缓冲刷新到重做日志文件

**额外的缓冲池**
在 InnoDB 存储引擎中，对一些数据结构本身的内存进行分配时，需要从额外的内存池中进行申请。例如: 分配了缓冲池,但是每个缓冲池中的帧缓冲还有对应的缓冲控制对象，这些对象记录以一些诸如 LRU, 锁,等待等信息,而这个对象的内存需要从额外的内存池中申请。

**后台线程主要工作**

- 刷新内存池中的数据，保证缓冲池中缓存的数据最新
- 将已修改数据文件刷新到磁盘文件
- 保证数据库异常时InnoDB能恢复到正常运行状态
- ![image-20200731140027071](/Users/ano/Desktop/notes/image/image-20200731140027071.png)



核心的后台线程，主要负责将缓冲池中的数据异步刷新到磁盘，保证数据的一致性，包括脏页的刷新、合并插入缓冲、undo页的回收等。
Master thread在主循环中，分两大部分操作，每秒钟的操作和每10秒钟的操作：

- **每秒一次的操作**

- - 日志缓冲刷新到磁盘: 即使这个事务还没有提交（总是），这点解释了为什么再大的事务commit时都很快；
  - 合并插入缓冲（可能）: 合并插入并不是每秒都发生，InnoDB会判断当前一秒内发生的IO次数是否小于5，如果是，则系统认为当前的IO压力很小，可以执行合并插入缓冲的操作。
  - 至多刷新100个InnoDB的缓冲池的脏页到磁盘（可能) : 这个刷新100个脏页也不是每秒都在做，InnoDB引擎通过判断当前缓冲池中脏页的比例(buf_get_modified_ratio_pct)是否超过了配置文件中innodb_max_drity_pages_pct参数(默认是90，即90%)，如果超过了这个阈值，InnoDB引擎认为需要做磁盘同步操作，将100个脏页写入磁盘。

- **每10秒一次的操作**

- - 刷新100个脏页到磁盘（可能）: InnoDB引擎先判断过去10秒内磁盘的IO操作是否小于200次，如果是，认为当前磁盘有足够的IO操作能力，即将100个脏页刷新到磁盘。
  - 合并至多5个插入缓冲（总是）: 此次的合并插入缓冲操作总会执行，不同于每秒操作时可能发生的合并操作。
  - 将日志缓冲刷新到磁盘（总是）: InnoDB引擎会再次执行日志缓冲刷新到磁盘的操作，与每秒发生的操作一样。
  - 删除无用的undo页（总是）: 当对表执行update，delete操作时，原先的行会被标记为删除，但是为了一致性读的关系，需保留这些行版本的信息，在进行10S一次的删除操作时，InnoDB引擎会判断当前事务系统中已被删除的行是否可以删除，如果可以，InnoDB会立即将其删除。InnoDB每次最多删除20个Undo页。
  - 产生一个检查点（checkpoing）；



**IO threads**
在 InnoDB 存储引擎中大量使用了异步 IO 来处理写 IO 请求，IO Thread 的工作主要是负责这些 IO 请求的回调.。分别为write、read、insert buffer和log IO thread。线程数量可以通过参数进行调整。5.6以后的版本可以通过innodb_write_io_threads和innodb_read_io_threads来限制读写线程，而在5.6版本以前，只有一个参数innodb_file_io_threads来控制读写总线程数。

**purge threads**
负责回收已经使用并分配的undo页，purge操作默认是由master thread中完成的，为了减轻master thread的工作，提高cpu使用率以及提升存储引擎的性能。用户可以在参数文件中添加如下命令来启动独立的purge thread。
innodb_purge_threads=1
从innodb1.2版本开始，可以指定多个innodb_purge_threads来进一步加快和提高undo回收速度。

**page cleaner threads**
Page Cleaner Thread是在InnoDB1.2.X版本中引入的。其作用是将之前版本中脏页的刷新操作都放入到单独的线程中来完成。 其目的是减轻master thread的工作以及对于用户查询线程的阻塞，进一步提高InnoDB存储引擎的性能。

## 三、InnoDB重要特性

MySQL InnoDB通过如下重要特性实现了更好的新能和更高的特性

- 插入缓冲（insert buffer）
- 两次写（Double write）
- 自适应哈希索引（adaptive hash index）
- 异步io（Async IO）
- 刷新领接页（Flush Neighbor Page）





# REDIS

## Redis 为何单线程

Redis 是单进程单线程的模型，因为 Redis 完全是基于内存的操作，CPU 不是 Redis 的瓶颈，Redis 的瓶颈最有可能是机器内存的大小或者网络带宽。单线程容易实现，而且 CPU 不会成为瓶颈，那就顺理成章的采用单线程的方案了（毕竟采用多线程会有很多麻烦）。

## Redis 和 Memcached 的区别

- **存储方式上：**Memcache 会把数据全部存在内存之中，断电后会挂掉，数据不能超过内存大小。Redis 有部分数据存在硬盘上，这样能保证数据的持久性。
- **数据支持类型上：**Memcache 对数据类型的支持简单，只支持简单的 key-value，而 Redis 支持五种数据类型。
- **使用底层模型不同：**它们之间底层实现方式以及与客户端之间通信的应用协议不一样。Redis 直接自己构建了 VM 机制，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。
- **Value 的大小：**Redis 可以达到 1GB，而 Memcache 只有 1MB。

## Redis 为何这么快

- Redis 完全基于内存，绝大部分请求是纯粹的内存操作，非常迅速，数据存在内存中。

- 数据结构简单，对数据操作也简单。

- 采用单线程，避免了不必要的上下文切换和竞争条件，不存在多线程导致的 CPU 切换，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有死锁问题导致的性能消耗。

- 使用多路复用 IO 模型，非阻塞 IO。

  

## Redis 的淘汰策略

![image-20200623114805276](/Users/ano/Desktop/notes/image/image-20200623114805276.png)



## Redis 的持久化机制了解吗？

Redis 为了保证效率，数据缓存在了内存中，但是会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件中，以保证数据的持久化。

Redis 的持久化策略有两种：

- **RDB：**快照形式是直接把内存中的数据保存到一个 dump 的文件中，定时保存，保存策略。

- **AOF：**把所有的对 Redis 的服务器进行修改的命令都存到一个文件里，命令的集合。

  Redis 默认是快照 RDB 的持久化方式。

当 Redis 重启的时候，它会优先使用 AOF 文件来还原数据集，因为 AOF 文件保存的数据集通常比 RDB 文件所保存的数据集更完整。你甚至可以关闭持久化功能，让数据只在服务器运行时存。

## RDB 是怎么工作的？

Redis 是会以快照"RDB"的形式将数据持久化到磁盘的一个二进制文件 dump.rdb。

工作原理：当 Redis 需要做持久化时，Redis 会 fork 一个子进程，子进程将数据写到磁盘上一个临时 RDB 文件中。当子进程完成写临时文件后，将原来的 RDB 替换掉，这样的好处是可以 copy-on-write。

优点：这种文件非常适合用于备份：比如，可以在最近的 24 小时内，每小时备份一次，并且在每个月的每一天也备份一个 RDB 文件。

这样的话，即使遇上问题，也可以随时将数据集还原到不同的版本。RDB 非常适合灾难恢复。

缺点：可能丢失数据。

### save&BGSave

https://draveness.me/whys-the-design-redis-bgsave-fork/

其中 `SAVE` 命令在执行时会直接阻塞当前的线程，由于 Redis 是 [单线程](https://draveness.me/whys-the-design-redis-single-thread) 的，所以 `SAVE` 命令会直接阻塞来自客户端的所有其他请求，这在很多时候对于需要提供较强可用性保证的 Redis 服务都是无法接受的。我们往往需要 `BGSAVE` 命令在后台生成 Redis 全部数据对应的 RDB 文件，当我们使用 `BGSAVE` 命令时，Redis 会立刻 `fork` 出一个子进程，子进程会执行『将内存中的数据以 RDB 格式保存到磁盘中』这一过程，而 Redis 服务在 `BGSAVE` 工作期间仍然可以处理来自客户端的请求。

## Redis 为什么在使用 RDB 进行快照时会通过子进程的方式进行实现：

1. 通过 `fork` 创建的子进程能够获得和父进程完全相同的内存空间，父进程对内存的修改对于子进程是不可见的，两者不会相互影响；
2. 通过 `fork` 创建子进程时不会立刻触发大量内存的拷贝，内存在被修改时会以页为单位进行拷贝，这也就避免了大量拷贝内存而带来的性能问题；



## AOF 怎么工作

```java
appendfsync yes   
appendfsync always     #每次有数据修改发生时都会写入AOF文件。
appendfsync everysec   #每秒钟同步一次，该策略为AOF的缺省策略。
```

AOF 可以做到全程持久化，只需要在配置中开启 appendonly yes。这样 Redis 每执行一个修改数据的命令，都会把它添加到 AOF 文件中，当 Redis 重启时，将会读取 AOF 文件进行重放，恢复到 Redis 关闭前的最后时刻。

优点： 让 Redis 变得非常耐久。可以设置不同的 Fsync 策略，AOF的默认策略是每秒钟 Fsync 一次，在这种配置下，就算发生故障停机，也最多丢失一秒钟的数据。

缺点：相同的数据集来说，AOF 的文件体积通常要大于 RDB 文件的体积。根据所使用的 Fsync 策略，AOF 的速度可能会慢于 RDB。

https://www.cnblogs.com/cc11001100/p/6487158.html

so 

如果你非常关心你的数据，但仍然可以承受数分钟内的数据丢失，那么可以只使用 RDB 持久化。

AOF 将 Redis 执行的每一条命令追加到磁盘中，处理巨大的写入会降低Redis的性能。

数据库备份和灾难恢复：定时生成 RDB 快照非常便于进行数据库备份，并且 RDB 恢复数据集的速度也要比 AOF 恢复的速度快。当然了，Redis 支持同时开启 RDB 和 AOF，系统重启后，Redis 会优先使用 AOF 来恢复数据，这样丢失的数据会最少。

AOF 后台重写

当子进程在执行 AOF 重写时， 主进程需要执行以下三个工作：

1. 处理命令请求。
2. 将写命令追加到现有的 AOF 文件中。
3. 将写命令追加到 AOF 重写缓存中。

这样一来可以保证：

1. 现有的 AOF 功能会继续执行，即使在 AOF 重写期间发生停机，也不会有任何数据丢失。
2. 所有对数据库进行修改的命令都会被记录到 AOF 重写缓存中。

## Redis 雪崩了解吗?

同一时间大面积失效，瞬间 Redis 跟没有一样.

Solution: 在批量往 Redis 存数据的时候，把每个 Key 的失效时间都加个随机值就好了，这样可以保证数据不会再同一时间大面积失效。

缓存雪崩的事前事中事后的解决方案如下。

- 事前：redis 高可用，主从+哨兵，redis cluster，避免全盘崩溃。
- 事中：本地 ehcache 缓存 + hystrix 限流&降级，避免 MySQL 被打死。
- 事后：redis 持久化，一旦重启，自动从磁盘上加载数据，快速恢复缓存数据。



## 缓存穿透和击穿?

缓存穿透是指缓存和数据库中都没有的数据，而用户（黑客）不断发起请求。

example : id 都是从 1 自增的，如果发起 id=-1 的数据或者 id 特别大不存在的数据，这样的不断攻击导致数据库压力很大，严重会击垮数据库。

solution : 在接口层增加校验，比如用户鉴权，参数做校验，不合法的校验直接 return，比如 id 做基础校验，id<=0 直接拦截。布隆过滤器（Bloom Filter）也能很好的预防缓存穿透的发生。

缓存击穿指一个 Key 非常热点，在不停地扛着大量的请求，大并发集中对这一个点进行访问，当这个 Key 在失效的瞬间，持续的大并发直接落到了数据库上，就在这个 Key 的点上击穿了缓存。

solution ： 设置热点数据永不过期，或者加上互斥锁就搞定了

<img src="/Users/ano/Desktop/notes/image/image-20200710000746609.png" alt="image-20200710000746609" style="zoom:50%;" />



## REDIS   HA 

持久化、复制、哨兵和集群，其主要作用和解决的问题是：

- 持久化：持久化是最简单的高可用方法(有时甚至不被归为高可用的手段)，主要作用是数据备份，即将数据存储在硬盘，保证数据不会因进程退出而丢失。
- 复制：复制是高可用Redis的基础，哨兵和集群都是在复制基础上实现高可用的。复制主要实现了数据的多机备份，以及对于读操作的负载均衡和简单的故障恢复。缺陷：故障恢复无法自动化；写操作无法负载均衡；存储能力受到单机的限制。
- 哨兵：在复制的基础上，哨兵实现了自动化的故障恢复。缺陷：写操作无法负载均衡；存储能力受到单机的限制。
- 集群：通过集群，Redis解决了写操作无法负载均衡，以及存储能力受到单机限制的问题，实现了较为完善的高可用方案。

### 主从

#### 主从复制过程/作用？ https://zhuanlan.zhihu.com/p/60239657

复制过程：                                      

- 从节点执行 slaveof [masterIP] [masterPort]，保存主节点信息。
- 从节点中的定时任务发现主节点信息，建立和主节点的 Socket 连接。
- 从节点发送 Ping 信号，主节点返回 Pong，两边能互相通信。
- 连接建立后，主节点将所有数据发送给从节点（数据同步）。
- 主节点把当前的数据同步给从节点后，便完成了复制的建立过程。接下来，主节点就会持续的把写命令发送给从节点，保证主从数据一致性。
- ![image-20200623151253153](/Users/ano/Desktop/notes/image/image-20200623151253153.png)

#### **psync 命令需要 3 个组件支持：**

1.主从节点各自复制偏移量
2.主节点复制积压缓冲区
3.主节点运行 ID

#### 主从复制会存在以下问题：

- 一旦主节点宕机，从节点晋升为主节点，同时需要修改应用方的主节点地址，还需要命令所有从节点去复制新的主节点，整个过程需要人工干预。

- 主节点的写能力受到单机的限制。

- 主节点的存储能力受到单机的限制。

- 原生复制的弊端在早期的版本中也会比较突出，比如：Redis 复制中断后，从节点会发起 psync。
  此时如果同步不成功，则会进行全量同步，主库执行全量备份的同时，可能会造成毫秒或秒级的卡顿。

  

### **主从复制的作用**

主从复制的作用主要包括：

1. 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
2. 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。
3. 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。
4. 高可用基石：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。



## Redis 的过期策略是如何实现的？

redisDb 结构的 expire 字典（过期字典）保存了所有键的过期时间，过期字典的键是一个指向键空间中的某个键对象的指针，过期字典的值保存了键所指向的数据库键的过期时间

![image-20200728015102720](/Users/ano/Desktop/notes/image/image-20200728015102720.png)

### 哨兵

#### 哨兵有哪些功能

Redis Sentinel（哨兵）主要功能包括**主节点存活检测、主从运行情况检测、自动故障转移、主从切换**。Redis Sentinel 最小配置是一主一从。Redis 的 Sentinel 系统可以用来管理多个 Redis 服务器。

该系统可以执行以下四个任务：

- **监控：**不断检查主服务器和从服务器是否正常运行。

- **通知：**当被监控的某个 Redis 服务器出现问题，Sentinel 通过 API 脚本向管理员或者其他应用程序发出通知。

- **自动故障转移：**当主节点不能正常工作时，Sentinel 会开始一次自动的故障转移操作，它会将与失效主节点是主从关系的其中一个从节点升级为新的主节点，并且将其他的从节点指向新的主节点，这样人工干预就可以免了。

- **配置提供者：**在 Redis Sentinel 模式下，客户端应用在初始化时连接的是 Sentinel 节点集合，从中获取主节点的信息。

  

#### 哨兵工作原理 https://juejin.im/post/5b7d226a6fb9a01a1e01ff64

![image-20200623200842696](/Users/ano/Desktop/notes/image/image-20200623200842696.png)

哨兵系统的搭建过程，有几点需要注意：

（1）哨兵系统中的主从节点，与普通的主从节点并没有什么区别，故障发现和转移是由哨兵来控制和完成的。

（2）哨兵节点本质上是redis节点。

（3）每个哨兵节点，只需要配置监控主节点，便可以自动发现其他的哨兵节点和从节点。

（4）在哨兵节点启动和故障转移阶段，各个节点的配置文件会被重写(config rewrite)。



##### 哨兵节点支持的命令

哨兵节点作为运行在特殊模式下的redis节点，其支持的命令与普通的redis节点不同。在运维中，我们可以通过这些命令查询或修改哨兵系统；不过更重要的是，哨兵系统要实现故障发现、故障转移等各种功能，离不开哨兵节点之间的通信，而通信的很大一部分是通过哨兵节点支持的命令来实现的。下面介绍哨兵节点支持的主要命令。

（1）基础查询：通过这些命令，可以查询哨兵系统的拓扑结构、节点信息、配置信息等。

- info sentinel：获取监控的所有主节点的基本信息
- sentinel masters：获取监控的所有主节点的详细信息
- sentinel master mymaster：获取监控的主节点mymaster的详细信息
- sentinel slaves mymaster：获取监控的主节点mymaster的从节点的详细信息
- sentinel sentinels mymaster：获取监控的主节点mymaster的哨兵节点的详细信息
- sentinel get-master-addr-by-name mymaster：获取监控的主节点mymaster的地址信息，前文已有介绍
- sentinel is-master-down-by-addr：哨兵节点之间可以通过该命令询问主节点是否下线，从而对是否客观下线做出判断

（2）增加/移除对主节点的监控

sentinel monitor mymaster2 192.168.92.128 16379 2：与部署哨兵节点时配置文件中的sentinel monitor功能完全一样，不再详述

sentinel remove mymaster2：取消当前哨兵节点对主节点mymaster2的监控

（3）强制故障转移

sentinel failover mymaster：该命令可以**强制对****mymaster****执行故障转移，**即便当前的主节点运行完好；例如，如果当前主节点所在机器即将报废，便可以提前通过failover命令进行故障转移。

## Redis 集群

![image-20200718220229933](/Users/ano/Desktop/notes/image/image-20200718220229933.png)

这里要先将节点握手讲清楚。我们让两个redis节点之间进行通信的时候，需要在客户端执行下面一个命令

```
127.0.0.1:7000>cluster meet 127.0.0.1:7001
```

如下图所示
![img](/Users/ano/Desktop/notes/image/725429-20190829164714480-362698612.png)

意思很简单，让7000节点和7001节点知道彼此存在！
在握手成功后，连个节点之间会**定期**发送ping/pong消息，交换**数据信息**，如下图所示。
![img](/Users/ano/Desktop/notes/image/725429-20190829164722650-2064735559.png)

在这里，我们需要关注三个重点。

- (1)交换什么数据信息
- (2)数据信息究竟多大
- (3)定期的频率什么样

*到底在交换什么数据信息？*
交换的数据信息，由消息体和消息头组成。
消息体无外乎是一些节点标识啊，IP啊，端口号啊，发送时间啊。
我们来看消息头，结构如下
![img](/Users/ano/Desktop/notes/image/725429-20190829164739879-973731722.jpg)

注意看红框的内容，type表示消息类型。
另外，消息头里面有个myslots的char数组，长度为16383/8，这其实是一个bitmap,每一个位代表一个槽，如果该位为1，表示这个槽是属于这个节点的。

### 为啥16384个节点

https://www.cnblogs.com/rjzheng/p/11430592.html

(1)如果槽位为65536，发送心跳信息的消息头达8k，发送的心跳包过于庞大。
如上所述，在消息头中，最占空间的是`myslots[CLUSTER_SLOTS/8]`。
当槽位为65536时，这块的大小是:
`65536÷8÷1024=8kb`
因为每秒钟，redis节点需要发送一定数量的ping消息作为心跳包，如果槽位为65536，这个ping消息的消息头太大了，浪费带宽。
(2)redis的集群主节点数量基本不可能超过1000个。
如上所述，集群节点越多，心跳包的消息体内携带的数据越多。如果节点过1000个，也会导致网络拥堵。因此redis作者，不建议redis cluster节点数量超过1000个。
那么，对于节点数在1000以内的redis cluster集群，16384个槽位够用了。没有必要拓展到65536个。
(3)槽位越小，节点少的情况下，压缩比高
Redis主节点的配置信息中，它所负责的哈希槽是通过一张bitmap的形式来保存的，在传输过程中，会对bitmap进行压缩，但是如果bitmap的填充率slots / N很高的话(N表示节点数)，bitmap的压缩率就很低。
如果节点数很少，而哈希槽数量很多的话，bitmap的压缩率就很低。

**Failover的流程**

一、主观下线

集群中每个节点都会定期向其他节点发送ping消息，接收节点回复pong消息作为响应。如果在cluster-node-timeout时间内通信一直失败，则发送节点会认为接收节点存在故障，把接收节点标记为主观下线（pfail）状态。


二、客观下线

当某个节点判断另一个节点主观下线后，相应的节点状态会跟随消息在集群内传播。通过Gossip消息传播，集群内节点不断收集到故障节点的下线报告。当半数以上持有槽的主节点都标记某个节点是主观下线时，触发客观下线流程。

集群中的节点每次接收到其他节点的pfail状态，都会尝试触发客观下线，流程说明：

\1. 首先统计有效的下线报告数量，如果小于集群内持有槽的主节点总数的一半则退出。

\2. 当下线报告大于槽主节点数量一半时，标记对应故障节点为客观下线状态。

\3. 向集群广播一条fail消息，通知所有的节点将故障节点标记为客观下线，fail消息的消息体只包含故障节点的ID。广播fail消息是客观下线的最后一步，它承担着非常重要的职责：

​	\1. 通知集群内所有的节点标记故障节点为客观下线状态并立刻生效。

​	\2. 通知故障节点的从节点触发故障转移流程。 

故障切换

故障节点变为客观下线后，如果下线节点是持有槽的主节点则需要在它的从节点中选出一个替换它，从而保证集群的高可用。下线主节点的所有从节点承担故障恢复的义务，当从节点通过内部定时任务发现自身复制的主节点进入客观下线时，将会触发故障切换流程。

1.资格检查

每个从节点都要检查最后与主节点断线时间，判断是否有资格替换故障的主节点。如果从节点与主节点断线时间超过cluster-node-time*cluster-slave-validity-factor，则当前从节点不具备故障转移资格。参数cluster-slavevalidity-factor用于从节点的有效因子，默认为10。

2.准备选举时间

当从节点符合故障切换资格后，更新触发切换选举的时间，只有到达该时间后才能执行后续流程。

这里之所以采用延迟触发机制，主要是通过对多个从节点使用不同的延迟选举时间来支持优先级问题。复制偏移量越大说明从节点延迟越低，那么它应该具有更高的优先级来替换故障主节点。

3.发起选举

当从节点定时任务检测到达故障选举时间（failover_auth_time）到达后，发起选举流程如下：

1> 更新配置纪元

2> 广播选举消息

在集群内广播选举消息（FAILOVER_AUTH_REQUEST），并记录已发送过消息的状态，保证该从节点在一个配置纪元内只能发起一次选举。

4.选举投票

只有持有槽的主节点才会处理故障选举消息（FAILOVER_AUTH_REQUEST），因为每个持有槽的节点在一个配置纪元内都有唯一的一张选票，当接到第一个请求投票的从节点消息时回复FAILOVER_AUTH_ACK消息作为投票，之后相同配置纪元内其他从节点的选举消息将忽略。

Redis集群没有直接使用从节点进行领导者选举，主要因为从节点数必须大于等于3个才能保证凑够N/2+1个节点，将导致从节点资源浪费。使用集群内所有持有槽的主节点进行领导者选举，即使只有一个从节点也可以完成选举过程。

5.替换主节点

当从节点收集到足够的选票之后，触发替换主节点操作：

1> 当前从节点取消复制变为主节点。

2> 执行clusterDelSlot操作撤销故障主节点负责的槽，并执行clusterAddSlot把这些槽委派给自己。

3> 向集群广播自己的pong消息，通知集群内所有的节点当前从节点变为主节点并接管了故障主节点的槽信息。

## 布隆过滤器原理？

利用高效的数据结构和算法快速判断出你这个 Key 是否在数据库中存在，不存在你 return 就好了，存在你就去查 DB 刷新 KV 再 return。

### 场景

- 数据库防止穿库。 Google Bigtable，HBase 和 Cassandra 以及 Postgresql 使用BloomFilter来减少不存在的行或列的磁盘查找。避免代价高昂的磁盘查找会大大提高数据库查询操作的性能。
- 业务场景中判断用户是否阅读过某视频或文章，比如抖音或头条，当然会导致一定的误判，但不会让用户看到重复的内容。还有之前自己遇到的一个比赛类的社交场景中，需要判断用户是否在比赛中，如果在则需要更新比赛内容，也可以使用布隆过滤器，可以减少不在的用户查询db或缓存的次数。
- 缓存宕机、缓存击穿场景，一般判断用户是否在缓存中，如果在则直接返回结果，不在则查询db，如果来一波冷数据，会导致缓存大量击穿，造成雪崩效应，这时候可以用布隆过滤器当缓存的索引，只有在布隆过滤器中，才去查询缓存，如果没查询到，则穿透到db。如果不在布隆器中，则直接返回。
- WEB拦截器，如果相同请求则拦截，防止重复被攻击。用户第一次请求，将请求参数放入布隆过滤器中，当第二次请求时，先判断请求参数是否被布隆过滤器命中。可以提高缓存命中率。



redis对事务是部分支持的，如果是在入队时报错，那么都不会执行；在非入队时报错，那么成功的就会成功执行。



## redis事务三大特性：

单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
没有隔离级别的概念：队列中的命令没有提交之前都不会实际的被执行，因为事务提交前任何指令都不会被实际执行，也就不存在”事务内的查询要看到事务里的更新，在事务外查询不能看到”这个让人万分头痛的问题
不保证原子性：redis同一个事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚

## 数据类型

### sds

- Redis 的字符串表示为 `sds` ，而不是 C 字符串（以 `\0` 结尾的 `char*`）。

- 对比 C 字符串，sds有以下特性：

  - 可以高效地执行长度计算（`strlen`）；
  - 可以高效地执行追加操作（`append`）；
  - 二进制安全；

- `sds` 会为追加操作进行优化：加快追加操作的速度，并降低内存分配的次数，代价是多占用了一些内存，而且这些内存不会被主动释放

  

### 双端链表

Redis 实现了自己的双端链表结构。双端链表还是 Redis 列表类型的底层实现之一， 当对列表类型的键进行操作 —— 比如执行 [RPUSH](http://redis.readthedocs.org/en/latest/list/rpush.html#rpush) 、 [LPOP](http://redis.readthedocs.org/en/latest/list/lpop.html#lpop) 或 [LLEN](http://redis.readthedocs.org/en/latest/list/llen.html#llen) 等命令时， 程序在底层操作的可能就是双端链表。

双端链表主要有两个作用：

- 作为 Redis 列表类型的底层实现之一；
- 作为通用数据结构，被其他功能模块所使用；

双端链表及其节点的性能特性如下：

- 节点带有前驱和后继指针，访问前驱节点和后继节点的复杂度为 O(1) ，并且对链表的迭代可以在从表头到表尾和从表尾到表头两个方向进行；

- 链表带有指向表头和表尾的指针，因此对表头和表尾进行处理的复杂度为 O(1) ；

- 链表带有记录节点数量的属性，所以可以在 O(1) 复杂度内返回链表的节点数量（长度）；

  

### dict

 大部分针对数据库的命令， 比如 [DBSIZE](http://redis.readthedocs.org/en/latest/server/dbsize.html#dbsize) 、 [FLUSHDB](http://redis.readthedocs.org/en/latest/server/flushdb.html#flushdb) 、 [RANDOMKEY](http://redis.readthedocs.org/en/latest/key/randomkey.html#randomkey) ， 等等， 都是构建于对字典的操作之上的； 而那些创建、更新、删除和查找键值对的命令， 也无一例外地需要在键空间上进行操作。dict也用作 Hash 类型键的底层实现之一；

- 字典是由键值对构成的抽象数据结构。

- Redis 中的数据库和哈希键都基于字典来实现。

- Redis 字典的底层实现为哈希表，每个字典使用两个哈希表，一般情况下只使用 0 号哈希表，只有在 rehash 进行时，才会同时使用 0 号和 1 号哈希表。

- 哈希表使用链地址法来解决键冲突的问题。

- Rehash 可以用于扩展或收缩哈希表。

- 对哈希表的 rehash 是分多次、渐进式地进行的。

- 字典的 rehash 操作实际上就是执行以下任务：

  1. 创建一个比 `ht[0]->table` 更大的 `ht[1]->table` ；
  2. 将 `ht[0]->table` 中的所有键值对迁移到 `ht[1]->table` ；
  3. 将原有 `ht[0]` 的数据清空，并将 `ht[1]` 替换为新的 `ht[0]` ；

  

### 跳跃表

- 跳跃表是一种随机化数据结构，查找、添加、删除操作都可以在对数期望时间下完成。
- 跳跃表目前在 Redis 的唯一作用，就是作为有序集类型的底层数据结构（之一，另一个构成有序集的结构是字典）。
- 为了满足自身的需求，Redis 基于 William Pugh 论文中描述的跳跃表进行了修改，包括：
  1. `score` 值可重复。
  2. 对比一个元素需要同时检查它的 `score` 和 `memeber` 。
  3. 每个节点带有高度为 1 层的后退指针，用于从表尾方向向表头方向迭代。

### 整数集合（intset）

* Intset 用于有序、无重复地保存多个整数值，会根据元素的值，自动选择该用什么长度的整数类型来保存元素。

* 当一个位长度更长的整数值添加到 intset 时，需要对 intset 进行升级，新 intset 中每个元素的位长度，会等于新添加值的位长度，但原有元素的值不变。

* 升级会引起整个 intset 进行内存重分配，并移动集合中的所有元素，这个操作的复杂度为 O(N)O(N) 。

* Intset 只支持升级，不支持降级。

* Intset 是有序的，程序使用二分查找算法来实现查找操作，复杂度为 O(lgN)O(lg⁡N) 。



### Ziplist

因为 ziplist 节约内存的性质， 哈希键、列表键和有序集合键初始化的底层实现皆采用 ziplist.

ziplist 是由一系列特殊编码的内存块构成的列表，可以保存字符数组或整数值，同时是哈希键、列表键和有序集合键的底层实现之一。



![image-20200708201423179](/Users/ano/Desktop/notes/image/image-20200708201423179.png)

为什么Hash与List会使用ziplist来存储数据呢？

因为

1. ziplist会比hashtable与ziplist节省跟多的内存
2. 内存中以连续块方式保存的数据比起hashtable与linkedlist使用的链表可以更快的载入缓存中
3. 当ziplist的长度比较小的时候，从ziplist读写数据的效率比hashtable或者linkedlist的差异并不大。

本质上，使用ziplist就是以时间换空间的一种优化，但是他的时间损坏小到几乎可以忽略不计，但却能带来可观的内存减少，所以满足条件时，Redis会使用ziplist作为Hash与List的存储结构。
链接：https://www.jianshu.com/p/fc9d65b5bb9a

### RedisObject

![image-20200708200804376](/Users/ano/Desktop/notes/image/image-20200708200804376.png)

### watch

![image-20200709205220653](/Users/ano/Desktop/notes/image/image-20200709205220653.png)

通过 `watched_keys` 字典， 如果程序想检查某个键是否被监视， 那么它只要检查字典中是否存在这个键即可； 如果程序要获取监视某个键的所有客户端， 那么只要取出键的值（一个链表）， 然后对链表进行遍历即可。

在任何对数据库键空间（key space）进行修改的命令成功执行之后 （比如 [FLUSHDB](http://redis.readthedocs.org/en/latest/server/flushdb.html#flushdb) 、 [SET](http://redis.readthedocs.org/en/latest/string/set.html#set) 、 [DEL](http://redis.readthedocs.org/en/latest/key/del.html#del) 、 [LPUSH](http://redis.readthedocs.org/en/latest/list/lpush.html#lpush) 、 [SADD](http://redis.readthedocs.org/en/latest/set/sadd.html#sadd) 、 [ZREM](http://redis.readthedocs.org/en/latest/sorted_set/zrem.html#zrem) ，诸如此类）， `multi.c/touchWatchedKey` 函数都会被调用 —— 它检查数据库的 `watched_keys` 字典， 看是否有客户端在监视已经被命令修改的键， 如果有的话， 程序将所有监视这个/这些被修改键的客户端的 `REDIS_DIRTY_CAS` 选项打开

当客户端发送 [EXEC](http://redis.readthedocs.org/en/latest/transaction/exec.html#exec) 命令、触发事务执行时， 服务器会对客户端的状态进行检查：

- 如果客户端的 `REDIS_DIRTY_CAS` 选项已经被打开，那么说明被客户端监视的键至少有一个已经被修改了，事务的安全性已经被破坏。服务器会放弃执行这个事务，直接向客户端返回空回复，表示事务执行失败。
- 如果 `REDIS_DIRTY_CAS` 选项没有被打开，那么说明所有监视键都安全，服务器正式执行事务。

最后，当一个客户端结束它的事务时，无论事务是成功执行，还是失败， `watched_keys` 字典中和这个客户端相关的资料都会被清除。



## 事务

Redis 事务保证了其中的一致性（C）和隔离性（I），但并不保证原子性（A）和持久性（D）。

A：

单个 Redis 命令的执行是原子性的，但 Redis 没有在事务上增加任何维持原子性的机制，所以 Redis 事务的执行并不是原子性的。如果一个事务队列中的所有命令都被成功地执行，那么称这个事务执行成功。另一方面，如果 Redis 服务器进程在执行事务的过程中被停止 —— 比如接到 KILL 信号、宿主机器停机，等等，那么事务执行失败。当事务失败时，Redis 也不会进行任何的重试或者回滚动作。

C:

* 入队错误:带有不正确入队命令的事务不会被执行，也不会影响数据库的一致性。

* 执行的过程中发生错误 : 比如说，对一个不同类型的 key 执行了错误的操作， 那么 Redis 只会将错误包含在事务的结果中， 这不会引起事务中断或整个失败，不会影响已执行事务命令的结果，也不会影响后面要执行的事务命令， 所以它对事务的一致性也没有影响。

  

* Redis 进程被终结： 如果 Redis 服务器进程在执行事务的过程中被其他进程终结，或者被管理员强制杀死，那么根据 Redis 所使用的持久化模式，可能有以下情况出现：

  - 内存模式：如果 Redis 没有采取任何持久化机制，那么重启之后的数据库总是空白的，所以数据总是一致的。

  - RDB 模式：在执行事务时，Redis 不会中断事务去执行保存 RDB 的工作，只有在事务执行之后，保存 RDB 的工作才有可能开始。所以当 RDB 模式下的 Redis 服务器进程在事务中途被杀死时，事务内执行的命令，不管成功了多少，都不会被保存到 RDB 文件里。恢复数据库需要使用现有的 RDB 文件，而这个 RDB 文件的数据保存的是最近一次的数据库快照（snapshot），所以它的数据可能不是最新的，但只要 RDB 文件本身没有因为其他问题而出错，那么还原后的数据库就是一致的。

  - AOF 模式：因为保存 AOF 文件的工作在后台线程进行，所以即使是在事务执行的中途，保存 AOF 文件的工作也可以继续进行，因此，根据事务语句是否被写入并保存到 AOF 文件，有以下两种情况发生：

    1）如果事务语句未写入到 AOF 文件，或 AOF 未被 SYNC 调用保存到磁盘，那么当进程被杀死之后，Redis 可以根据最近一次成功保存到磁盘的 AOF 文件来还原数据库，只要 AOF 文件本身没有因为其他问题而出错，那么还原后的数据库总是一致的，但其中的数据不一定是最新的。

    2）如果事务的部分语句被写入到 AOF 文件，并且 AOF 文件被成功保存，那么不完整的事务执行信息就会遗留在 AOF 文件里，当重启 Redis 时，程序会检测到 AOF 文件并不完整，Redis 会退出，并报告错误。需要使用 redis-check-aof 工具将部分成功的事务命令移除之后，才能再次启动服务器。还原之后的数据总是一致的，而且数据也是最新的（直到事务执行之前为止）。

    

## 订阅与发布模式

![image-20200709211009546](/Users/ano/Desktop/notes/image/image-20200709211009546.png)

- 订阅信息由服务器进程维持的 `redisServer.pubsub_channels` 字典保存，字典的键为被订阅的频道，字典的值为订阅频道的所有客户端。
- 当有新消息发送到频道时，程序遍历频道（键）所对应的（值）所有客户端，然后将消息发送到所有订阅频道的客户端上。
- 订阅模式的信息由服务器进程维持的 `redisServer.pubsub_patterns` 链表保存，链表的每个节点都保存着一个 `pubsubPattern` 结构，结构中保存着被订阅的模式，以及订阅该模式的客户端。程序通过遍历链表来查找某个频道是否和某个模式匹配。
- 当有新消息发送到频道时，除了订阅频道的客户端会收到消息之外，所有订阅了匹配频道的模式的客户端，也同样会收到消息。
- 退订频道和退订模式分别是订阅频道和订阅模式的反操作。



## LUA 脚本

- 初始化 Lua 脚本环境需要一系列步骤，其中最重要的包括：
  - 创建 Lua 环境。
  - 载入 Lua 库，比如字符串库、数学库、表格库，等等。
  - 创建 `redis` 全局表格，包含各种对 Redis 进行操作的函数，比如 `redis.call` 和 `redis.log` ，等等。
  - 创建一个无网络连接的伪客户端，专门用于执行 Lua 脚本中的 Redis 命令。
- Reids 通过一系列措施保证被执行的 Lua 脚本无副作用，也没有有害的写随机性：对于同样的输入参数和数据集，总是产生相同的写入命令。
- [EVAL](http://redis.readthedocs.org/en/latest/script/eval.html#eval) 命令为输入脚本定义一个 Lua 函数，然后通过执行这个函数来执行脚本。
- [EVALSHA](http://redis.readthedocs.org/en/latest/script/evalsha.html#evalsha) 通过构建函数名，直接调用 Lua 中已定义的函数，从而执行相应的脚本。

## Redis 慢查询

- 针对慢查询日志有三种操作，分别是查看、清空和获取日志数量：
  - 查看日志：在日志链表中遍历指定数量的日志节点，复杂度为 O(N)O(N) 。
  - 清空日志：释放日志链表中的所有日志节点，复杂度为 O(N)O(N) 。
  - 获取日志数量：获取日志的数量等同于获取 `server.slowlog` 链表的数量，复杂度为 O(1)O(1) 。
- Redis 用一个链表以 FIFO 的顺序保存着所有慢查询日志。
- 每条慢查询日志以一个慢查询节点表示，节点中记录着执行超时的命令、命令的参数、命令执行时的时间，以及执行命令所消耗的时间等信息。

## REDIS 数据库

因为数据库本身是一个字典， 所以对数据库的操作基本上都是对字典的操作， 加上以下一些维护操作：

- 更新键的命中率和不命中率，这个值可以用 [INFO](http://redis.readthedocs.org/en/latest/server/info.html#info) 命令查看；

- 更新键的 LRU 时间，这个值可以用 [OBJECT](http://redis.readthedocs.org/en/latest/key/object.html#object) 命令来查看；

- 删除过期键（稍后会详细说明）；

- 如果键被修改了的话，那么将键设为脏（用于事务监视），并将服务器设为脏（等待 RDB 保存）；

- 将对键的修改发送到 AOF 文件和附属节点，保持数据库状态的一致；

  

  在数据库中， 所有键的过期时间都被保存在 `redisDb` 结构的 `expires` 字典里：

  通过 `expires` 字典， 可以用以下步骤检查某个键是否过期：

  1. 检查键是否存在于 `expires` 字典：如果存在，那么取出键的过期时间；
  2. 检查当前 UNIX 时间戳是否大于键的过期时间：如果是的话，那么键已经过期；否则，键未过期。

  

  我们知道了过期时间保存在 `expires` 字典里， 又知道了该如何判定一个键是否过期， 现在剩下的问题是， 如果一个键是过期的， 那它什么时候会被删除？

  1. 定时删除：在设置键的过期时间时，创建一个定时事件，当过期时间到达时，由事件处理器自动执行键的删除操作。对内存是最友好的,对 CPU 时间是最不友好的,  目前 Redis 事件处理器对时间事件的实现方式 —— 无序链表， 查找一个时间复杂度为 O(N)O(N) —— 并不适合用来处理大量时间事件。
  2. 惰性删除：放任键过期不管，但是在每次从 `dict` 字典中取出键值时，要检查键是否过期，如果过期的话，就删除它，并返回空；如果没过期，就返回键值。对 CPU 时间来说是最友好的.
  3. 定期删除：每隔一段时间，对 `expires` 字典进行检查，删除里面的过期键.定期删除是这两种策略的一种折中：
     - 它每隔一段时间执行一次删除操作，并通过限制删除操作执行的时长和频率，籍此来减少删除操作对 CPU 时间的影响。
     - 另一方面，通过定期删除过期键，它有效地减少了因惰性删除而带来的内存浪费。

Redis 使用的过期键删除策略是**惰性删除加上定期删除**， 这两个策略相互配合，可以很好地在合理利用 CPU 时间和节约内存空间之间取得平衡。



## redis TTL实现原理

### TTL存储的数据结构

 redis针对TTL时间有专门的dict进行存储，就是redisDb当中的dict *expires字段，dict顾名思义就是一个hashtable，key为对应的rediskey，value为对应的TTL时间。
链接：https://www.jianshu.com/p/53083f5f2ddc
https://juejin.im/post/6844903965927145479

![image-20200804213323189](/Users/ano/Desktop/notes/image/image-20200804213323189.png)

>  图中过期字段和键空间中键对象有重复，实际中不会出现重复对象，键空间的键和过期字典的键都指向同一个键对象

## 过期键会被保存在更新后的 RDB 文件、 AOF 文件或者重写后的 AOF 文件里面吗？

### 更新后的 RDB 文件

在创建新的 RDB 文件时，程序会对键进行检查，过期的键不会被写入到更新后的 RDB 文件中。因此，过期键对更新后的 RDB 文件没有影响。

### AOF 文件

在键已经过期，但是还没有被惰性删除或者定期删除之前，这个键不会产生任何影响，AOF 文件也不会因为这个键而被修改。当过期键被惰性删除、或者定期删除之后，程序会向 AOF 文件追加一条 `DEL` 命令，来显式地记录该键已被删除。

### AOF 重写

和 RDB 文件类似， 当进行 AOF 重写时， 程序会对键进行检查， 过期的键不会被保存到重写后的 AOF 文件。因此，过期键对重写后的 AOF 文件没有影响。

### 复制

当服务器带有附属节点时， 过期键的删除由主节点统一控制：

- 如果服务器是主节点，那么它在删除一个过期键之后，会显式地向所有附属节点发送一个 `DEL` 命令。
- 如果服务器是附属节点，那么当它碰到一个过期键的时候，它会向程序返回键已过期的回复，但并不真正的删除过期键。因为程序只根据键是否已经过期、而不是键是否已经被删除来决定执行流程，所以这种处理并不影响命令的正确执行结果。当接到从主节点发来的 `DEL` 命令之后，附属节点才会真正的将过期键删除掉。

附属节点不自主对键进行删除是为了和主节点的数据保持绝对一致， 因为这个原因， 当一个过期键还存在于主节点时，这个键在所有附属节点的副本也不会被删除。这种处理机制对那些使用大量附属节点，并且带有大量过期键的应用来说，可能会造成一部分内存不能立即被释放，但是，因为过期键通常很快会被主节点发现并删除，所以这实际上也算不上什么大问题。

- 数据库主要由 `dict` 和 `expires` 两个字典构成，其中 `dict` 保存键值对，而 `expires` 则保存键的过期时间。
- 数据库的键总是一个字符串对象，而值可以是任意一种 Redis 数据类型，包括字符串、哈希、集合、列表和有序集。
- `expires` 的某个键和 `dict` 的某个键共同指向同一个字符串对象，而 `expires` 键的值则是该键以毫秒计算的 UNIX 过期时间戳。
- Redis 使用惰性删除和定期删除两种策略来删除过期的键。
- 更新后的 RDB 文件和重写后的 AOF 文件都不会保留已经过期的键。
- 当一个过期键被删除之后，程序会追加一条新的 `DEL` 命令到现有 AOF 文件末尾。
- 当主节点删除一个过期键之后，它会显式地发送一条 `DEL` 命令到所有附属节点。
- 附属节点即使发现过期键，也不会自作主张地删除它，而是等待主节点发来 `DEL` 命令，这样可以保证主节点和附属节点的数据总是一致的。
- 数据库的 `dict` 字典和 `expires` 字典的扩展策略和普通字典一样。它们的收缩策略是：当节点的填充百分比不足 10% 时，将可用节点数量减少至大于等于当前已用节点数量。



## Redis 分别提供了哪些持久化机制

- RDB 将数据库的快照（snapshot）以二进制的方式保存到磁盘中。
- AOF 则以协议文本的方式，将所有对数据库进行过写入的命令（及其参数）记录到 AOF 文件，以此达到记录数据库状态的目的。

## RDB

- `rdbSave` 会将数据库数据保存到 RDB 文件，并在保存完成之前阻塞调用者。

- [SAVE](http://redis.readthedocs.org/en/latest/server/save.html#save) 命令直接调用 `rdbSave` ，阻塞 Redis 主进程； [BGSAVE](http://redis.readthedocs.org/en/latest/server/bgsave.html#bgsave) 用子进程调用 `rdbSave` ，主进程仍可继续处理命令请求。

- [SAVE](http://redis.readthedocs.org/en/latest/server/save.html#save) 执行期间， AOF 写入可以在后台线程进行， [BGREWRITEAOF](http://redis.readthedocs.org/en/latest/server/bgrewriteaof.html#bgrewriteaof) 可以在子进程进行，所以这三种操作可以同时进行。

- 为了避免产生竞争条件， [BGSAVE](http://redis.readthedocs.org/en/latest/server/bgsave.html#bgsave) 执行时， [SAVE](http://redis.readthedocs.org/en/latest/server/save.html#save) 命令不能执行。

- 为了避免性能问题， [BGSAVE](http://redis.readthedocs.org/en/latest/server/bgsave.html#bgsave) 和 [BGREWRITEAOF](http://redis.readthedocs.org/en/latest/server/bgrewriteaof.html#bgrewriteaof) 不能同时执行。

- 调用 `rdbLoad` 函数载入 RDB 文件时，不能进行任何和数据库相关的操作，不过订阅与发布方面的命令可以正常执行，因为它们和数据库不相关联。

- RDB 文件的组织方式如下：

  ```
  +-------+-------------+-----------+-----------------+-----+-----------+
  | REDIS | RDB-VERSION | SELECT-DB | KEY-VALUE-PAIRS | EOF | CHECK-SUM |
  +-------+-------------+-----------+-----------------+-----+-----------+
  
                        |<-------- DB-DATA ---------->|
  ```

- 键值对在 RDB 文件中的组织方式如下：

  ```
  +----------------------+---------------+-----+-------+
  | OPTIONAL-EXPIRE-TIME | TYPE-OF-VALUE | KEY | VALUE |
  +----------------------+---------------+-----+-------+
  ```

  RDB 文件使用不同的格式来保存不同类型的值。

  

## AOF



 AOF 文件的整个过程可以分为三个阶段：

1. 命令传播：Redis 将执行完的命令、命令的参数、命令的参数个数等信息发送到 AOF 程序中。
2. 缓存追加：AOF 程序根据接收到的命令数据，将命令转换为网络通讯协议的格式，然后将协议内容追加到服务器的 AOF 缓存中。
3. 文件写入和保存：AOF 缓存中的内容被写入到 AOF 文件末尾，如果设定的 AOF 保存条件被满足的话， `fsync` 函数或者 `fdatasync`函数会被调用，将写入的内容真正地保存到磁盘中。

Redis 目前支持三种 AOF 保存模式，它们分别是：

1. `AOF_FSYNC_NO` ：不保存，

   在这种模式下， SAVE 只会在以下任意一种情况中被执行：

   - Redis 被关闭
   - AOF 功能被关闭
   - 系统的写缓存被刷新（可能是缓存已经被写满，或者定期保存操作被执行）

2. `AOF_FSYNC_EVERYSEC` ：每一秒钟保存一次。SAVE 原则上每隔一秒钟就会执行一次， 因为 SAVE 操作是由后台子线程调用的， 所以它不会引起服务器主进程阻塞

3. `AOF_FSYNC_ALWAYS` ：每执行一个命令保存一次。

对于三种 AOF 保存模式， 它们对服务器主进程的阻塞情况如下：

1. 不保存（`AOF_FSYNC_NO`）：写入和保存都由主进程执行，两个操作都会阻塞主进程。
2. 每一秒钟保存一次（`AOF_FSYNC_EVERYSEC`）：写入操作由主进程执行，阻塞主进程。保存操作由子线程执行，不直接阻塞主进程，但保存操作完成的快慢会影响写入操作的阻塞时长。
3. 每执行一个命令保存一次（`AOF_FSYNC_ALWAYS`）：和模式 1 一样。安全性是最高的， 但性能也是最差的

- AOF 文件通过保存所有修改数据库的命令来记录数据库的状态。
- AOF 文件中的所有命令都以 Redis 通讯协议的格式保存。
- 不同的 AOF 保存模式对数据的安全性、以及 Redis 的性能有很大的影响。
- AOF 重写的目的是用更小的体积来保存数据库状态，整个重写过程基本上不影响 Redis 主进程处理命令请求。
- AOF 重写是一个有歧义的名字，实际的重写工作是针对数据库的当前值来进行的，程序既不读写、也不使用原有的 AOF 文件。
- AOF 可以由用户手动触发，也可以由服务器自动触发。

REDIS Server

- 服务器经过初始化之后，才能开始接受命令。
- 服务器初始化可以分为六个步骤：
  1. 初始化服务器全局状态。
  2. 载入配置文件。
  3. 创建 daemon 进程。
  4. 初始化服务器功能模块。
  5. 载入数据。
  6. 开始事件循环。
- 服务器为每个已连接的客户端维持一个客户端结构，这个结构保存了这个客户端的所有状态信息。
- 客户端向服务器发送命令，服务器接受命令然后将命令传给命令执行器，执行器执行给定命令的实现函数，执行完成之后，将结果保存在缓存，最后回传给客户端。

## REDIS Hyperlog log



链接：https://juejin.im/post/5d066d11e51d4510a50335bc

场景： 统计一个集合中不重复的元素个数，比如日常需求场景有，统计页面的UV或者统计在线的用户数、注册IP数。。

简单的做法就是记录集合中的所有不重复的 集合S，新来一个元素x，首先判断x在不在S中，如果不在，则将x加入到S，否则不记录。常用的SET数据结构就可以实现。

但是这样实现，如果数据量越来越大，会造成什么问题？

- 当统计的数据量变大时，相应的存储内存会线性增长。
- 当集合S越大时，判断x元素是否在集合S中的所花的成本会越大。



常用的基数计数有三种： B+树、bitmap、概率算法。

- B+ 树。 B+ 树插入和查找效率比较高。可以快速查找元素是否存在，以及进行插入。如果要计算基数值（不重复的元素值），则只需要树的节点个数即可。但是依然存在没有节省内存空间的问题。
- bitmap。 bitmap 是通过一个bit数组来存在特定数据的一种数据结构。基数计数则将每一个元素对应到bit数组的其中一位，比如Bit数组010010101，代表[1,4,6,8]。新加入的元素只需要已有的Bit数组和新加入的元素进行按位或计算。这种方式可以大大减少内存，如果存储1亿数据的话，大概只需要 100000000/8/1024/1024 ≈ 12M 的内存。 相比B+树确实节省不少，但是在某些非常大数据的场景下，如果有10000个对象有1亿数据，则需要120G内存，可以说在特定场景下内存的消耗还是蛮大的。
- 概率算法，概率算法是通过牺牲准确率来换取空间，对于不要求绝对准确率的场景下，概率算法是一种不错的选择，因为概率算法不直接存储数据集合本身，通过一定的概率统计方法预估基数值，同时保证误差在一定范围内，这种方式可以大大减少内存。HyperLogLog就是概率算法的一种实现，下面重点介绍一下此算法。

HyperLogLog 原理思路是通过给定 n 个的元素集合，记录集合中数字的比特串第一个1出现位置的最大值k，也可以理解为统计二进制低位连续为零的最大个数。通过k值可以估算集合中不重复元素的数量m，m近似等于2^k。

下图来源于网络，通过给定一定数量的用户User，通过Hash得到一串Bitstring，记录其中最大连续零位的计数为4，User的不重复个数为 2 ^ 4 = 16

![image-20200710215403750](/Users/ano/Desktop/notes/image/image-20200710215403750.png)



Redis采用了16384个桶来存储计算HyperLogLog，那所占的内存会是多少？ Redis最大可以统计2^64个数据，也就是说每个桶的最大maxbits需要 6 个bit来存储(2^6=64)。那么所占内存就是 16384 * 6 / 8 = 12kb。

* 目的是做基数统计，故不是集合，不会保存元数据，只记录数量而不是数值。
* 耗空间极小，支持输入非常体积的数据量
* 核心是基数估算算法，主要表现为计算时内存的使用和数据合并的处理。最终数值存在一定误差
* redis中每个hyperloglog key占用了12K的内存用于标记基数（官方文档）
* pfadd命令并不会一次性分配12k内存，而是随着基数的增加而逐渐增加内存分配；而pfmerge操作则会将sourcekey
* 合并后存储在12k大小的key中，这由hyperloglog合并操作的原理（两个hyperloglog合并时需要单独比较每个桶的值）可以很容易理解。
* 误差说明：基数估计的结果是一个带有 0.81% 标准错误（standard error）的近似值。是可接受的范围
* Redis 对 HyperLogLog 的存储进行了优化，在计数比较小时，它的存储空间采用稀疏矩阵存储，空间占用很小，仅仅在计数慢慢变大，稀疏矩阵占用空间渐渐超过了阈值时才会一次性转变成稠密矩阵，才会占用 12k 的空间



## Sorted set(Zset)

![image-20200727195050035](/Users/ano/Desktop/notes/image/image-20200727195050035.png)

# 