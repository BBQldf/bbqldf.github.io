---
layout:     post
title:     Java Interview-04：Spring与分布式
subtitle:   每日小问——Spring系列
date:       2022-03-08
author:     ldf
header-img: img/post-bg-interview01.jpg
catalog: true
tags:
    - java基础
    - 面试
    - code
---

# 一、Spring

什么是Bean? Spring Bean是**被实例的,组装的及被Spring 容器管理的Java对象**。 Spring 容器会自动完成@bean对象的实例化。

什么是反射？对于任意一个对象，都能够调用它的任意方法和属性；这种**动态获取信息以及动态调用对象方法**的功能称为java语言的反射机制。

## 1、Spring中bean的四种属性注入方式

- **setter方法注入**

在xml文件中，使用set注入的方式就是通过property标签，如下所示：

```java
<!-- 定义car这个bean，id为myCar -->
<bean id="myCar" class="cn.tewuyiang.pojo.Car">
    <!-- 
        为car的属性注入值，因为speed和price都是基本数据类型，所以使用value为属性设置值；
        注意，这里的name为speed和price，不是因为属性名就是speed和price，
        而是set方法分别为setSpeed和setPrice，名称是通过将set删除，然后将第一个字母变小写得出；
    -->
    <property name="speed" value="100"/>
    <property name="price" value="99999.9"/>
</bean>
```

​    // 获取user这个bean `User user = context.getBean(User.class);`

- **构造器注入**

通过constructor-arg标签为构造器传入参数值

```java
<bean id="myCar" class="cn.tewuyiang.pojo.Car">
    <!-- 通过constructor-arg的name属性，指定构造器参数的名称，为参数赋值 -->
    <constructor-arg name="speed" value="100" />
    <constructor-arg name="price" value="99999.9"/>
</bean>
```

property的name属性，是通过set方法的名称得来；而constructor-arg的name，则是构造器参数的名称。

- **静态工厂注入**

静态工厂注入就是我们编写一个静态的工厂方法，这个工厂方法会返回一个我们需要的值，然后在配置文件中，我们指定使用这个工厂方法创建bean。

先定义一个静态工厂：

```java
public class SimpleFactory {

    /**
     * 静态工厂，返回一个Car的实例对象
     */
    public static Car getCar() {
        return new Car(12345, 5.4321);
    }
}
```

然后在xml中指定`<bean id="car" class="cn.tewuyiang.factory.SimpleFactory" factory-method="getCar"/>`

- **实例工厂注入**

实例工厂与静态工厂类似，不同的是，静态工厂调用工厂方法不需要先创建工厂类的对象，因为静态方法可以直接通过类调用，所以在上面的配置文件中，并没有声明工厂类的bean。但是，**实例工厂，需要有一个实例对象，才能调用它的工厂方法。**

三大步骤：

1. Spring容器需要先创建一个SimpleFactory对象
2. 创建三个bean，通过factory-bean指定工厂对象
3. 将上面通过实例工厂方法创建的bean，注入到user中

```java
<!-- 声明实例工厂bean，Spring容器需要先创建一个SimpleFactory对象，才能调用工厂方法 -->
<bean id="factory" class="cn.tewuyiang.factory.SimpleFactory" />

<!-- 
    通过实例工厂的工厂方法，创建三个bean，通过factory-bean指定工厂对象，
    通过factory-method指定需要调用的工厂方法
-->
<bean id="name" factory-bean="factory" factory-method="getName" />
<bean id="age" factory-bean="factory" factory-method="getAge" />
<bean id="car" factory-bean="factory" factory-method="getCar" />

<bean id="user" class="cn.tewuyiang.pojo.User">
    <!-- 将上面通过实例工厂方法创建的bean，注入到user中 -->
    <property name="name" ref="name"/>
    <property name="age" ref="age"/>
    <property name="car" ref="car"/>
</bean>
```

## 2、IoC控制反转

把对象创建和对象间的调用过程交给Spring进行管理，**通过反射实现**。

那什么是反射？

1. 在程序运行期间去动态创建对象；即：把要创建的对象放到配置文件中，让Java去读取配置文件。这样我就不用每次要修改的时候，要修改源代码，而是直接修改配置文件（比如：你要创建一个Dog对象，你要new dog();再创建一个Cat对象，你要new cat();然后往里面注入属性。这不好，你可以直接在配置文件中修改`bean=com.ref.Cat`，然后直接加载bean就行！实现了解耦合！
2. 反射机制配合着注解，实现了对象的赋值。
   1. 实例分析：我们给dog赋值，正常的是dog.setId(2)；这个就是正向的；现在是id.set(dog,2)，这样就是反过来的，即反射！
   2. 有一个面试题：`private属性能在外面赋值吗？`正常是不可以的，外面调用不到这个属性，但是现在可以了，我用反射，直接获取属性id，然后反向注入。（前提是开启暴力反射id.setAccessible(true))



- 以前是：`A = new A();`现在是放到SpringIoC容器中，你只要告诉spring就可以拿到。（松耦合）三种方式**创建Bean对象**（注意，这里只是创建对象，并不是上面的属性注入）：

  - xml文件配置（ClassPathXmlApplicationContext("xxx.xml")）
  - 注解实现（AnnotationConfigApplicationContext），
    - 配置类：用一个@Configuration的java类（里面的对象用@Bean包裹）代替xml文件，把在xml中配置的内容放到配置类中
    - 扫包+注解：@Component

- 实现方法：**依赖注入（DI）**(核心:@Autowired（先按类型ByType，想按名字ByName找配置@Qualifier("config")）;这种DI实现了，我不仅能创建单个对象，对象之间的关系我也可以互相调用，配合@Component)。[有三种方式](https://zhuanlan.zhihu.com/p/90939765)

  - 接口注入：

    - 采用这种注入方式，学渣只是在做作业时，才临时抱佛脚地找一下学霸。

  - Getter、Setter注入

    - 这种方式学霸和学渣只是暂时的合作关系，如果学渣赖上了另一个学霸（调用set()方法传入了另一个对象），那么学渣和学霸的合作关系就结束了。

  - 构造函数注入

    - ```java
      public class StupidStudent {
          private SmartStudent smartStudent;
          
          public StupidStudent(SmartStudent smartStudent) {
              this.smartStudent = smartStudent;
          }		//构造器
          
          public doHomewrok() {
              smartStudent.doHomework();
              System.out.println("学渣抄作业");
          }
      }
      
      
      public class StudentTest {
          public static void main(String[] args) {
              SmartStudent smartStudent = new SmartStudent();
              StupidStudent stupidStudent = new StupidStudent(smartStudent);	//调用构造器
              stupidStudent.doHomework();
          }
      }
      ```

    - 这种方式好比学渣从一开始就赖上了一个学霸，并且和这个学霸建立了长期合作关系。



## 3、AoP切面编程

在运行时，**动态地将代码切入到类的指定方法、指定位置上**的编程思想就是面向切面的编程。比如，有一个计时器，统计一段程序的运行时间；如果程序多了，我就要每次都加上一个计时器，我就把这个计时器拿出来，然后通过AoP去动态织入进去。

AoP有五种织入方式：

- Before前置增强
- After后置增强
- Around环绕增强
- AfterReturning最终增强
- AfterThrowing异常增强

实现方式：代理

### 3.1 代理

1. 静态代理：需要为每一个实体类创建一个代理类，代理类可以扩展出更多的功能。（但是很麻烦，不容易扩展）
2. 动态代理：基于JDK（代理需要实现一个InvocationHandler接口，本体也要实现接口）和cglib（默认用，不需要实现接口）
   - 动态：强调代理是在运行的时候才生成代理类，而不是在编译的时候（静态）

## 4、Bean的生命周期

它的整体过程是：普通的Java类——>beanDefinition对象——>Spring Bean对象。

网上很多教程讲的很类似，一般就是实例化、赋值、初始化、销毁四个阶段。**往往忽视了实例化这一步**~因为这一步，还会对后面的循环依赖有效果！

以注解类变成Spring Bean为例，Spring会扫描指定包下面的Java类，然后将其变成beanDefinition对象，然后**Spring会根据beanDefinition来创建bean**。

### 4.1 BeanDefinition对象

![img](https://img-blog.csdnimg.cn/img_convert/f22ca0ff96d74a6a3cbf4a10787cacf7.png)

BeanDefinition 是定义 Bean 的配置元信息接口，他有很多方法，主要作用，其实就是预先设置一些bean的属性：

- 设置父 bean 名称、是否为 primary
- Bean的作用域（setScope）
- Bean的延迟加载（setLazyInit）
- Bean之间的依赖设置（setDependsOn）@DependsOn
- 一些生命周期的回调：setFactoryMethodName
- 初始方法（setInitMethodName）、销毁（setDestoryMethodName）方法等

可以使用 BeanDefinitionBuilder 或 new BeanDefinition 实现类构建 BeanDefinition 对象。

所以，beanDefinition只是一个建模的过程，他存储着我们日常给Spring Bean定义的元数据（@Scope、@Lazy、@DependsOn等等）

PS：**为什么不直接使用对象的class对象来创建bean呢？**

- 因为在class对象仅仅能描述一个对象的创建（只描述了类的信息），它不足以用来描述一个Spring bean，而对于是否为懒加载、是否是首要的、初始化方法是哪个、销毁方法是哪个，这个Spring中特有的属性在class对象中并没有，所有Spring就定义了beanDefinition来完成bean的创建。

### 4.2 流程

会有**8个后置处理的方法**、**4个后置处理器的类**贯穿在对象的：实例化（1）、赋值（2）、初始化（3-8）、销毁（9）

1. Spring对bean进行实例化，默认bean是单例。
   - Spring容器启动，然后扫描（如：xml配置，@Component注解，JavaConfig），把普通的java对象变成BeanDefinition，并且存到一个**BeanDefinition Map中**（我记得这个Map的key应该是beanName，value则是BeanDefinition对象）——到这里，都还没有实例化！！
   - 然后对这个BeanDefinition Map做一个遍历，并且验证一些基本属性：Bean的名字？是否单例？是否原型？是否懒加载？是否primary？是否有DependsOn（这个是后面解决循环依赖的）？是否FactoryBean？——这一步会执行BeanFactoryPostProcessor这个Bean工厂后置处理器的逻辑（可以对Bean原信息进行修改）
   - 然后，Spring中通过**反射**选择合适的构造器来把对象实例化
   - 去判断这个实例化的类，有没有在单例池中？有没有被提前暴露？
     - 如果没有提前暴露，才会去创建Bean
2. Spring对bean进行依赖注入（这一步很简单）
   - Spring根据BeanDefinition中的信息进行依赖注入。
3. 检查Aware相关接口并设置相关依赖（**本质上就是用于对SpringBean的扩展**）
   - 如果bean实现了BeanNameAware接口，spring将bean的id传给setBeanName()方法；——写名字
   - 实现了BeanFactoryAware接口，调用setBeanFactory方法，将BeanFactory实例传进来；——创建实例！
   - 实现了ApplicationContextAware接口，调用setApplicationContext方法，将应用上下文的引用传进来；
   - **场景**：比如我希望通过代码程序的方式去获取指定的Spring Bean，可以用一个类，去实现ApplicationContextAware接口，来获取ApplicationContext对象进而获取Spring Bean。
4. 5-7又会用到**BeanPostProcessor后置处理器**（我在后面进一步讲解这个类，**它是AOP的关键**），它有两个方法：一个是before，一个是after
5. BeanPostProcessor前置处理，它的postProcessBeforeInitialization方法将被调用；
6. 执行init相关的方法，执行顺序：检查是否有@PostConstruct、是否实现了InitializingBean接口、是否定义了init-method方法。（这些都是Spring给我们的**“扩展”**，比如说：对象实例化后，我要做些初始化的相关工作或者就启个线程去Kafka拉取数据）
   - @PostConstruct：在指定方法上加上@PostConstruct指定该方法是在初始化之后调用。`@PostConstruct     public void postConstruct()`
   - InitializingBean接口：（调用afterPropertiesSet方法）通过实现 InitializingBean接口来定制初始化之后的操作方法。`@Override public void afterPropertiesSet()`
   - init-method方法：通过bean元素的 init-method指定初始化之后的操作方法`@Bean(initMethod = "initMethod", name = "initSequenceBean")`
   - 区别：
7. BeanPostProcessor后置处理，它的postProcessAfterInitialization方法将被调用；
8. Spring容器发布事件，把Bean放入单例池。**此时，bean已经准备就绪**，可以被应用程序使用了，他们将一直驻留在应用上下文中。
9. 直到该应用上下文被销毁；销毁时，也有三个区别：DisposableBean、destroyMethod和@PreDestroy（**可以类比init阶段的三个函数**）
   - 若bean实现了DisposableBean接口，spring将调用它的distroy()接口方法。`@Override public void destroy() throws Exception`
   - 同样的，@Bean注入对象时，通过`@Bean(destroyMethod= “destroyMethod”)`的方式，使当前bean中的destroyMethod方法在bean销毁之前被调用
   - 直接在类中的某个方法上加上该注解，使该方法在bean销毁前被调用`@PreDestory     public void PreDestory()`

注意：<font color='red'>如果bean的scope设为prototype时，当容器关闭时，destroy方法不会被调用。</font>对于prototype作用域的bean，有一点非常重要，那就是Spring不能对一个prototype bean的整个生命周期负责：**容器在初始化、配置、装饰或者是装配完一个prototype实例后，将它交给客户端，随后就对该prototype实例不闻不问了**。（让Spring容器释放被prototype作用域bean占用资源的一种可行方式是，通过使用bean的后置处理器，该处理器持有要被清除的bean的引用）

![BeanPostProcessor后置处理器](https://img-blog.csdnimg.cn/20181216171805266.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4ODIyOTMz,size_16,color_FFFFFF,t_70)

不同实例之间的差别：

- 多实例模式：bean的创建、初始化在bean被使用的时候，bean的销毁由JVM的垃圾回收器进行处理。
- 单实例懒加载的bean（注意:懒加载一般指的是单实例bean）：创建、初始化在bean第一次被使用的时候，之后便和单实例bean一样了，销毁在IOC容器关闭的时候。
- 单实例bean：bean的创建、初始化在项目启动时，销毁在IOC容器关闭时。
  



## 5、Bean的作用域

五种作用域（2个容器对象+3个web对象）分别是：

1. Singleton：默认就是这个，Spring IoC容器中只存在这么一个共享的Bean实例。每次获取到的对象都是同一个对象。**它是在容器创建的时候就同时创建了**；也可以设置lazy-init，延迟加载。
2. Prototype：表示一个bean定义对应多个对象实例。Prototype作用域的bean会导致**在每次对该bean请求**（将其注入到另一个bean中，或者以程序的方式调用容器的getBean()方法）时都会创建一个新的bean实例。容器创建的时候还不会创建bean，只有到调用bean（比如getBean()方法）才会创建。
3. Request：表示在一次HTTP请求中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。
4. Session：表示在一个HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。
5. Global Session：表示在一个全局的HTTP Session中，一个bean定义对应一个实例。global session作用域类似于标准的HTTP Session作用域，不过仅仅在基于portlet的web应用中才有意义。Portlet规范定义了全局Session的概念，它被所有构成某个portlet web应用的各种不同的portlet所共享。

## 6、设计原则（7大）+ 设计模式（9大）

> 这些都是思想类的。最基础的就是避免业务调整的时候，需要不断地修改源代码（实战开发的大忌），并且避免代码不宜堵不易读（大量的if-else逻辑），增强代码可维护性（也就是“最佳实现”的思想）——比如查表法代替
>
> 参考：
>
> 1. [B站视频-java设计模式](https://www.bilibili.com/video/BV1rU4y1f7uy?spm_id_from=333.880.my_history.page.click)
> 2. [git地址](https://github.com/fuzhengwei/CodeDesignTutorials)

思考一下：为什么Spring会安排这么多的设计模式？

- 为了代码可重用性、增加可维护性，让代码更容易被他人理解、保证代码可靠性。设计模式使代码编写真正工程化。**一方面便于后期扩展，二方面便于研发。**

- （但是这个一定要善用，用得不好，或者强行用，看上去代码简化了，但是在真正开发的时候，代码可能会交给另外的人，这会给别人很大的困惑！易读性下降了）

### 6.1 遵循设计模式七大原则：

> 可以看到，符合这样几个原则的类，都不会太臃肿，并且每个类的功能也相对单一。并且会由某一个（抽象）接口统一了功能

- **单一职责原则**（一个类只有一个职责）

- **开闭原则**（对扩展开放，但是对修改关闭）；
  - 也就是新增的功能可以新增一个类就可以了，并且这个类可以直接去（实现一些接口）/（继承一些实现）即可，并不需要在原来的接口（或者类）中进行代码修改
  - 比如：扩展类可以是计算一些高精度的场景需要，基础类是低精度（圆的面积）


- 里氏代换原则（子类对父类的覆盖或者实现）

  - 主要阐述了有关继承的一些原则，也就是什么时候应该使用继承，什么时候不应该使用继承
  - 这里和开闭原则很类似。其实就是说子类中的一些方法实现，和父类要对齐；比如子类是用来描述储蓄卡，那父类可以是银行卡大类，不能父类是一个公交卡大类；那他们设计的理念就不匹配；再来一个孙子类是信用卡继承了储蓄卡，那信用卡最好去实现一些新的功能，而不是重写父类的方法
  - 如果通过重写父类的方法来完成新的功能，这样写起来虽然简单，但是整个继承体系的可复用性会比较差，特别是运用**多态比较频繁**时，程序运行出错的概率会非常大。（因为你是有一个继承链存在的）
  - 如果程序违背了里氏替换原则，则继承类的对象在基类出现的地方会出现运行错误。这时其修正方法是：取消原来的继承关系，重新设计它们之间的关系
  - 例子：最有名的是“正方形不是长方形”。（长方形类：两个属性，宽度和高度；正方形类：一个属性，边）例如，企鹅、鸵鸟和几维鸟从生物学的角度来划分，它们属于鸟类；但从类的继承关系来看，由于它们不能继承“鸟”会飞的功能，所以它们不能定义成“鸟”的子类。

- **依赖倒转原则**（对抽象接口编程，而不是对实现进行编程）；也就是不要一直继承

  - 在程序中，就是高层模块不应该依赖底层模块，两者其实都应该依赖一个抽象模块，因为当下层剧烈变动时上层也要跟着变动。

  - 并且项目开发中一定要注意系统调用层级，dao层不能调用service层，循环调用的问题

  - 其实这里和接口隔离也有关系，模块之间相互耦合本质上还是功能没有抽象好。

  - **最经典的例子**：司机开车是一个类；奔驰车又是一个类；司机要开奔驰，那它就在在代码中注入奔驰车的对象；如果又来了一辆宝马车，司机想开宝马车的话，他还要在写一个方法，注入宝马车对象；这不合理！并且要是司机换人了呢？所以，[我们可以把司机抽象为一个接口，车抽象为一个接口，然后不同的人，不同的车型去实现这个接口；](https://blog.csdn.net/u014590757/article/details/80078302)

  - ```java
    Driver laozhang = new Driver("laozhang");
    Benz benz = new Benz();
    BMW bmw = new BMW();
    
    zhangsan.drive(benz);
    zhangsan.drive(bmw);
    ```

    

  - 例如：**Spring SPI机制**（JDK内置的一种服务提供发现机制。）；**原理就是**ClassPath路径下的META-INF/services文件夹中， 以接口的全限定名来命名文件名，文件里面写该接口的实现。然后再资源加载的方式，

    - **好处：**读取文件的内容(接口实现的全限定名)， 然后再去加载类。使得第三方服务模块的装配控制的逻辑与调用者的业务代码分离，而不是耦合在一起。
    - **坏处：**虽然ServiceLoader也算是使用的延迟加载，但是基本只能通过遍历全部获取，也就是接口的实现类全部加载并实例化一遍。如果你并不想用某些实现类，它也被加载并实例化了，这就造成了浪费。


- 接口隔离原则（一个接口最好也只有少量的方法）

  - 单一职责要求的是类和接口职责单一，注重的是职责，这是业务逻辑上的划分。（不要随便实现接口）
  - 而接口隔离原则要求接口的方法尽量少（只暴露给调用的类它需要的方法）
  - 举例：一个数据运算的接口，他下面有4个基础方法加减乘除，还有2个高阶方法积分求导；如果一个小学生要实现这个接口，他就很不方便，还要实现2个高阶方法；所以最好的方法是把这两类方法分成两个接口

- 迪米特法则**，又称最少知道原则**

  - 只关心public方法，对其他对象尽可能少了解？因为其他都是私有类，外部不需要关注；也可以理解为私有类是我自己写的东西，别人不要管我怎么操作

  - 其实本质上，就是一些类不要去依赖别人，减少类之间的耦合；成为一个独立的模块

  - 举个例子：一个教师会对外提供一个班级的信息（学生成绩，转班的情况，家庭情况）；校长为了保证对信息的掌握原则，他也要获得这些信息，他不能自己去弄这些信息（直接和班级信息表交互），只能通过查询老师提供的接口去完成。**相当于用老师把数据进行了封装**

- ~~合成/聚合复用原则。（要尽量在一个新的对象中引入（注入）已有的对象，尽量不要使用继承）——继承还是耦合性有点高~，对象注入的话，一般是通过接口和抽象类，这种更稳定一些。~~

  

### 6.2   九大  设计模式

> 简单功能实现就是三步：编译属性，创建方法，调用展示。但是当功能复杂，功能迭代之后，开发就变得困难了。
>
> 

#### 6.2.1 简单工厂：

- 实现方式：BeanFactory。 Spring中的BeanFactory就是简单工厂模式的体现，根据传入一个唯一的标识来获得Bean对象，但是否是在传入参数后创建还是传入参数前创建这个要根据具体情况来定。
- 它由三种角色组成：
  - 工厂类角色：这是本模式的核心，含有一定的商业逻辑和判断逻辑，根据逻辑不同，产生具体的工厂产品。如例子中的Driver类。
  - 抽象产品角色：它一般是具体产品继承的父类或者实现的接口。由接口或者抽象类来实现。如例中的Car接口。
  - 具体产品角色：工厂类所创建的对象就是此角色的实例。在java中由一个具体类实现，如例子中的Benz、Bmw类。
- 当暴发户增加了一辆车的时候，只要符合抽象产品制定的合同，那么只要通知工厂类知道就可以被客户使用了。（即创建一个新的车类，继承抽象产品Car）那么对于产品部分来说，它是符合开闭原则的——对扩展开放、对修改关闭；
- 缺点：但是工厂类不太理想，因为**每增加一辆车，都要在工厂类中增加相应的商业逻辑和判断逻辑，**这显自然是违背开闭原则的。

#### 6.2.2 [工厂方法:](https://www.jianshu.com/p/696faca3a7e5)

- 在该模式中，**工厂父类**负责提供创建产品对象的公共接口，而**工厂子类**负责生成具体的产品对象，换而言之，调用方只需要知道产品的类名或者某个标识就可以了，不需要知道产品对象的详细创建过程，将具体类的实例化操作延迟到工厂子类中完成，降低模块之间的耦合性。（就是对原来的简单工厂再进行一次封装）
- **当您希望通过重用现有对象而不是每次重新构建它们来节省系统资源时，请使用工厂方法。**
- 创建流程：
  - 先创建Shape和Color工厂（接口）——可以说是具体工厂角色
  - 每个工厂都创建一个实现类Red.java和Circle.java——具体的产品
  - 再创建工厂创造器FactoryProducer类——这个是工厂类本身的角色

```java
public class FactoryProducer  {
    public static AbstractFactoryDemo getFactory(String choice) {
        if (choice.equalsIgnoreCase("SHAPE")) return new ShapeFactory();
        if (choice.equalsIgnoreCase("COLOR")) return new ColorFactory();
        return null;
    }
}
```

- **工厂方法模式是对简单工厂模式进一步的解耦，在工厂方法模式中是一类产品对应一个工厂类，而这些工厂类都继承于一个抽象工厂。**这相当于是把原本会随着业务扩展而庞大的简单工厂类，拆分成了一个个的具体产品工厂类，这样代码就不会都耦合在同一个类里。
- 实现方式：FactoryBean接口。spring会在使用getBean()调用获得该bean时，会自动调用该bean的getObject()方法，所以返回的不是factory这个bean，而是这个bean.getOjbect()方法的返回值
- 他一般是和其他的方法模式结合起来的，他可以提供策略模式的一些服务，可以**提供**具体的适配器，可以提供具体的模板模式；避免通过if-else来实现
- 典型的例子有
  - spring与mybatis的结合。（数据库）
  - 日志记录器：记录可能记录到本地硬盘、系统事件、远程服务器等，用户可以选择记录日志到什么地方
  - 造冰箱的工厂，可以造不同牌子的冰箱

#### 6.2.3 单例模式:

- Spring依赖注入Bean实例默认是单例的。
- 如多个模块使用同一个数据源连接对象等等。

#### 6.2.4 适配器模式:

- 实现方式：**SpringMVC中的适配器HandlerAdatper。**
- 实现原理：HandlerAdatper根据Handler规则执行不同的Handler。

#### 6.2.5 装饰器模式:

- Spring中用到的包装器模式在类名上有两种表现：一种是类名中含有Wrapper，另一种是类名中含有Decorator。

#### 6.2.6 <font color='red'>**代理模式:**</font>

- 实现方式：**相当于 AOP底层，就是动态代理模式的实现**。
  - 静态代理：需要手工编写代理类，代理类引用被代理对象
  - 动态代理：在内存中构建的，不需要手动编写代理类
- 实现原理： 切面在应用运行的时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象创建动态的创建一个代理对象。
- 织入：把切面应用到目标对象并创建新的代理对象的过程。
- 场景：过滤器

#### 6.2.7 观察者模式:

- 实现方式：spring的**事件驱动**模型使用的是 观察者模式 。

#### 6.2.8 <font color='red'>**策略模式:**</font>

- 定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换。主要解决：**在有多种算法相似的情况下，使用 if...else 所带来的复杂和难以维护。**（一般配合工厂模式）
- 用来保证类的：开闭原则和单一原则
- 使用场景： 1、如果在一个系统里面有许多类，它们之间的区别仅在于它们的行为，那么使用策略模式可以动态地让一个对象在许多行为中选择一种行为。 2、一个系统需要动态地在几种算法中选择一种。 3、如果一个对象有很多的行为，如果不用恰当的模式，这些行为就只好使用多重的条件选择语句来实现。
- 具体实现：CrabCookingStrategy 、CrabCooking(抽象策略接口)、(具体策略)1.BraisedCrabs 2.SteamedCrabs、Kitchen(环境)

```java
public class CrabCookingStrategy {
    private Kitchen kitchen;    //厨房
    private CrabCooking qzx, hsx;    //大闸蟹加工者  
    CrabCookingStrategy() {
        System.out.println("策略模式在大闸蟹做菜中的应用");
        List arrayList= new ArrayList<>();
        //环境
        kitchen = new Kitchen();
        // 清蒸
        qzx = new SteamedCrabs();
        // 红烧
        hsx = new BraisedCrabs();

        //修改策略
        itemStateChanged(qzx);
        System.out.println(kitchen.getStrategy());
        itemStateChanged(hsx);
        System.out.println(kitchen.getStrategy());
        itemStateChanged(qzx);
        System.out.println(kitchen.getStrategy());
    }
    //这是第一种方式，通过传递类型，然后在里面进行判断，创建对应的服务
    public void itemStateChanged(Object action) {
        if (action == qzx) {
            kitchen.setStrategy(qzx);
            kitchen.cookingMethod(); //清蒸
        } else if (action == hsx) {
            kitchen.setStrategy(hsx);
            kitchen.cookingMethod(); //红烧
        }
    }
    //这是第二种方式，直接传入对应的class，然后我们来做实例化
    public ICommodity itemStateChanged2(Class<? extends ICommodity> clazz) {
        if(clazz == null){
            return null;
        }else{
            return clazz/newInstance();
        }
        /*比如可以这样调用：
        ICommodity commodity = storeFactory.getCommodity(Coupon.class);//优惠券类型
        commodity.sendCommodity(xxxx);
        */
    }
    
    public static void main(String[] args) {
        new CrabCookingStrategy();
    }
}
//抽象策略类：大闸蟹加工类
interface CrabCooking {
    public void cookingMethod();    //做菜方法
}
//具体策略类：清蒸大闸蟹
class SteamedCrabs implements CrabCooking {
    private static final long serialVersionUID = 1L;
    public void cookingMethod() {
    System.out.println("清蒸大闸蟹");
    }
}
//具体策略类：红烧大闸蟹
class BraisedCrabs implements CrabCooking {
    private static final long serialVersionUID = 1L;
    public void cookingMethod() {
        System.out.println("红烧大闸蟹");
    }
}
//环境类：厨房
class Kitchen {

    //抽象策略
    private CrabCooking strategy;
    public void setStrategy(CrabCooking strategy) {
        this.strategy = strategy;
    }
    public CrabCooking getStrategy() {
        return strategy;
    }
    public void cookingMethod() {
        System.out.println("做菜");
        strategy.cookingMethod();    //做菜
    }
}
```

- 实现代表：Spring框架的**资源访问Resource接口** 。该接口提供了更强的资源访问能力，Spring 框架本身大量使用了 **Resource 接口来访问底层资源**。

#### 6.2.8 **模版方法模式:**

- 具体实现： **JDBC的抽象和对Hibernate的集成，**都采用了一种理念或者处理方式，那就是模板方法模式与相应的Callback接口相结合。
- 为什么说JDBC是模板方法模式？因为JDBC的操作就是个模板化的操作，完全固定。
  - 加载驱动程序;
  - 获得数据库连接;
  - 操作数据库，实现增删改查, 连接模式有2种: createStatement / prepareStatement;
  - 关闭数据库连接;

#### 6.2.9 策略模式+工厂模式







### 6.3 请你写一下单例模式加载Bean的几种方法——懒汉模式、饿汉模式？

为什么一定要选单例模式呢？

- 单例模式保证了系统内存中该类只存在一个对象，节省了系统资源，对于一些需要频繁创建销毁的对象，使用单例模式可以极大地提高系统性能，例如数据库连接池，session工厂等。

我们在写代码的时候，想要实现单例模式，其实本质上就是去满足单例模式的几个属性：

1. 对于某个单例类，只能存在一个对象实例。（一个实例）
2. 这个类只提供一个取得该对象实例的静态方法。（一个静态获取方法）——别人不能通过这个类的其他方法获取到对象实例
   - 所以，所有的代码里面，加上一个**私有化构造器**,防止外部通过new生成实例

有这么几种实现方式：

> 概括一下：饿汉式就是利用类加载过程中，静态常量和静态代码块自动加载，且只加载一次实现的；
>
> 懒汉式有三大类，一是通过**锁机制**（Synchronized、volatile）；二是通过**静态内部类**，外部类加载时不需要加载静态内部类，不被加载则不占用内存，当外部类调用getInstance方法时，才加载静态内部类（所以还是延迟加载，即懒加载），静态属性保证了全局唯一，静态变量初始化保证了线程安全，所以这里的方法没有加synchronized关键字（也就是说退化成了静态常量，根据JVM的加载机制，也只会加载一次）；三是通过**枚举**（本质上是用静态字段来实现的）

1. 饿汉式（静态常量）

```java
    //1、私有化构造器,防止外部new产生实例
    private SingletonDemo1(){};
    //2、定义一个静态常量，接收Singleton内部实例；相当于已经构造好了
    private final static SingletonDemo1 instance = new SingletonDemo1();

    //定义公开的方法，给外部调用，获得实例对象
    public static SingletonDemo1 getInstance(){
        return instance;
    }
    public static void main(String[] args){
        SingletonDemo1 a1 = SingletonDemo1.getInstance();
        SingletonDemo1 a2 = SingletonDemo1.getInstance();
        System.out.println(a1==a2);
    }
```

2. 饿汉式（静态代码块）

```java
    //1、私有化构造器,防止外部new产生实例
    private SingletonDemo2(){};
    //2、定义一个静态对象，准备接收Singleton内部实例
    private static SingletonDemo2 instance;

    //3、通过静态代码块创建对象实例；也是相当于先行创建好
    static{
        instance = new SingletonDemo2();
    }

    //定义公开的方法，给外部调用，获得实例对象
    public static SingletonDemo2 getInstance(){
        return instance;
    }
    public static void main(String[] args){
        SingletonDemo2 a1 = SingletonDemo2.getInstance();
        SingletonDemo2 a2 = SingletonDemo2.getInstance();
        System.out.println(a1==a2);
    }
```

3. 懒汉式（线程不安全）——最常见的

```
    private SingletonDemo3(){
        System.out.println("SingletonDemo3 has loaded");
    };
    private static SingletonDemo3 instance;
    public static SingletonDemo3 getInstance(){
        if(instance==null){
            instance = new SingletonDemo3();
        }
        return instance;
    }

    public static void main(String[] args) {
        SingletonDemo3 a1 = SingletonDemo3.getInstance();
        SingletonDemo3 a2 = SingletonDemo3.getInstance();
        System.out.println(a1==a2);
    }
```

4. **懒汉式（线程安全，Synchronized同步方法实现）**——最简单的方式

```
    //1、私有化构造器,防止外部new产生实例
    private SingletonDemo4(){};

    //2、定义一个static实例对象接收对象实例
    private static SingletonDemo4 instance;

    //Synchronized关键字，修饰的静态方法
    public static synchronized SingletonDemo4 getInstance() {
        if(instance==null){
            instance = new SingletonDemo4();
        }
        return instance;
    }

    public static void main(String[] args) {
        SingletonDemo4 a1 = SingletonDemo4.getInstance();
        SingletonDemo4 a2 = SingletonDemo4.getInstance();
        System.out.println(a1==a2);
    }
```

5. 懒汉式（线程安全，同步代码块实现）——不够安全（非原子操作，会出现指令重排），极度不推荐——双重检查+volatile解决

6. 懒汉式（双重检查）——不够安全（比5安全一点；非原子操作，会出现指令重排），不推荐

7. 懒汉式（双重检查+volatile）——安全

```java
   /**
     双重校验+volatile锁
     */
    //     1. 私有化构造器,防止外部通过new生成实例
    private SingletonDemo7(){
        System.out.println("SingletonDemo7 has loaded");
    };

    // 2. 定义volatile实例对象,保证生成实例对象的可见性,并用于接收静态代码块生成的内部实例
    private static volatile SingletonDemo7 instance;

    //     3. 定义一个public方法提供给外部调用，获取实例对象，同时再在内部进行两次判断，第二次进行synchronized代码块加锁，一旦实例化成功，锁外的线程即可通过volatile获得实例。
    public static SingletonDemo7 getInstance() {
        if(instance==null){
            synchronized (SingletonDemo7.class){
                if(instance==null){
                    instance = new SingletonDemo7();
                }
            }
        }
        return instance;
    }
    public static void main(String[] args) {
        SingletonDemo7 a1 = SingletonDemo7.getInstance();
        SingletonDemo7 a2 = SingletonDemo7.getInstance();
        System.out.println(a1==a2);
    }
```

8. **饿汉式实现了懒汉式（静态内部类）——伪懒加载**



9. 懒汉式（枚举）——最推荐的方式

> 因为**枚举类型是线程安全的，并且只会装载一次**，设计者充分的利用了枚举的这个特性来实现单例模式，枚举的写法非常简单，而且枚举类型是所用单例实现中唯一一种不会被破坏的单例实现模式。

```
public class SingletonDemo9 {
    /**
     * 懒汉式（枚举）——最推荐的方式
     线程安全，实现简单
     上面的方案都不是适合反序列化，和反射
     防止反序列化重新创建新的对象（依次读取）
     两种解决方案：
     1. 枚举
     */
    public static void main(String[] args) {
        SingletonDemo_enum instance = SingletonDemo_enum.INSTANCE;
        instance.getInstance();
    }

}

enum SingletonDemo_enum{
    INSTANCE;       //只有一个枚举value
    public void getInstance(){
        System.out.println("SingletonDemo9 has loaded");
    }
}
```

一般到上面9种，就已经差不多了（面试官估计也就累了~）。然后下面的是扩展，把**ThreadLocal、CAS**也加入近来，形成一个完整的体系！！

```
2. 在Singleton类中添加readResolve()方法，在反序列化时被反射调用，如果定义了这个方法，就返回这个方法的值，如果没有定义，则返回新new出来的对象。
public class Singleton implements Serializable {

    //私有构造方法
    private Singleton() {}

    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    //对外提供静态方法获取该对象
    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
    
    /**
     * 下面是为了解决序列化反序列化破解单例模式
     */
    private Object readResolve() {
        return SingletonHolder.INSTANCE;
    }
}

```



10. 饿汉式实现了懒汉式（ThreadLocal）——变量副本，线程隔离；伪懒加载；

```

```

11. 饿汉式实现了懒汉式（CAS）——原子操作，compareAndSet



### 总结

- 单例模式保证了 系统内存中该类只存在一个对象，节省了系统资源，对于一些需要频繁创建销毁的对象，使用单例模式可以极大地提高系统性能，例如数据库连接池，`session`工厂等。
- 一般采用饿汉式，若对资源十分在意可以采用静态内部类，不建议采用懒汉式及双重检测











懒加载带来的危险？？？（请参考本文的第八点）



## 7、解释一下工厂模式中：简单工厂、工厂方法、抽象工厂的区别？

实际上这个设计模式有三个变种，分别是『简单工厂模式』、『工厂方法模式』以及『抽象工厂模式』，可能大部人所熟知的是前两种，抽象工厂模式有一定的场景限制，很少出现在大家的视野中！

**简单工厂模式：**

- Spring 的 BeanFactory 其实就是一个简单工厂模式，他定义了一个 BeanFactory 工厂，然后会有 DefaultListableBeanFactory 去实现这个工厂声明的所有能力。
- 简单工厂说白了就是一个超级工厂，他可以生产各种各样的产品，产品之间无关联。比如，我定一个工厂，他可以实现冰箱、空调、洗衣机方法；
- 如果要加需求，有格力冰箱、海尔冰箱、海信冰箱等等，他们生产的冰箱或空调都不一样，如果用简单工厂的话，就需要做区分，增加更多的方法，生产格力冰箱的，生产海尔冰箱的，非常的丑陋。

**工厂方法模式（FactoryBean）：**

- 其实理论上来说，可以把简单工厂模式理解为工厂方法模式的一种特例，将他的那个超级大工厂拆分成多个工厂就是工厂方法模式了。
- 工厂方法模式，需要区分不同的工厂，这里我们创建格力工厂、海尔工厂和海信工厂。
- 格力工厂返回的是格力的空调、格力的冰箱以及格力的电视机，海尔和海信也都会返回他们自己品牌的产品，这里就不贴他们的代码了。
- 工厂方法对外提供了生产产品的能力，具体产生何种类型的产品，将由具体的工厂决定。

**抽象工厂模式（AbstractFactory）：**——有点像工厂之上又加了一层[工厂](https://www.runoob.com/design-pattern/abstract-factory-pattern.html)——所以调用的时候，先使用工厂生成器FactoryProducer（传入具体要获取的工厂名字）获取AbstractFactory对象，然后进一步（向具体的实现方法中）传入类型信息来获取实体类（Bean）的对象。

- 设想一下，你使用了工厂方法模式，你的工厂提供的能力非常多，可以生产冰箱、电视、空调、洗衣机、电脑以及桌子等等，这样你就会产生很多的工厂。
- 抽象工厂的作用就是在一定前提下，帮你分类这些工厂，比如按品牌分类，或者按照价格等级分类，这样会大大缩减系统中的工厂数量。前提就是你的这些工厂需要在两个维度上具备共性：
  - 产品等级：比如都可以按照类型区分成三类，电视机、冰箱和空调。
  - 产品族：按照品牌区分成海尔、海信和 TCL。

## 8、懒加载带来的危险？

- 懒汉很懒，只有在系统用到某个类的实例的时候，才会实例化出一个唯一实例。
- 饿汉方式实现的单例模式是极其简单的，但缺点也很明显，即便这个类一时半会不会被使用到，但也必须在编译的时候初始化分配堆内存，创建这个内部实例。

**问题：**多线程环境下，线程 A 和线程 B 同时判断 instance==null，都去实例化 instance，导致 instance 被实例化两次，堆中产生一个无引用对象，并发量大的情况下，会有更多的无用对象被创建，甚至可能提前触发 GC。（对象不唯一，且堆中创建了大量的无效对象）

- 懒汉方式优化一（加本地锁）：给 instance 加 volatile 修饰是为了防止 jvm 指令重排序，通过再次判断可以保证此实例的**唯一实例化。**
- 懒汉方式优化二（枚举类）:<font color='red'>使用枚举类实现懒汉单例模式是最佳实践！！</font>枚举类本质上是用静态字段来实现的，只有当调用 getInstance 方法获取实例的时候，才会触发枚举类的加载，然后按照上面说的，生成一个静态字段并初始化其内部的单例 instance，因为 jvm 保证只能一个线程进行类加载，所以整个过程看起来非常的简单。（其实也就是实现了按顺序取用，可以在反编译的class文件中看到，enum是依次读取的，就不会出现线程冲突的问题了）





## 9、Spring的7大事务传播机制？和5大隔离级别？

### 9.0 啥叫传播？

简单的理解就是多个事务方法相互调用时,事务如何在这些方法间传播。(类比Mysql的事务机制，以及它的4大隔离级别——读未提交，读已提交，可重复度，序列化)

> 比如，方法A是一个事务的方法，方法A执行过程中调用了方法B，那么方法B有无事务以及方法B对事务的要求不同都会对方法A的事务具体执行造成影响，同时方法A的事务对方法B的事务执行也有影响，这种影响具体是什么就由两个方法所定义的事务传播类型所决定。

所以，我们要搞清楚事务的传播机制，这样我们才能搞清楚事务在不同方法调用的时候是怎么传递的，是重新创建事务还是使用父方法的事务？如果父方法回滚了，对子方法的事务是否有影响？事务不一致怎么办？

### 9.1、Spring的7大事务传播机制

在Spring源码中这七种类型被定义为了枚举。源码在org.springframework.transaction.annotation包下的Propagation:`public enum Propagation{...}`

1. REQUIRED：给方法A`@Transactional(propagation= Propagation.REQUIRED)`注解，如果当前没有事务，则自己新建一个事务，**子方法**是必须运行在一个事务中的； 如果当前存在事务，则加入这个事务，成为一个整体。
   - [举例：](https://blog.csdn.net/qq_17555933/article/details/116108558)老爸没钱吃饭，儿子有钱，儿子会自己买了自己吃【哈哈，很自私】。老爸有钱吃饭，会分吃的给儿子【这叫父爱】。

2. SUPPORTS：给方法A加上`@Transactional(propagation = Propagation.SUPPORTS)`注解，如果当前有事务，则使用事务；如果当前没有事务，则不使用事务（不会创建）。——所以要配合REQUIRED来使用
   - 举例：老爸没饭吃，儿子也没饭吃；老爸有饭吃，我也有饭吃。【同甘共苦】（不存在儿子有饭，老爸没饭的情况）

3. MANDATORY：给方法A加上`@Transactional(propagation = Propagation.MANDATORY)`注解，该传播属性强制必须存在一个事务，如果不存在，则抛出异常。——也是要配合REQUIRED使用
   - 举例：老爸叫儿子买包烟，儿子说给跑腿费【事务】我就去，否则我不去。【前提是要有事务，否则抛个异常给你】

4. REQUIRES_NEW：如果当前有事务，则挂起该事务，并且自己创建一个新的事务给自己使用；如果当前没有，则创建一个同Propagation.REQUIRES一样的事务。
   - 老爸请儿子吃饭，儿子偏不要，我有钱，为什么要你请？我自己去吃饭。但是如果吃饭失败，老爸和儿子可以不一起回滚（老爸有Required就一起，没有就不一起）。其实就是独立的个体了。
5. NOT_SUPPORTED：如果当前有事务，则把事务挂起，自己不使用事务去运行数据库操作，也不会回滚（没有事务）。
   - 老爸有钱吃饭，分点给儿子，儿子不鸟老爸，把钱放一边，这点钱能吃啥，不吃。
6. NEVER：从不使用事务。如果当前有事务，则抛出异常。
   - 老爸叫儿子买烟，给一个亿的跑腿费，儿子在上分打游戏没空，不鸟他，抛个异常给他。（儿子不会接受任何事务）
7. NESTED（嵌套）：如果当前有事务，则开启子事务（嵌套事务），嵌套事务是独立提交或者回滚； 如果当前没有事务，则开启同Propagation.REQUIRED一样的事务。（大套小）
   - 老爸带儿子出去斗地主，把钱都输光了，怕被老婆知道，让儿子一起背锅。老爸拿钱给儿子去赌一把，把钱输光了，就把责任推给儿子，让儿子一个人背锅。

### 9.2 Spring的5大隔离级别

> 涉及到事务，除了传播，就是隔离。这是两个方面。

类比数据库的4大隔离级别：由低到高分别为Read uncommitted 、Read committed 、Repeatable read 、Serializable 。

Spring增加了一个隔离级别：

1. ISOLATION_DEFAULT：使用数据库默认的隔离级别
2. ISOLATION_READ_UNCOMMITTED：允许读取尚未提交的修改，可能导致脏读、幻读和不可重复读
3. ISOLATION_READ_COMMITTED：允许从已经提交的事务读取，可防止脏读、但幻读，不可重复读仍然有可能发生
4. isolation_repeatable_read：对相同字段的多次读取的结果是一致的，除非数据被当前事务自生修改。可防止脏读和不可重复读，但幻读仍有可能发生
5. isolation_serializable：完全服从acid隔离原则，确保不发生脏读、不可重复读、和幻读，但执行效率最低。

> **PS：几种常用数据库的默认隔离级别**
>
> **MySQL**
>
> mysql默认的事务处理级别是'REPEATABLE-READ',也就是可重复读。
>
> **Oracle**
>
> oracle数据库支持READ COMMITTED 和 SERIALIZABLE这两种事务隔离级别。
>
> 默认系统事务隔离级别是READ COMMITTED,也就是读已提交。
>
> **SQL Server**
>
> 默认系统事务隔离级别是read committed,也就是读已提交







## 10、怎么解决Spring对象循环引用的问题？

三个角度去回答？先讲Spring的**Bean**怎么产生的？（BeanDefinition）；然后，讲为什么Bean不同于普通的对象，它更复杂，所以才会产生更多的问题）（也就是Bean的生命周期）；最后，讲他怎么解决的！

<font color='red'>PS：默认只支持单例，因为一创建就会实例化；原型不会，但是原型也有解决方法</font>

核心：利用了缓存（Map机制）！

- 问题来源：如果把两个或者多个Bean相互之间去持有对方引用的时候，就会发生循环依赖。而循环依赖，会导致注入死循环！
- 三种形态：
  - A依赖B，B依赖A：（互相装配）
  - A依赖B，B依赖C，C依赖A：
  - A依赖A：（自我依赖，自己装配自己）
- 解决方法：Spring的三级缓存！（一级缓存是成熟Bean，二级缓存是早期Bean，三级缓存是代理Bean）流程：
  - 当我们通过getBean()去获得一个对象实例的时候，Spring会先从一级缓存中去找，没有，就去二级缓存找；如果都没有找到，意味着这个Bean还没有实例化。
  - 然后才去实例化它，这个Bean叫**早期Bean**；并且把这个Bean加入到二级缓存中，并用一个标记表示他是否存在循环依赖。（如果不存在，才把Bean放到二级缓存；存在，就标记出来，并在在等待下一次轮询的时候去赋值，也就是**解析@Autowired注解**的时候去赋值）
  - 赋值完成后，将目标Bean存入一级缓存，这个时候就是成熟的Bean了。
- 总结：先取一级缓存，再取二级缓存。

但是这样的讲解，还是让人看不懂？？？我在这里面没看到Bean生命周期的任何一步，导致我看不懂！！

<font color='red'>————————————[更细粒度的答案](https://www.zhihu.com/question/38597960)————————————</font>

- 场景：如果现在有个A对象，它的属性是B对象，而B对象的属性也是A对象，说白了就是A依赖B，而B又依赖A。
- 解决流程：
  - 首先A对象实例化，然后对属性进行注入，发现依赖B对象
  - B对象此时还没创建出来，所以转头去实例化B对象
  - B对象实例化之后，发现需要依赖A对象，那A对象已经实例化了嘛，所以B对象最终能完成创建
  - B对象返回到A对象的属性注入的方法上，A对象最终完成创建

（这就是大白话？？？所以你会发现，最后是B先创建好，A再创建好！！！从这个顺序你就可以看到了）

更细致一点的流程：

1. Spring有三级缓存，本质上就是三个Map（singletonObjects一级缓存；earlySingletonObjects二级缓存，还没进行属性注入，由三级缓存变过来的；singletonFactories三级缓存中放着对象工厂）。其中，key是BeanName，Value是ObjectFactory
2. spring进行扫描，反射后把A封装成beanDefinition对象，放入beanDefinitionMap
3. 遍历map->验证（是否单例、是否延迟加载、是否抽象），并且推断构造方法。
4. 准备开始进行实例，去单例池中查，发现没有A这个Bean。然后就去二级缓存中找，这个时候还没有提前暴露。所以生成一个objectFactory对象（这是一个代理对象）暴露到三级缓存中。
5. **上面这才完成了实例化这一步。**然后，A属性注入的时候，发现依赖B。就要去实例化B。
6. B一样走上面的操作，当B属性注入需要去获取A对象，这里就是从三级缓存里拿出ObjectFactory，从ObjectFactory.getObject得到对应的Bean（就是对象A，但是现在还只是引用，也没有具体的值）
   - <font color='red'>但是这种说法仍然不够准确！！</font>我看到有些教程说的是，A这个时候的ObjectFactory是在二级缓存的，但是并不是：
     - 看一下源码，`ObjectFactory<T>`是定义在三级缓存中的。
   - 然后A再走一遍生命周期，再把当走到去二级缓存中找的时候找到了，然后才往B中注入objectFactory对象。（这也是不太准，还是要按我这个主流程走，B在第六步就已经拿到了A。当然，你说A的ObjectFactory就在二级缓存也是不无道理，因为反正A会被从三级缓存拿到二级缓存来。）
7. 然后三级缓存中的A就要被删掉，并且在二级缓存中加入A。然后B继续往后走，Bean都创建好了，加入到了一级缓存。A继续第5步（那怎么继续呢？等待Spring下一次**轮询**的时候），进行属性注入，最后也加入到了一级缓存。
8. 到此，A和B才真正创建好了！！（并且B在第6步获得的A的对象引用也完成了初始化）

### 10.1 二、三级缓存的作用？

这个问题其实要反过来想？是为啥会有三级代理？然后还要有二级代理？

- 三级缓存的作用是用来存储**代理Bean**的。当调用getBean()方法的时候，发现目标Bean需要通过代理工厂来创建，这个时候就把创建好的实例保存到三级缓存中。
  - 假设没有第三级缓存，只有第二级缓存（Value存对象，而不是工厂对象）。那如果有AOP的情况下，岂不是在存入第二级缓存之前都需要先去做AOP代理？这就很麻烦了，我们不如当B依赖A时，直接得到A对应的代理对象。

- 最终也会把三级缓存同步到一级缓存。
- 二级缓存，是为了性能！！从三级缓存的工厂里创建出对象，再扔到二级缓存（这样就不用每次都要从工厂里拿）



### 10.2 Spring在哪些情况下无法解决循环依赖？

四种！！

1. 多例Bean通过Setter方法注入的时候（原型只有在用到时才会走生命周期流程，但是原型不存在一个已经实例化好的bean，所以会无限的创建->依赖->创建->依赖->...。）——<font color='red'>原型Bean也有解决方法，比如，设置成非懒加载</font>
2. 构造器注入的Bean（因为加入singletonFactories三级缓存的前提是执行了构造器（单例对象此时已经被实例化好了），所以构造器的循环依赖没法解决；即构造方法不能创建出对象）
3. 单例的代理Bean通过Setter方法注入的时候
4. 设置了@DependsOn注解的Bean

其实，Spring只能解决单例模式下、非构造器创建的Bean！！

### 10.3 什么是提前暴露？

先理解怎么找到这个循环依赖：

- Spring容器会将每一个正在创建的Bean标识符放在一个“当前创建Bean池”中，**Bean标识符**在创建过程中将一直保持在这个池中，因此如果在创建Bean过程中发现自己已经在“当前创建Bean池”里时将抛出BeanCurrentlyInCreationException异常来表示**循环依赖**。（为什么能标记？这也是得益于单例模式！！一开始创建的，就会去实例化！这样，单例Bean一开始都会加到容器里面）

找到了，怎么解决？我们在创建bean的时候，首先想到的是从cache中获取这个单例的bean，这个缓存就是singletonObjects。它有两个主要的方法：

- isSingletonCurrentlyInCreation()判断当前单例bean是否正在创建中，也就是没有初始化完成(比如A的构造器依赖了B对象所以得先去创建B对象， 或者在A的populateBean过程中依赖了B对象，得先去创建B对象，这时的A就是处于创建中的状态。)
- allowEarlyReference 是否允许从singletonFactories中通过getObject拿到对象
  

PS：

1. **为什么要使用X的objectFacory对象而不是直接使用X对象？**
   - 利于拓展，程序员可以通过beanPostProcess接口操作objectFactory对象生成自己想要的对象
2. **是不是只能支持单例(scope=singleton)而不支持原型(scope=prototype)？**
   - 是。因为单例是spring在启动时进行bean加载放入单例池中，在依赖的bean开始生命周期后，可以直接从二级缓存中取到它所依赖的bean的objectFactory对象从而结束循环依赖。而原型只有在用到时才会走生命周期流程，但是原型不存在一个已经实例化好的bean，所以会无限的创建->依赖->创建->依赖->...。
3. **循环依赖是不是只支持非构造方法？**
   是。类似死锁问题

## 11、@Configuration原理

先看它的作用：

- 用于定义配置类,可替换xml配置文件；
- 被注解的类内部包含有一个或多个被@Bean注解的方法,这些方法将会被AnnotationConfigApplicationContext或AnnotationConfigWebApplicationContext类进行扫描,并用于构建bean定义,初始化Spring容器。

**流程：**

1. @Configuration注解的Bean，在BeanDefinition加载注册到IOC容器之后，进行postProcessBeanFactory处理时会进行CGLIB动态代理
2. 将@PropertySource、@ComponentScan、@Import、@ImportResource、@Bean等直接注解的类的BeanDefinition，是在ConfigurationClassParser#parse()中直接进行加载注册
3. 通过ConfigurationClassBeanDefinitionReader#loadBeanDefinitions()开始将@Configuration注解类内部@Import、@Bean进行BeanDefinition的加载注册

## 12、怎么解决单例Bean引用原型Bean，导致原型Bean作用域失效？

问题：当Singeton的bean当中引用了一个ProtoType的bean时，由于Singleton的bean只会初始化一次，所以作用域为ProtoType的bean不会被重新初始化，永远会是同一个bean。

解决方案：

- 实现Spring的ApplicationContextAware接口，在每次使用prototypeObject时，重新调用getBean方法。
- 通过注解@Lookup实现，在xml的单例Bean中配置`<bean id="singletonObject" class="com.example.demo.application.data.SingletonObject" scope="singleton"> <lookup-method name="createPrototypeObject" bean="prototypeObject"/>`
- aop:scoped-proxy标签；在xml中配置`<bean id="prototypeObject" class="com.example.demo.application.data.PrototypeObject" scope="prototype"> <aop:scoped-proxy />`



# 二、SpringMVC

## 1、工作流程

1. 用户发送请求到前端控制器DispatcherServlet
2. DispatcherServlet收到请求，调用HandlerMapping处理器映射器
3. HandlerMapping找到具体的处理器（可以根据xml配置、注解进行查找）**，生成处理器对象及处理器拦截器**(如果有则生成)一并返回给DispatcherServlet。
4. DispatcherServlet调用HandlerAdapter处理器适配器
5. HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)。
6. 后端处理完之后，返回结果给ModelAndView
7. HandlerAdapter将ModelAndView返回给DispatcherServlet。
8. DispatcherServlet将ModelAndView传给ViewReslover视图解析器。
9. ViewReslover解析后返回具体View。
10. DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。
11. DispatcherServlet响应用户。

## 2、配置springmvc-context.xml文件

1. 配置自动扫描包`<context:component-scan base-package="com.zjn" />`,完成Bean创建和自动依赖注入的功能。
2. 开启注解功能，将控制器与方法映射加入到容器中
3. 配置试图解析器







# 三、SpringBoot

## 1、springboot常用注解

1. @SpringBootApplcation：表示是一个SpringBoot工程
   - @SpringBootConfiguration：启动类
   - @EnableAutoConfiguration：向Spring容器中导入Selector，用来加载SpringFactories定义的自动配置类，将他们加载为配置Bean
   - @ComponentScan：标识扫描路径；
2. @Bean：定义Bean
   - @Autowired 用在属性、方法上，把配置好的Bean拿来用，完成属性、方法的组装。（依赖注入）
   - @Component，泛指组件
   - @Respository：用于数据持久层，标记DAO类
   - @Service：用于服务器层，注入到DAO层
   - @Controller：MVC控制层Bean，注入到Services层，标识当前类是一个控制器servlet
     - @RestController：继承了@Controller，使用这个特性，可以开发REST服务
   - @Configuration：声明配置类
   - @Scope：声明Bean的作用域
3. HTTP请求类型
   - @GetMapping
   - @PostMapping
   - @PutMapping
   - @DeleteMapping
4. 前后端参数传递
   - @RequestParam:用在方法的参数前面
   - @PathVariable：路径变量
   - @RequestBody：获取请求Body中的数据
   - @ResponseBody：表示该方法的返回结果直接写入HTTP body中（@RestController注解相当于@ResponseBody ＋ @Controller合在一起的作用。**返回的内容就是Return 里的内容。**）**前后端分离的时候多半用这种，不用再返回到View中去进行解析！**
5. 读取配置
   - @Value：直接读取属性名
   - @ConfigurationProperties：读取配置信息并与Bean绑定
   - @PropertySource：指定加载自定义配置文件
6. 参数校验—— 引入spring-boot-starter-validation.(比如，我要检查一个输入传参是不是邮件格式)
7. 异常处理
8. JPA数据持久化
9. JSON格式处理
10. 测试处理——@Test；

## 2、SpringBoot的属性配置优先级

1. 命令行参数`java -jar .\springbootconfiguraiton.jar --cl.name="Spring Boot Arguments"`。SpringApplication 类默认会把以“--”开头的命令行参数转化成应用中可以使用的配置参数，如 “--name=Alex” 会设置配置参数 “name” 的值为 “Alex”。
2. JVM系统属性（cmd输入`java -D xxx=value`），关键就是这个-D，指定了这是一个系统属性。
3. 操作系统环境变量(在cmd中输入export xxx=value；然后把项目打包，运行这个包)
4. 打包在应用程序内的application.properties或者application.yml文件
5. 通过@PropertySource标注的属性源
6. 默认属性

## 3、什么是Starter，原理是什么？

以前要使用一个组件，操作流程是：

1. 在Maven中导入使用的数据库的依赖（即JDBC的jar）
2. pom.xml中引入jpa的依赖
3. 在xxx.xml中配置一些属性信息
4. 反复的调试直到可以正常运行

但是每次有新的，就要重复一次，很麻烦！

Starter的理念是：

- starter会把所有用到的依赖都给包含进来，避免了开发者自己去引入依赖所带来的麻烦。需要注意的是不同的starter是为了解决不同的依赖，所以它们内部的实现可能会有很大的差异，例如jpa的starter和Redis的starter可能实现就不一样

**starter的实现：**虽然不同的starter实现起来各有差异，但是他们基本上都会使用到两个相同的内容：

- ConfigurationProperties：保存配置，并且这些配置都可以有一个默认值除此之外，starter的ConfigurationProperties还使得所有的配置属性被聚集到一个文件中（一般在resources目录下的application.properties），这样**我们就告别了Spring项目中XML地狱。**
- AutoConfiguration：在Spring上下文中创建一个Bean对象，并给他加上属性。就是去调用上面的具体的ConfigurationProperties。

**<font size='5'>源码分析（重点！！）：Starter本质上就是利用了SpringBoot的自动配置原理</font>**

1. springboot的都知道启动类上经常又个@SpringBootApplication注解，这个注解点进去看下是包括一个自动配置很关键的注解，那就是@EnableAutoConfiguration注解。
2. 点进去，关键功能是@import注解导入自动配置功能类AutoConfigurationImportSelector类（`@Import(EnableAutoConfigurationImportSelector.class)`）
3. 点进去，主要方法**getCandidateConfigurations**()使用了SpringFactoriesLoader.loadFactoryNames()方法加载**META-INF/spring.factories**的文件（spring.factories声明具体自动配置）。他将读取到的资源，包装成一个个的properties资源，返回！
4. META-INF/spring.factories：自动装配的文件，

```factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.ruoyi.common.redis.configure.RedisConfig,\
  com.ruoyi.common.redis.service.RedisService
```

5. 每一个内容就对应一个配置类（这个就是AutoConfiguration），他们的头都一样：（xxxAutoConfiguration和xxxProperties配合使用）

```
@Configuration		表示这是一个配置
@EnableConfigurationProperties(xxx.class)	这是在自动配置(用户配置的)application.properties中的属性

下面三个是Spring的低层注解：根据不同的条件来判断当前配置类是否生效
@ConditionalOnBean 当容器里有指定的Bean的条件下。
@ConditionalOnClass 当类路径下有指定的类的条件下。
@ConditionalOnProperty 指定的属性是否有指定的值。
```

class内部就是定义了具体的方法（比如redis类）：

```
public class RedisService
{
    @Autowired
    public RedisTemplate redisTemplate;
    public <T> void setCacheObject(final String key, final T value)
    {
        redisTemplate.opsForValue().set(key, value);
    }
public <T> void setCacheObject(final String key, final T value, final Long timeout, final TimeUnit timeUnit)
    {
        redisTemplate.opsForValue().set(key, value, timeout, timeUnit);
    }
}
```

因为上面已经有`@EnableConfigurationProperties(xxxProperties.class)`,他就会去xxxProperties类中找具体的配置（这个class中可以自定义，也可以由application.properties传入。

所以这就是一个**闭环**，使用Maven打包该项目。之后创建一个SpringBoot项目，在项目中添加这个starter作为依赖，然后使用SringBoot来运行我们的starter。

<font size='4' color='red'>**举个例子：**</font>

1. 创建一个starter项目（引入一些基本Maven依赖）
2. 创建一个ConfigurationProperties用于保存你的配置信息（比如这里创建HttpProperties（这个类就是定义了一个属性，其默认值是 http://www.baidu.com/，我们可以通过在 application.properties 中添加配置 http.url=https://www.zhihu.com 来覆盖参数的值。）

```
@ConfigurationProperties(prefix = "http") // 自动获取配置文件中前缀为http的属性，把值传入对象参数
@Setter
@Getter
public class HttpProperties {

    // 如果配置文件中配置了http.url属性，则该默认属性会被覆盖
    private String url = "http://www.baidu.com/";

}
```

3. 创建一个AutoConfiguration，引用定义好的配置信息；在AutoConfiguration中实现所有starter应该完成的操作

```
@Configuration
@EnableConfigurationProperties(HttpProperties.class)
public class HttpAutoConfiguration {

    @Resource
    private HttpProperties properties; // 使用配置

    // 在Spring上下文中创建一个对象
    @Bean
    @ConditionalOnMissingBean
    public HttpClient init() {
        HttpClient client = new HttpClient();

        String url = properties.getUrl();
        client.setUrl(url);
        return client;
    }

}
```

4. 把这个类加入spring.factories配置文件中进行声明

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.common.HttpAutoConfiguration,\
```

在上面的AutoConfiguration中我们实现了自己要求：在Spring的上下文中创建了一个HttpClient类的bean，并且我们把properties中的一个参数赋给了该bean。

5. 到此，我们的starter已经创建完毕了，使用Maven打包该项目。之后创建一个SpringBoot项目，在项目中添加我们之前打包的starter作为依赖，然后使用SringBoot来运行我们的starter

6. 创建我们的业务类

```
@Setter
@Getter
public class HttpClient {

    private String url;

    // 根据url获取网页数据
    public String getHtml() {
        try {
            URL url = new URL(this.url);
            URLConnection urlConnection = url.openConnection();
            BufferedReader br = new BufferedReader(new InputStreamReader(urlConnection.getInputStream(), "utf-8"));
            String line = null;
            StringBuilder sb = new StringBuilder();
            while ((line = br.readLine()) != null) {
                sb.append(line).append("\n");
            }
            return sb.toString();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "error";
    }

}
```

这个业务只包含了一个 url 属性和一个 getHtml 方法，用于获取一个网页的HTML数据。

由于我们在AutoConfiguration中已经对HttpClient这个类实现了配置（即init()函数），那url就会被注入响应的信息。

之后我们在application.properties中加入配置`http.url=https://www.zhihu.com/`,再次运行程序，此时打印的结果应该是知乎首页的HTML了，证明properties中的数据确实被覆盖了。



# 四、分布式

## 1、什么是分布式事务？

分布式事务就是指事务的参与者、支持事务的服务器、资源服务器以及事务管理器分别位于不同的分布式系统的不同节点之上。

产生的原因：

1. 数据库分库分表。当数据库单表一年产生的数据超过1000W，那么就要考虑分库分表。如果一个操作既访问01库，又访问02库，而且要保证数据的一致性
2. 业务的服务化：比如原来单机支撑了整个电商网站，现在对整个网站进行拆解，分离出了订单中心、用户中心、库存中心。

## 2、常见的分布式事务解决方案

1. **基于XA协议的两阶段提交（基于数据库实现）**

XA中大致分为两部分：事务管理器和本地资源管理器。

- 第一阶段：
  - 事务管理器通知参与该事务的各个资源管理器，通知他们开始准备事务。
  - 资源管理器接收到消息后开始准备阶段，写好事务日志并执行事务，但不提交，然后将是否就绪的消息返回给事务管理器
- 第二阶段：
  - 事务管理器在接受各个消息后，开始分析，如果有任意其一失败，则发送回滚命令，否则发送提交命令。
  - 各个资源管理器接收到命令后，执行（耗时很少），并将提交消息返回给事务管理器。

二阶段提交协议的存在的弊端是阻塞，因为事务管理器要收集各个资源管理器的响应消息，如果其中一个或多个一直不返回消息，则事务管理器一直等待，应用程序也被阻塞，甚至可能**永久阻塞**。

2. **消息事务+最终一致性（基于事务，或者说基于中间件）**

所谓的消息事务就是基于消息中间件的两阶段提交，本质上是对消息中间件的一种特殊利用，它是将本地事务和发消息放在了一个分布式事务里，保证要么本地操作成功成功并且对外发消息成功，要么两者都失败，开源的RocketMQ就支持这一特性

1. A系统向消息中间件发送一条预备消息
2. 消息中间件保存预备消息并返回成功
3. A执行本地事务
4. A发送提交消息给消息中间件

通过以上4步完成了一个消息事务。

- 步骤三出错，这时候需要回滚预备消息，怎么回滚？A系统实现一个消息中间件的回调接口，消息中间件会去不断执行回调接口，检查A事务执行是否执行成功，如果失败则回滚预备消息；

- 步骤四出错，这时候A的本地事务是成功的，那么消息中间件要回滚A吗？答案是不需要，其实通过回调接口，消息中间件能够检查到A执行成功了，这时候其实不需要A发提交消息了，消息中间件可以自己对消息进行提交，从而完成整个消息事务（其实也就是说不需要步骤4，A再发送消息给中间件了）

其中B系统的操作由消息驱动，只要消息事务成功，那么A操作一定成功，消息也一定发出来了，这时候B会收到消息去执行本地操作，如果本地操作失败，消息会**重投**（这里要注意幂等性，重复的消息不能重复地执行），直到B操作成功，这样就变相地实现了A与B的分布式事务。

3. **TCC编程模式（基于业务层面）**

TCC提供了一个编程框架，将整个业务逻辑分为三块：Try、Confirm和Cancel三个操作。以在线下单为例，Try阶段会去扣库存，Confirm阶段则是去更新订单状态，如果更新订单失败，则进入Cancel阶段，会去恢复库存。（try阶段通过预留资源的方式避免了同步阻塞资源的情况，但是TCC编程需要业务自己实现try,confirm,cancle方法，对业务入侵太大，实现起来也比较复杂。）假设有AB两个操作，假设A操作耗时短，那么A就能较快的完成自身的try-confirm-cancel流程，释放资源，无需等待B操作。如果事后出现问题, 追加执行补偿性事务即可。



## 3、CAP定理

一个分布式系统不可能同时满足一致性（C:Consistency)，可用性（A: Availability）和分区容错性（P：Partition tolerance）这三个基本需求，最多只能同时满足其中的2个。**而分区容忍性（P）必须要实现，**所以我们多数情况下需要在一致性（C）和可用性（A）之间进行权衡。

**分区容错性**：分布式系统在遇到任何网络分区故障的时候，仍然能够对外提供满足一致性和可用性的服务，除非整个网络环境都发生了故障。

## 4、Base-理论：最终一致性

BASE全称：Basically Available(基本可用)，Soft state（软状态）和 Eventually consistent（最终一致性）

- Basically Available(基本可用)：允许损失部分可用性，即保证核心可用。例如，在一个电商网站上，正常情况下，用户可以顺利完成每一笔订单，但是到了大促期间，为了保护购物系统的稳定性，部分消费者可能会被引导到一个**降级页面，只提供降级服务。**
- Soft state（软状态）：允许系统中的数据存在中间状态，并认为该状态不影响系统的整体可用性，即允许系统在多个不同节点的数据副本存在数据延时。

- Eventually consistent（最终一致性）：软状态可以存在数据延时，但必须有个时间期限。在期限过后，应当保证所有副本保持数据一致性。从而达到数据的最终一致性。这个时间期限视实际情况而定。

## 5、一致性协议之-2PC&3PC

### 5.1、2PC——Two-Phase Commit——二阶段提交协议

> 主要目的是为了保证分布式系统数据的一致性（这个和上面的基于XA的二阶段提交一致）

- 阶段一：提交事务请求
  - 事务询问（协调者 --> 参与者）：各单位是否准备好
  - 执行事务（参与者）
  - 返回对事务询问的响应（参与者 --> 协调者）
- 阶段二：执行事务提交
  - 协调者向所有参与者发出正式提交事务的请求（即Commit请求）。
  - 参与者执行Commit请求，并释放整个事务期间占用的资源。
  - 各参与者向协调者反馈Ack完成的消息。
  - 协调者收到所有参与者反馈的Ack消息后，即完成事务提交。

任何一个参与者反馈NO或者超时

缺点：同步阻塞，单点问题，数据不一致，过于保守；

**单点问题：**协调者是2PC的核心，如果协调者出现问题，那么整个流程将无法运转，更重要的是：其他参与者将会处于一直锁定事务资源的状态中，而无法继续完成事务操作。
**数据不一致：**假设当协调者向所有的参与者发送 commit请求时发生网络错误或者自身崩溃，导致最终只有部分参与者收到了 commit 请求。这将导致严重的数据不一致问题。

### 5.2、3PC——Three-Phase Commit——三阶段提交协议

与2PC不同的是，3PC有两个改动点：

1. 引入超时机制。同时在协调者和参与者中都引入超时机制。
2. 在第一阶段和第二阶段中插入一个准备阶段。保证了在最后提交阶段之前各参与节点的状态是一致的。

![3PC](https://img-blog.csdn.net/20180627184103801?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ1MTM4ODM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

增加了一个preCommit预处理阶段，相当于把commit前置了。假如协调者从所有的参与者获得的反馈都是Yes响应，那么就会执行事务的预执行。假如有任何一个参与者向协调者发送了No响应，或者等待超时之后，协调者都没有接到参与者的响应，那么就执行事务的中断。（在第二阶段就终止了）

在doCommit阶段（第三阶段），如果参与者无法及时接收到来自协调者的doCommit或者rebort请求时，**会在等待超时之后，会继续进行事务的提交**。（第三阶段是肯定要提交了）

相对于2PC，3PC主要解决的是**单点故障问题，并减少阻塞**，因为一旦参与者无法及时收到来自协调者的信息之后，他会默认执行commit。**而不会一直持有事务资源并处于阻塞状态。**

- **但是这种机制也会导致数据一致性问题，**因为，由于网络原因，协调者发送的abort响应没有及时被参与者接收到，那么参与者在等待超时之后执行了commit操作。这样就和其他接到abort命令并执行回滚的参与者之间存在数据不一致的情况。

## 6、Seata怎么解决分布式事务一致性？

可以先讲一下什么是分布式事务，然后说里面的一些概念（CAP理论、Base里面）

`Seata`有四种模式：

- AT（默认）：二阶段提交，AT 模式的一阶段、二阶段提交和回滚均由 Seata 框架自动生成，用户只需编写“业务 SQL”，便能轻松接入分布式事务，AT 模式是一种对业务无任何侵入的分布式事务解决方案。（相比于XA协议，AT把对数据库的操作，转移到Seata中，实现sql解析，自实现undolog；随之而来的就是阻塞降低，性能提升）
- TCC：三阶段。TCC 模式对业务代码有一定的侵入性，但是 TCC 模式无 AT 模式的全局行锁，TCC 性能会比 AT 模式高很多。（也是基于补偿的方式）
- Sage：长事务解决方法，事件驱动，补偿方案。业务流程中的每个参与者都要提交本地事务，当出现任何一个参与者失败的时候，则通过补偿的方案，把之前成功的参与者进行回滚。适合业务流程长/多，且无法提供TCC模式要求的三个接口。
- XA协议：强一致性，基于数据库实现。二阶段提交

## 8、[分布式一致性算法](https://www.cnblogs.com/yxy-ngu/p/12587341.html#_label1_1)：选主+复制日志

> 其实问到这个问题，真没必要解释太清楚；
>
> 核心就两个：
>
> 1. 选主 Leader Election
> 2. 复制日志 Log Replication
>
> 把这两部都做好了，就能解决master-slaver的同步
>
> 还不如把面试官引入到下面的问题——一致性hash问题

### 8.1 强一致性算法——Paxos、Raft（muti-paxos）、ZAB（muti-paxos）

除了上面Base理论中的2PC、3PC最终一致性方法外，还有非常经典的强一致性（保证系统改变提交后理科改变集群的状态）——Paxos、Raft（muti-paxos）、ZAB（muti-paxos）。

#### 8.1.1 Paxos：

> 前提：这里是非拜占庭将军问题（没有叛徒），但是可以接受有消息延迟

**paxos算法（Paxos Made Simple）就解决了在消息没有损坏（corrupted）的可信任环境的前提下，保证系统一致性。**Proposal提案，即分布式系统的修改请求，可以表示为[提案编号N，提案内容value]，Propser询问Acceptor中的多数派是否接收过N号的提案，如果都没有进入下一步，否则本提案不被考虑，Acceptor开始表决，Acceptor无条件同意从未接收过的N号提案，达到多数派同意后，进入下一步，Learner记录提案。

> Paxos的最大特点就是难，不仅难以理解，更难以实现。

它利用大多数 (Majority) 机制保证了2F+1的容错能力，即2F+1个节点的系统最多允许F个节点同时出现故障。

Paxos将系统中的角色分为提议者 (Proposer)，决策者 (Acceptor)，和最终决策学习者 (Learner):

- **Proposer**: 提出提案 (Proposal)。Proposal信息包括提案编号 (Proposal ID) 和提议的值 (Value)。
- **Acceptor**：参与决策，回应Proposers的提案。收到Proposal后可以接受提案，若Proposal获得多数Acceptors的接受，则称该Proposal被批准。
- **Learner**：不参与决策，从Proposers/Acceptors学习最新达成一致的提案（Value）。

在多副本状态机中，每个节点（或者说副本）同时具有Proposer、Acceptor、Learner三种角色。

Paxos算法通过一个决议分为两个阶段（Learn阶段之前决议已经形成）：

1. 第一阶段：Prepare阶段。Proposer向Acceptors发出Prepare请求，Acceptors针对收到的Prepare请求进行Promise承诺。
2. 第二阶段：Accept阶段。Proposer收到多数Acceptors承诺的Promise后，向Acceptors发出Propose请求，Acceptors针对收到的Propose请求进行Accept处理。
3. 第三阶段：Learn阶段。Proposer在收到多数Acceptors的Accept之后，标志着本次Accept成功，决议形成，将形成的决议发送给所有Learners。

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220921224059.png)

> 原始的Paxos算法（Basic Paxos）只能对一个值形成决议，决议的形成至少需要两次网络来回，在高并发情况下可能需要更多的网络来回，极端情况下甚至可能形成活锁。如果想连续确定多个值，Basic Paxos搞不定了。
>
> 因此有了下面的muti-paxos机制：Raft

缺点：

1. 活锁：当某一proposer提交的proposal被拒绝时，可能是因为acceptor承诺返回了更大编号的proposal,因此proposer提高编号继续提交。如果2个proposer
   都发现自己的编号过低转而提出更高编号的proposal,会导致死循环（这样，他们会一直交替上升自己的编号），这种情况也称为活锁。
   - 解决方法：二进制指数退避算法（CSMA/CD），其实就是退避算法是以冲突窗口大小为基准的，每个节点有一个冲突计数器C。退避的时间与冲突次数具有指数关系，冲突次数越多，退避的时间就可能越长，若达到限定的冲突次数，该节点就停止发送数据。
2. 效率低：每个完整流程的提议都需要经过两大轮请求返回（prepare一次，然后accept一次）
   - 原因：这种就是一个全分布式的情况，没有主从的概念在里面，导致每个人每次都要先确定好自己是不是主，然后在发送命令。



#### 8.1.2 [Raft(muti-paxos)](http://thesecretlivesofdata.com/raft/)-Redis && ETCD && 新的kafka

> 本质上就是一个Lear选举过程，但是理解起来更简单。 

Raft实现一致性的最重要两个功能就是：（**Redis中哨兵集群中的哨兵领导者选举和Redis-Cluster中的主节点选举都是基于Raft算法实现的。**ETCD的分布式协同也是基于Raft机制）

1. 选主 Leader Election（每个被选择的主，会不断地通过heartbeat的方式来通知自己的状态；集群中的每个节点都时刻处于Leader, Follower, Candidate这三个角色之一）
   - 当集群初始化时候，每个节点都是Follower角色；
   - 集群中存在至多1个有效的主节点，通过心跳与其他节点同步数据；
   - 当Follower在一定时间内没有收到来自主节点的心跳，会将自己角色改变为Candidate，并发起一次选主投票；当收到包括自己在内超过半数节点赞成后，选举成功；当收到票数不足半数选举失败，或者选举超时。若本轮未选出主节点，将进行下一轮选举（出现这种情况，是由于多个节点同时选举，所有节点均为获得过半选票）。
   - Candidate节点收到来自主节点的信息后，会立即终止选举过程，进入Follower角色。为了避免陷入选主失败循环，每个节点未收到心跳发起选举的时间是一定范围内的随机值，这样能够避免2个节点同时发起选主。
2. 复制日志 Log Replication：是指主节点将每次操作形成日志条目，并持久化到本地磁盘，然后通过网络IO发送给其他节点。
   - 其他节点根据日志的逻辑时钟(TERM)和日志编号(INDEX)来判断是否将该日志记录持久化到本地。当主节点收到包括自己在内超过半数节点成功返回，那么认为该日志是可提交的(committed），并将日志输入到状态机，将结果返回给客户端。
3. <font color='red'>安全性:</font>
   - 截止此刻，选主以及日志复制并不能保证节点间数据一致。
     - 场景：当某个节点挂掉了，一段时间后再次重启，并当选为主节点。而在其挂掉这段时间内，集群若有超过半数节点存活，集群会正常工作，那么会有日志提交。这些提交的日志无法传递给挂掉的节点。当挂掉的节点再次当选主节点，**它将缺失部分已提交的日志。**在这样场景下，按Raft协议，它**将自己日志复制给其他节点，会将集群已经提交的日志给覆盖掉**。
   - 一个解决这个问题的简单办法是，新当选的主节点会询问其他节点，和自己数据对比，确定出集群已提交数据，然后将缺失的数据同步过来。**这个方案有明显缺陷，增加了集群恢复服务的时间（集群在选举阶段不可服务），并且增加了协议的复杂度。**
   - Raft解决的办法是，在选主逻辑中，对能够成为主的节点加以限制，确保选出的节点已定包含了集群已经提交的所有日志(其实也就是看最后这个节点的是不是最后更新提交的那些半数节点之一)。如果新选出的主节点已经包含了集群所有提交的日志，那就不需要从和其他节点比对数据了。
     - 这里存在一个问题，加以这样限制之后，还能否选出主呢？答案是：只要仍然有超过半数节点存活，这样的主一定能够选出。因为已经提交的日志必然被集群中超过半数节点持久化，显然前一个主节点提交的最后一条日志也被集群中大部分节点持久化。当主节点挂掉后，集群中仍有大部分节点存活，那这存活的节点中一定存在一个节点包含了已经提交的日志了。

#### 8.1.3 ZAB（muti-paxos）-Zookeeper

> 所有事务请求必须由一个全局唯一的服务器来协调处理，这个服务器被称为Leader（只有一个），而余下的服务器被称为Follower。 

Leader负责将一个客户端的事务请求转换为一个事务提议（Proposal），并将该Propsal分发给集群中的所有Follower。 之后等待所有Follower的反馈，一旦超过半数的Follower进行了正确的反馈，那么Leader会再次向所有Follower发起Commit消息， 要求将上一次Proposal进行提交。**（Zookeeper就是这样用的，有一个选主过程）**

使用不同的消息模型，**「拜占庭将军问题」有不同的解法。**

- 如果将军之间使用口头消息(oral messages)，也就是说，消息被转述的时候是可能被篡改的，那么要对付m个叛徒，需要至少有3m+1个将军（其中至少2m+1个将军是忠诚的）。
- 如果将军之间使用签名消息(signed messages)，也就是说，消息被发出来之后是无法伪造的，只要被篡改就会被发现，那么对付m个叛徒，只需要至少m+2个将军，也就是说至少2个忠诚的将军（如果只有1个忠诚的将军，显然这个问题没有意义）。这种情况实际相当于对忠诚将军的数目没有限制。

## 9、分布式算法之 一致性Hash算法

> 利用hash算法，出现数据迁移时固有的。但是这也要看场景，hash算法在无状态数据中，迁移是没有影响的，因为不涉及到存储。但是如果是缓存之类的集群,节点的动态上下线会导致几乎所有的key的重新映射,这样造成的影响是数据错乱,相同备份的数据同时存在于集群中的多个节点,造成内存空间的浪费。

分布式缓存，有个hash缓存策略：

- 简单做法：hash(key)%机器数；但是这个时候机器数一旦变化，就要重新计算。但是重新计算时，机器数增加了，原本X文件缓存到A服务器，这个时候缓存到B了，就会出现**大量缓存失效**（缓存雪崩问题）的问题，进而可以导致系统开销增大。
- 改进一下：我们构造一个hash环，也就是2^32；我们先把服务器hash上去，几个服务器就把这个环切割成了多个区域；然后把文件hash上去，根据区域去选择服务器。这个时候，如果服务器增加，只会导致**小部分缓存失效**！而不会让所有压力，一次性都来到后端服务器。（所以，一般是有状态的情况下才会选择一致性hash算法）
  - 但是还有个问题——**hash偏斜！**就是说服务器在hash环上的位置特别近，导致文件缓存的服务器出现偏移，即A-B-C，大部分都在A和C上，B上很少。解决方法就是映射虚拟节点，把hash分布均匀化处理。（其实也是一种解决hash冲突的方法）

### 9.1 简述这里hash算法的选择问题

有状态负载均衡策略，除了要达到让负载均衡分散到节点的目标以外，还需要实现将同一对象的请求分发到同一个节点。例如业务场景需要将同一个用户的全部请求发送到后端同一个节点处理的情况。

- 主要负载均衡策略是一致性hash

北极星默认提供基于[割环法的一致性hash负载均衡策略](https://git.woa.com/polaris/polaris-go/blob/master/plugin/loadbalancer/ringhash/continuum.go)，可以通过一致性hash的方式从可用的服务实例中选择一个实例进行返回 （走动态路由，以及一致性hash负载均衡策略） 

```
const DefaultHashFuncName = "murmur3"

var (
	murmur3HashPool = &sync.Pool{}
)

//通过seed的算法获取hash值
func murmur3HashWithSeed(buf []byte, seed uint32) (uint64, error) {
	var pooled = seed == 0
	var hasher hash.Hash64
	if pooled {
		poolValue := murmur3HashPool.Get()
		if !reflect2.IsNil(poolValue) {
			hasher = poolValue.(hash.Hash64)
			hasher.Reset()
		}
	}
	if nil == hasher {
		hasher = murmur3.New64WithSeed(seed)
	}
	var value uint64
	var err error
	if err = WriteBuffer(hasher, buf); nil == err {
		value = hasher.Sum64()
	}
	if pooled {
		murmur3HashPool.Put(hasher)
	}
	return value, err
}
```

可以看到北极星的hash算法调用了murmur3算法（这个也基本上是底层支撑令牌的通用算法了），相比于MD5、sha1这种加密型Hash算法，MurmurHash 是一种非加密型哈希函数，适用于一般的哈希检索操作。

- **为什么叫非加密型？**

加密哈希函数旨在保证安全性，很难找到碰撞。非加密哈希函数只是试图避免非恶意输入的冲突。作为较弱担保的交换，它们通常更快。如果数据量小，或者不太在意哈希碰撞的频率，甚至可以选择生成哈希值小的哈希算法，占用更小的空间（比如，murmur3就提供了32位hash和128位的hash运算）。

所以，可以看到一些涉及到用户数据的场景，MD5出现频率高，但是像寻址这种安全风险低的行为，[murmur3才是主流。](https://juejin.cn/post/6844903747320021000)

他与其它Hash算法项目，优势主要体现在：

1. **运算速度快（比安全散列算法快几十倍）**——MD5只能处理固定长度，少的地方需要填充；MD5涉及到三个量之间的线性运算；并且MD5需要经历4层循环（murmur3只需要一层）

```
//MD5的四个基础线性函数
F(X,Y,Z) = (X & Y) | ((~X) & Z);
G(X,Y,Z) = (X & Z) | (Y & (~Z)); 
H(X,Y,Z) = X ^ Y ^ Z; 
I(X,Y,Z) = Y ^ (X | (~Z));
```

2. **变化足够激烈，相似的字符串如“abc”和“abd”能够均匀散落在哈希环上**

特别是第2点，是它的特色；原因在于它的hash是分段hash：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/001.png)

他有五个固定的参数“c1=0xcc9e2d51” “c2=0x1b873593” “m=0x5” “n=0xe6546b64” “Hash=seed”；（经过模拟退火算法求出来的最佳参数，也叫“幻数”）

1. 分段后的数据，进行移位后重新拼接

```
abcd变成16进制并分别左移（留下e）
0x61→0x61000000 (左移24位)
0x62→0x00620000 (左移16位)
0x63→0x00006300 (左移8位)
0x64→0x00000064
```

2. 相加，赋值给k
   K = 0x61626364（伪代码中的 `k <——(k<<r1)OR(k>>(32-r1))`）

3. 对K进行hash操作，并在其中“加盐”，（k\*c1，k*c2）
4. 最后处理，遗留位（e）；Hash=Hash XOR k



## 10、分布式算法之 雪花算法UUID

雪花算法生成的Id由：1bit 不用 + 41bit时间戳+10bit工作机器id+12bit序列号，如下图：

![img](https://upload-images.jianshu.io/upload_images/4843132-200596b026e647f8.png?imageMogr2/auto-orient/strip|imageView2/2/w/490/format/webp)

**不用：**1bit，因为最高位是符号位，0表示正，1表示负，所以这里固定为0
 **时间戳：**41bit，服务上线的时间毫秒级的时间戳（为当前时间-服务第一次上线时间），这里为（2^41-1）/1000/60/60/24/365  = 49.7年
 **工作机器id：**10bit，表示工作机器id，用于处理分布式部署id不重复问题，可支持2^10 = 1024个节点
 **序列号：**12bit，用于离散同一机器同一毫秒级别生成多条Id时，可允许同一毫秒生成2^12 = 4096个Id，则一秒就可生成4096*1000 = 400w个Id



### 10.1、当有多个用户同时请求的时候，UUID会重复吗？

不会！

- 首先，生成ID的方法是加了synchronized 关键词，确保了线程安全，否则在并发情况下，生成的Id就有可能重复了。
- 如果同一毫秒生成多个id，则两者相等，
  - 解决方法，就是采用**sequence递增**
  - 如果sequence递增到4095重新回到0时，证明当前毫秒已经产生了4096个序列号，则使用tilNextMillis(lastTimestamp)方法**阻塞到下一毫秒**并赋值给timestamp。

缺点：

- 严重依赖系统时钟，如果时钟回拨，会出现时间戳重复，导致ID重复（所以可以搞个记录时钟的持久化层）
- UUID的唯一缺陷在于生成的结果串会比较长。



### 10.2、雪花算法生成Id重复导致数据库中表主键冲突，导致入库失败的问题？

针对微服务场景中，服务器器id重复导致的情况！

- 工作机器Id的作用，就是用于解决分布式Id重复的问题，这个workerId是通过构造方法传入的，如果我们用10位来存储这个值，那就是最多支持1024个节点
  - 我们用机器的唯一名来做key，那我们可以对这些机器名和workerId建立一个对应关系，如果存在就用之前的workerId，不存在就往上累加比如我们用计算机名做key。这样机器如果不断累加，最多支持1024台服务器
  - 如果是容器化部署，需要支持动态增加节点，并且每次部署的机器不一定一样时，就会有问题，如果发现不同，就往上累加，经过多次发版，就可能会超过1023，这个时候生成雪花Id时，工作机器id左移12位后，当进行或运算时，时间戳的位置就会被影响，比如workerId=1024，我们拿之前的举例第1000ms，那它和第1001ms、workerId=0配置，可能生成重复的Id

**优化方案为：**

1. 在redis中存储一个当前workerId的最大值
2. 每次生成workerId时，从redis中获取到当前workerId最大值，并+1作为当前workerId，并存入redis
3. 如果workerId为1023，自增为1024，则重置0，作为当前workerId，并存入redis

上述方案确保了，任何时间点不同服务器的workerId一定不一致。但是也有问题，如果自增1新分配的workerId还没释放掉，这个时候就会冲突了 比如我们第一个pod(workerId=0)一直没有重启过，但是第二个pod一直在重启，达到1024时回到0，则同时会有两个pod的workerId为0, 这两台pod上程序生成的Id就有可能重复。

## 11、跨域问题？？

### 11.1、怎么解决跨域一致性？（针对前后端分离）——微服务版本不需要考虑

**现象：**在`localhost:8080`的服务中访问`localhost:8181/list`接口时被CORS接口阻止（请求中缺少头信息）。

CORS策略：允许浏览器向跨源服务器，发出ajax请求，从而克服了AJAX只能同源使用的限制。 CORS依赖于服务器端的设定，**只要在服务器端进行了设置，就可以实现相应的资源访问**。

**分析：**后端其实接收到请求了，并且返回给前端了，只不过前端浏览器的设置（同源策略进行了拦截），导致了报错。

- 同源策略也是为了安全。他要求url中的协议、域名、端口3个属性都相同才叫做同源。

解决方案：上面的黑字部分——只要在服务器端进行了设置，就可以实现相应的资源访问，SpringBoot中有三种方式：

- 在目标函数上添加@CrossOrigin

- 添加CORS过滤器：新建一个配置类@Configuration，然后在里面注入corsFilter过利器

- 实现WebMvcConfigure接口，重写addCorsMappings方法，添加具体的规则：

  - ```
    @Override
    public void addCorsMappings (CorsRegistry registry) {
    registry.addMapping( pathPattern: " /**")
    .allowed0riginPatterns("*")
    .allowedMethods("GET", "POST", "PUT", "DELETE", "HEAD", "OPTIONS")
    .alLowCredentials(true )
    .maxAge (3600)
    .allowedHeaders ("*") ;
    ```

### 11.2、不同域名(多域名)下共享登录状态（单点登录问题）

共享登录状态，即：实现在一处登录后，访问另一个站点就可以不用在登录了。

1. 传统的做法是可以在cookie里面的domain属性设置需要跨域的域名，这样就可以在多个站点实现**共享cookie**，也就是可以通过这种方式共享登录状态。（用户登录成功拿到token(或者是session-id)后怎么让浏览器存储和分享到其它域名下？同域名很简单，把token存在cookie里，把cookie的路径设置成顶级域名下，这样所有子域都能读取cookie中的token）这种方式比较简单快捷，但是有一个缺陷就是，共享cookie的站点需要是同一个顶级域名，google.com是他的顶级域名，邮箱服务的mail.google.com和地图服务的map.google.com都是它的子域。但是，跨域的时候怎么办？谷歌公司还有一个域名，youtube.com，提供视频服务。
2. SSO实现单点登录：相比于单系统登录，sso需要一个独立的**认证中心**，只有认证中心能接受用户的用户名密码等安全信息，**其他系统不提供登录入口，只接受认证中心的间接授权。**间接授权通过令牌实现，sso认证中心验证用户的用户名密码没问题，创建授权令牌，在接下来的跳转过程中，**授权令牌作为参数发送给各个子系统，**子系统拿到令牌，即得到了授权，可以借此创建局部会话，局部会话登录方式与单系统的登录方式相同。
   - sso认证中心一直监听全局会话的状态，一旦全局会话销毁，监听器将通知所有注册系统执行注销操作




## 12、负载均衡

1. **为什么要负载均衡？**

- 单点到集群的过程中，要解决两个问题1）客户端的请求均匀地发布到多台目标服务器上；2）并且能够检测出服务器的运行状态，让客户端不去请求到宕机的服务器；

2. 如何实现负载均衡？

- 基于DNS：在DNS服务器上针对某个域名做多个IP映射，
- 基于硬件：大型网络交换机，如F5
- 基于软件：Nginx、LVS等

二层负载：要么通过虚拟的**MAC地址**去实现均衡

三层负载：是通过**虚拟IP**去实现均衡。

四层负载：报文的目标地址和端口；

七层负载：基于应用层的负载（比如服务响应时间），根据报文信息去实现均衡，比如cookie、RequestHeader等

**常用算法：**

1. 轮询：对服务器轮询分发请求
2. 随机：随机分发
3. 一致性Hash：具有相同Hash码的请求，永远发送到同一台服务器
4. 最小连接数：根据目标服务器的请求数量来决定请求的分发权重，即请求少的节点会获得更多的请求。（比较好的）
5. 基于权重

### 12.1、Nginx负载均衡

1. 主从模式：nginx 启动后会创建多个进程，一个 Master 进程和多个 Worker 进程，Master 进程主要负责读取配置文件，管理维护多个 Worker 进程，像是一个大内总管，Master 自身不处理用户请求，用户访问的 web service 都是通过多个 Worker 进程处理
2. 异步非阻塞机制：运用了epoll模型，提供了一个队列，排队解决，不会为每个请求分配cpu和内存资源，节省了大量资源，同时也减少了大量的CPU的上下文切换。以小孩上厕所为例，Nginx不会去询问每个小孩要不要上厕所（即每个去发资源），而是直接到厕所等小孩，有小孩就直接操作（这个时候再给资源）
3. 模块化设计：Nginx 共有五大类型的模块分别处理进程管理、事件驱动、邮件服务发送等操作

**为什么nginx 不使用多线程？**（本质上就是简单，加速）

- 采用独立的进程，彼此之间不会影响，提高可靠性。即使一个进程崩溃，其他进程也没事。
- 进程不共享资源，不需要加锁，减少锁的开销，提高可用性
- 进程数已经等于核心数，增加线程没有意义，增加切换的代价。
- 采用epoll，需要耗时的i/o已处理为非阻塞/全异步/事件驱动，再利用多线程处理，意义不大。如果进程中还有其他阻塞的逻辑，应该各个业务/模块自行解决。

### 12.2、解决并发：漏桶流算法和令牌桶算法？

- **漏桶算法：**网络世界中流量整形或速率限制时经常使用的一种算法，它的主要目的是控制数据注入到网络的速率，平滑网络上的突发流量。突发流量会进入到一个漏桶，漏桶会按照我们定义的速率依次处理请求，如果水流过大也就是突发流量过大就会直接溢出，则多余的请求会被拒绝。**所以漏桶算法能控制数据的传输速率。**（但是短时间内的大量流量，会出现溢出不处理）
- **令牌桶算法**：令牌桶算法用来控制发送到网络上的数据的数目，并允许突发数据的发送。如果令牌消耗速率小于生产令牌的速度，令牌就会一直产生直至装满整个令牌桶。(可以在短时间内应对突发的流量；)



# 五、微服务(SpringCloud)

## 1、SpringCloud的核心组件？

- 服务注册与发现：Netflix Eureka、**阿里巴巴的Nacos**、Zookeeper
- 客户端负载均衡：Ribbon、SpringCloud LoadBalancer（2020新加入）
- 服务熔断器（降级、隔离）： **Hystrix**（传统，不推荐），Alibaba Sentinel、Resillence4J
- 服务网关：**SpringCloud Gateway**、Netflix Zuul
- 服务接口调用：**Netflix Feign**、 Resttemplate、 Openfeign
- 链路追踪：Netlix Sleuth、 Skywalking（用的最多）、 Pinpoint
- ~~聚合Hystrix监控数据：Netflix Turbine~~
- 监控中心：**SpringBoot Admin**（可以看到一些实例，进程等）
- 配置中心：**Spring Cloud Config**、Apollo、**nacos**

## 2、微服务架构原理？

主要是面向SOA理念，更细小粒度服务的拆分,将功能分解到各个服务当中，从而降低系统的耦合性,并提供更加灵活的服务支持。

微服务的 优点：

- **独立开发** – 所有微服务都可以根据各自的功能轻松开发
- **独立部署** – 基于其服务，可以在任何应用程序中单独部署它们
- **故障隔离** – 即使应用程序的一项服务不起作用，系统仍可继续运行
- **混合技术堆栈** – 可以使用不同的语言和技术来构建同一应用程序的不同服务
- **粒度缩放** – 单个组件可根据需要进行缩放，无需将所有组件缩放在一起



## 3、注册中心？

1. 服务启动后向**Eureka注册**，Eureka Server会将注册信息向其他Eureka Server进行同步,当服务消费者要调用服务提供者，则向服务注册中心获取服务提供者地址，然后会将服务提供者地址缓存在本地，下次再调用时，则直接从本地缓存中获取服务列表来完成服务调用。
2. **Nacos** = Spring Cloud Eureka + Spring Cloud Config。更强大。
   - 生产者注册到nacos注册中心，步骤： 添加依赖：spring-cloud-starter-alibaba-nacos-discovery及springCloud
   - 在 application.yml 中配置nacos服务地址和应用名
   - 启动服务，然后登陆Nacos服务（8848）查看，结果如下
3. 整合Feign实现远程调用：
   - 在ConsumerApplication类（微服务）上添加@EnableFeignClients注解

## 4、配置中心？

在服务运行之前，将所需的配置信息从配置仓库拉取到本地服务，达到统一化配置管理的目的。（比如，统一存储在git上，或者MySQL上；比如Apollo就是先把配置信息放到数据库中，用户需要把配置文件拉取到本地）



## 5、Feign原理简述

在Spring Cloud Netflix栈中，各个微服务都是以HTTP接口的形式暴露自身服务的，因此在调用远程服务时就必须使用HTTP客户端。我们可以使用JDK原生的URLConnection、Apache的Http Client、**Netty的异步HTTP Client,** Spring的RestTemplate。**但是用起来最方便的还是要属Feign了。**

当服务数量不是很大时，使用普通的分布式RPC架构即可，当服务数量增长到一定数据，需要进行服务治理时，就需要考虑使用流式计算架构。（如果项目对性能要求不是很严格，可以选择使用Feign，它使用起来更方便。 如果需要提高性能，避开基于Http方式的性能瓶颈，可以使用Dubbo。）

### 5.1 定义

1. Feign是Netflix开发的声明式、模板化的HTTP客户端， Feign可以帮助我们更快捷、优雅地调用**HTTP API**，类似controller调用service。
2. Spring Cloud Feign是基于Netflix feign实现，整合了Spring Cloud Ribbon（负载均衡）和Spring Cloud Hystrix（熔断降级）。
3. Spring Cloud对Feign进行了增强，使Feign支持了Spring MVC注解，并整合了Ribbon和Eureka，从而让Feign的使用更加方便。

### 5.2 作用

现有的服务调用方式——利用拼接的方式：

- Eureka调用http://ip:port/path,我们要用代理切割重新拼接为http://serviceName/path.这种就必要原始，对用户而言看不出来，但是对于开发者而言就比较难以维护了，他要手动去实现地址的代理。（并且，一旦serviceName修改了接口/方法，开发者也要去配合修改）	

所以，Feign出来了，他来实现远程过程调用（RPC）。开发者只需要在需要创建一个接口（`public interface RemoteFileService`）并使用注解的方式(`@FeignClient(contextId = "remoteFileService", value = ServiceNameConstants.FILE_SERVICE, fallbackFactory = RemoteFileFallbackFactory.class)`)来配置它(`@PostMapping(value = "/upload"`)，即可完成对服务提供方的接口绑定（服务方的启动类上，要加上feign注解：@EnableFeignClients或者@EnableRyFeignClients），这就比较方便了！

实操：https://blog.csdn.net/qq_37488998/article/details/111871744

### 5.3 消息处理流程

1. 启动时，程序会进行包扫描，扫描所有包下所有@FeignClient注解的类，并将这些类注入到spring的IOC容器中。当定义的Feign中的接口被调用时，通过JDK的动态代理来生成RequestTemplate。
2. RequestTemplate中包含请求的所有信息，如请求参数，请求URL等。
3. RequestTemplate生产Request，然后将Request交给client处理，这个client默认是JDK的HTTPUrlConnection，也可以是OKhttp、Apache的HTTPClient等。
4. 最后client封装成LoadBaLanceClient，结合ribbon负载均衡地发起调用。

### 5.4 基于springcloud异步线程池、高并发请求feign的解决方案（PS:问队列，也可以回答线程池，或者中间件）

问题：在高并发请求下，如何管理注册的服务？或者问你，项目中哪里用到了线程池？——我们一定要理解清楚，Feign是为了微服务之间的调用（在getByOrderNo服务中心调用productAPI.getById(1)，productAPI是远程服务，然后getById是它的方法），不是为了客户端到服务器之间的调用（HTTP的@RestController等注解）。

除了Feign，自己也用过（但是线程池主要还是降低资源消耗，提高响应速度嘛。java中经常需要用到多线程来处理一些业务，我们非常不建议单纯使用继承Thread或者实现Runnable接口的方式来创建线程，那样势必有创建及销毁线程耗费资源、线程上下文切换问题。[比如创建一个单线程化的线程池，](https://blog.csdn.net/fanrenxiang/article/details/79855992)一般的处理基本不太需要）。

```java
    //创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。	
    public static void singleThreadExecutor() {
        //创建一个单线程化的线程池
        ExecutorService singleThreadExecutor = newSingleThreadExecutor();
        for (int i = 0; i < 5; i++) {
            final int index = i;
            singleThreadExecutor.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        //结果依次输出，相当于顺序执行各个任务
                        System.out.println(Thread.currentThread().getName() + "正在被执行,打印的值是:" + index);
                        Thread.sleep(5000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
        singleThreadExecutor.shutdown();
    }
```

ok，话说回来，看看[Feign是怎么利用的线程池](https://www.jb51.net/article/206288.htm)~~~~~~~~~~~~~~

1. 配置application.yaml（bootstrap.yml）文件（这个文件在nacos中配置）

```yaml
spring.application.name=scen-task-test
server.port=9009
feign.hystrix.enabled=true
#熔断器失败的个数==进入熔断器的请求达到1000时服务降级（之后的请求直接进入熔断器）
hystrix.command.default.circuitBreaker.requestVolumeThreshold=1000
#回退最大线程数
hystrix.command.default.fallback.isolation.semaphore.maxConcurrentRequests=50
#核心线程池数量
hystrix.threadpool.default.coreSize=130
#请求处理的超时时间
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=100000
ribbon.ReadTimeout=120000
#请求连接的超时时间
ribbon.ConnectTimeout=130000
eureka.instance.instance-id=${spring.application.name}:${spring.application.instance_id:${server.port}}
eureka.instance.preferIpAddress=true
eureka.client.service-url.defaultZone=http://127.0.0.1:9000/eureka
logging.level.com.test.user.service=debug
logging.level.org.springframework.boot=debug
logging.level.custom=info
```

2. 定义异步线程池的配置——AsynConfig.java——7大参数

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {
 @Override
 public Executor getAsyncExecutor() {
  //定义线程池
  ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
  //核心线程数
  taskExecutor.setCorePoolSize(20);
  //线程池最大线程数
  taskExecutor.setMaxPoolSize(100);
  //线程队列最大线程数
  taskExecutor.setQueueCapacity(10);
  //初始化
  taskExecutor.initialize();
  return taskExecutor;
 }
}
```

3. 编写异步工作任务，一个Feign的客户端，调用远程服务userservice.java——DoTaskClass.java

```java
@Component
public class DoTaskClass { 
 /**
  * 一个feign的客户端
  */
 private final UserService userService;
  
 @Autowired
 public DoTaskClass(UserService userService) {
  this.userService = userService;
 }
  
 /**
  * 核心任务
  *
  * @param uid
  */
 @Async
 public void dotask(String uid) {
  /**
   * 模拟复杂工作业务（109个线程同时通过feign请求微服务提供者）
   */
  {
   List<UserEducation> userEducationByUid = userService.findUserEducationByUid(uid);
   List<String> blackList = userService.getBlackList();
   String userSkilled = userService.getUserSkilled(uid);
   String userFollow = userService.getUserFollow(uid);
   User userById = userService.getUserById(uid);
   List<String> followList = userService.getFollowList(uid);
   int userActivityScore = userService.getUserActivityScore(uid);
  }
//  打印线程名称分辨是否为多线程操作
  System.out.println(Thread.currentThread().getName() + "===任务" + uid + "执行完成===");
 }
}
```

4. 服务端编写执行操作，接受到用户test请求——TestController.java

```java
public class TestController {
  
 /**
  * 此处仅用此feign客户端请求微服务获取核心工作所需参数
  */
 private final UserService userService;
  
 /**
  * 核心工作异步算法
  */
 private final DoTaskClass doTaskClass;
  
 @Autowired
 public TestController(DoTaskClass doTaskClass, UserService userService) {
  this.doTaskClass = doTaskClass;
  this.userService = userService;
 } 
  
 /**
  * 手动触发工作
  * @throws InterruptedException
  */
 @RequestMapping("/test")
 public void task() throws InterruptedException {
  /*
   取到1000个要执行任务的必备参数
   */
  List<User> userList = userService.findAllLite(1, 1000);
  for (int i = 0; i < userList.size(); i++) {
   try {
//    异步线程开始工作
    doTaskClass.dotask(userList.get(i).getId());
   } catch (Exception e) {
    /*
     若并发线程数达到MaxPoolSize+QueueCapacity=110（参考AsyncConfig配置）会进入catch代码块
     i--休眠3秒后重试（尝试进入线程队列：当且仅当109个线程有一个或多个线程完成异步任务时重试成功）
     */
    i--;
    Thread.sleep(3000*3);
   }
   System.out.println(i);
  }
 } 
}
```

在这里可以看到[基础的Feign工作流程](https://blog.csdn.net/BiandanLoveyou/article/details/118073668)

### 5.5 Feign使用 FallbackFactory 实现熔断降级

1. 创建 ProductFallBackFactory 并实现 FallbackFactory

```
package com.study.fallback;
 
import com.study.api.ProductAPI;
import com.study.entity.ProductEntity;
import feign.hystrix.FallbackFactory;
import org.springframework.stereotype.Component;
 
/**
 * @author biandan
 * @description
 * @signature 让天下没有难写的代码
 * @create 2021-06-20 下午 7:33
 */
@Component
public class ProductFallBackFactory implements FallbackFactory<ProductAPI> {
 
    //如果调用异常，使用熔断机制返回错误信息（目前写固定）
    @Override
    public ProductAPI create(Throwable throwable) {
        return new ProductAPI() {
            @Override
            public ProductEntity getById(Integer id) {
                System.out.println("系统调用产品微服务失败，异常信息：" + throwable);
                ProductEntity product = new ProductEntity();
                product.setId(166);
                product.setProductName("托底数据-feign产品");
                product.setPrice(new Float(2.25));
                return product;
            }
        };
    }
}
```

其中，Throwable throwable 就是捕获到的异常信息。

2. ProductAPI 需要修改 FeignClient 如下，使用 fallbackFactory

```java
@FeignClient(value = "product-server",fallbackFactory = ProductFallBackFactory.class)
```

重启测试，把product-server停掉。页面还能正常显示（一个托底页面）

在控制台：

```
2021-06-20 20:15:11.271  INFO 11792 --- [erListUpdater-0] c.netflix.config.ChainedDynamicProperty  : Flipping property: product-server.ribbon.ActiveConnectionsLimit to use NEXT property: niws.loadbalancer.availabilityFilteringRule.activeConnectionsLimit = 2147483647
系统调用产品微服务失败，异常信息：com.netflix.hystrix.exception.HystrixTimeoutException
```

可见，Feign的低层熔断技术用的是hystrix。



## 5、Dubbo

> 其实看到dubbo 的组成(服务端，消费者，注册中心，监控中心)你就能知道，一个常见的微服务框架就会有哪些部分，比如用nacos做注册与发现，用admin做监控。**dubbo更多地，他是支持了消息传递的功能，也就是中间件的一种扩展！所以他还可以支持多种协议，并且和rpc框架一起做扩展！**
>
> Dubbo详解，用心看这一篇文章就够了：https://blog.csdn.net/weixin_42039228/article/details/123678364

### 5.1、Spring Cloud和Dubbo有哪些区别

1. **dubbo是二进制传输，对象直接转成二进制，使用RPC通信**。SpringCloud是http传输，同时使用http协议一般会使用JSON报文, json再转二进制，消耗会更大。
2. Dubbo只是实现了服务治理，而Spring Cloud下面有几十个子项目分别覆盖了微服务架构下的方方面面，服务治理只是其中的一个方面，一定程度来说，Dubbo只是Spring Cloud Netflix中的一个子集。
3. 组成不一样：dubbo的服务注册中心为Zookeerper，服务监控中心为dubbo-monitor，无消息总线、服务跟踪、批量任务等组件；Spring Cloud的服务注册中心为spring-cloud netflix enruka，服务监控中心为spring-boot admin，有消息总线、数据流、服务跟踪、批量任务等组件；


### 5.2 Dubbo概述

> Dubbo是阿里巴巴开源的基于 Java 的高性能RPC（一种远程调用） 分布式服务框架，致力于提供高性能和透明化的RPC远程服务调用方案，以及SOA服务治理方案。
>
> 本质上是个远程服务调用的分布式框架（告别Web Service模式中的WSdl（Web服务描述语言，Web Services Description Language；基于 XML 的用于描述 Web Services 以及如何访问 Web Services 的语言），以服务者与消费者的方式在Dubbo上注册）

其核心部分包含：

1、**远程通讯：**提供对多种基于长连接的NIO框架抽象封装，包括多种线程模型，序列化，以及“请求-响应”模式的信息交换方式。
2、**集群容错：**提供基于接口方法的透明远程过程调用，包括多协议支持，以及软负载均衡，失败容错，地址路由，动态配置等集群支持。
3、**自动发现：**基于注册中心目录服务，使服务消费方能动态的查找服务提供方，使地址透明，使服务提供方可以平滑增加或减少机器。











## 6、Ribbon负载均衡原理是什么

原理：靠拦截器拦截住带有ribbon注解的方法，然后将方法转换成我们所需要的服务的IP地址

1. **Ribbon通过ILoadBalancer接口对外提供统一的选择服务器(Server)的功能，**此接口会根据不同的负载均衡策略
   (Rule)选择合适的Server返回给使用者。
2. **lRule是负载均衡策略的抽象**，ILoadBalancer通过调用IRule的choose()方法返回Server
3. IPing用来检测Server是否可用，ILoadBalancer的实现类维护- 个Timer每隔10s检测- -次Server的可用状态
4. IClientConfig主要定义了用于初始化各种客户端和负载均衡器的配置信息，器实现类为DefaultClientConfiglmpl

## 7、微服务熔断降级机制是什么

- 微服务框架是许多服务互相调用的，要是不做任何保护的话，某一个服务挂了，就会引|起连锁反应，导致别的服务也挂。

- Hystrix 是隔离、熔断以及降级的一个框架。它是基于aop的拦截机制，当拦截到请求异常的话：
  - 如果调用某服务报错(或者挂了)，就对该服务熔断，在5分钟内请求此服务直接就返回一个默认值，不需要每次都卡几秒，这个过程，就是所谓的熔断。
  - 但是熔断了之后就会少调用一个服务，此时需要做下标记，标记本来需要做什么业务,但是因为服务挂了，暂时没有做，等该服务恢复了,就可以手工处理这些业务。这个过程，就是所谓的降级。

```java
@Component
public class RemoteFileFallbackFactory implements FallbackFactory<RemoteFileService>
{
    private static final Logger log = LoggerFactory.getLogger(RemoteFileFallbackFactory.class);

    @Override
    public RemoteFileService create(Throwable throwable)
    {
        log.error("文件服务调用失败:{}", throwable.getMessage());
        return new RemoteFileService()
        {
            @PostMapping(value = "/upload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
            @Override
            public R<SysFile> upload(MultipartFile file)
            {
                return R.fail("上传文件失败:" + throwable.getMessage());
            }
        };
    }
}
```



## 8、服务网关（SpringCloud Gateway）

在微服务架构中，我们都会使用API网关来作为暴露服务的唯一出口。这样可以将与业务无关的各项控制，集中的在API网关中进行统一管理，从而使得业务服务可以更加专注于业务领域本身。

而在微服务构建的系统内部，各个服务之间的调度，我们通常采用注册中心和客户端负载均衡的方式来实现服务之间的调用。

- API网关通过注册中心发现所有后端服务，从来实现动态代理
- 后端服务集群间，通过注册中心互相发现对方，而实现直接调用（通常使用Ribbon、Feign这些框架）



## 9、微服务架构及注册中心eureka与nacos区别（startup.cmd -m standalone）

- nacos可以作为分布式注册中心，还可以作为分布式配置中心（SpringCloud config）
- nacos支持ap和cp模式切换。Eureka以cp为主
- Nacos与Eureka自我保护机制对比（如果Eureka Server在一定时间内（默认90秒）没有接收到某个微服务实例的心跳，Eureka Server将会移除该实例。）：
  - Eureka保护方式：当在短时间内，统计续约失败的比例，如果达到一定阈值，则会触发自我保护的机制，在该机制下，Eureka Server不会剔除任何的微服务（比如网络故障时，微服务其实是正常的，但是Eureka收不到信息，误以为是挂掉了；此时就先不要注销微服务），等到正常后，再退出自我保护机制。
  - Nacos保护方式：当域名健康实例 (Instance) 占总服务实例(Instance) 的比例小于阈值时，无论实例 (Instance) 是否健康，都会将这个实例 (Instance) 返回给客户端。
  - 范围不同：Nacos 的阈值是针对某个具体 Service 的，而不是针对所有服务的。但 Eureka的自我保护阈值是针对所有服务的。

## 10、Nacos（服务注册与发现），Feign（服务调用），Sentinel（熔断机制）关系图

![在这里插入图片描述](https://img-blog.csdnimg.cn/b7bc9570983447f79320250cd8033f28.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5p6X5aSn5Yi3,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

集成nacos和feign进行服务注册发现和调用！

## 11、 Skywalking链路追踪

- 由于微服务化项目拆分，会导致系统服务间调用链路愈发复杂，此时，一个前端请求可能最终需要调用多个后端服务才能完成实现。当整个请求不可用出现问题时，我们是没有办法判断请求是由哪个后端服务引发问题，这时我们需要快速定位故障点，找到调用异常的服务。 
- SkyWalking是中国人吴晟（华为）开源的一款APM工具，现在已属于Apache旗下开源项目, 是一个观察性分析平台和应用性能管理系统。提供分布式追踪、服务网格遥测分析、度量聚合和可视化一体化解决方案。
- 多种监控手段多语言自动探针，Java（把这个和IoT项目结合起来说）
- 支持告警
- 模块化，UI（很像nodered）、存储、集群管理多种机制可选

# 六、消息中间件

Kafka、ActiveMQ、RabbitMQ、RocketMQ。

## 1、消息中间件简介

支持在分布式系统中发送和接受消息的硬件或软件基础设施。**消息中间件就是用来解决分布式系统之间消息传递的问题。**它具有低耦合、可靠投递、广播、流量控制、最终一致性等一系列功能，成为异步RPC的主要手段之一。

### 1.1 典型使用场景

1. **系统解耦：**发布/订阅模式。即核心系统A生产核心数据，然后将核心数据发送到消息中间件，下游消费系统根据自身需求从中间件里获取消息进行消费，当不再需要数据时就不取消息进即可，这样系统之间耦合度就大大降低了。
2. **异步调用：**一个流程A——>B——>C——>D；当D非常慢时，就可以把它抽离出来，作一个异步调用。我们点一杯奶茶，下单、付款、通知商家制作都很快，然而到匹配外卖小哥配送这个过程很慢。作为用户来说，匹配外卖小哥这个过程延迟一些时间是可以接受的，只要我能快速下单成功，并且在一定时间范围内安排快递小哥送货即可。
3. **削峰填谷：**高并发中的流量控制。可以部署一层消息队列在机器前面，平时正常的每秒几百次请求，机器就正常的消费消息即可，一旦流量高峰到达时，大量消息会堆积在消息队列里面，机器只需要按照自己的最大负荷从消息队列里面消费，等流量高峰过了，慢慢地队列里面的消息也消费完毕了。

![img](https://upload-images.jianshu.io/upload_images/15986956-617b75d3a2cb24c6.png?imageMogr2/auto-orient/strip|imageView2/2/w/550/format/webp)

4. **日志处理：**日志处理是指将消息队列用在日志处理中，比如Kafka的应用，解决大量日志传输的问题。
5. **消息通讯：**以上实际是消息队列的两种消息模式，**点对点或发布订阅模式**。扩展一下，发布订阅中，消费者是既可以推（push）也可以拉（poll）的；在IoT里面，只有push，是因为能耗的问题，在大型应用程序中，没有能耗限制，用户服务是可以主动去消息队列中读信息的

当然，消息中间件的引入，无疑增加了系统的复杂度：

1. 在加入 MQ 之前，你不用考虑消息丢失或者说 MQ 挂掉等等的情况，但是，引入 MQ 之后你就需要去考虑了！（整体上）
2. 加入 MQ 之后，你需要保证消息没有被重复消费、处理消息丢失的情况、保证消息传递的顺序性等等问题！（功能上）
3. 一致性问题：中间件本身就是一个异步的过程（redis也一样），需要人去考虑消息一致性的问题。比如，万一消息的真正消费者并没有正确消费消息怎么办？怎么去弥补这个问题？

### 1.2 四种常见的消息中间件[对比](https://blog.csdn.net/u013521220/article/details/104352365)——Kafka、RabbitMQ、RocketMQ、ActiveMQ

| 对比方向 |                             概要                             |
| -------- | :----------------------------------------------------------: |
| 吞吐量   | 万级的 ActiveMQ 和 RabbitMQ 的吞吐量（ActiveMQ 的性能最差）要比 十万级甚至是百万级的 RocketMQ 和 Kafka 低一个数量级。 |
| 可用性   | 都可以实现高可用。ActiveMQ 和 RabbitMQ 都是基于主从架构实现高可用性。RocketMQ 基于分布式架构。 kafka 也是分布式的，一个数据多个副本，少数机器宕机，不会丢失数据，不会导致不可用 |
| 时效性   | RabbitMQ 基于 erlang 开发，所以并发能力很强，性能极其好，延时很低，达到微秒级。其他三个都是 ms 级。 |
| 功能支持 | 除了 Kafka，其他三个功能都较为完备。 Kafka 功能较为简单，主要支持简单的 MQ 功能，在大数据领域的实时计算以及日志采集被大规模使用，是事实上的标准 |
| 消息丢失 | ActiveMQ 和 RabbitMQ 丢失的可能性非常低， RocketMQ 和 Kafka 理论上不会丢失。 |

### 1.3 消息队列核心-如何保证消息不丢失

生产阶段：

- 生产阶段一般是通过confirm机制，producer把消息发送给broker，broker收到消息后会给客户端响应回执，producer收到回执则完成一次完整的消息发送。producer如果没有收到响应回执则会重发。

存储阶段

- 如果**Broker是单点**的，可以通过参数设置，当消息持久化后再给响应回执，如果是 Broker 是由多个节点组成的集群，需要将 **Broker 集群**配置成：至少将消息发送到 2 个以上的节点，再给客户端回复发送确认响应。这样当某个 Broker 宕机时，其他的 Broker 可以替代宕机的 Broker，也不会发生消息丢失

消费阶段：

- 消费阶段和生产阶段类似，都是通过confirm机制保障消息不丢失的，客户端从 Broker 拉取消息后，执行用户的消费业务逻辑，成功后，才会给 Broker 发送消费确认响应。如果 Broker 没有收到消费确认响应，下次拉消息的时候还会返回同一条消息，确保消息不会在网络传输过程中丢失，也不会因为客户端在执行消费逻辑中出错导致丢失。

## 2、Kafka

Kafka 是一个**分布式、流式处理**平台。

流平台具有三个关键功能：

1. **消息队列**：发布和订阅消息流，这个功能类似于消息队列，这也是 Kafka 也被归类为消息队列的原因。
2. **容错的持久方式存储记录消息流**： Kafka 会把消息持久化到磁盘，有效避免了消息丢失的风险。
3. **流式处理平台：** 在消息发布的时候进行处理，Kafka 提供了一个完整的流式处理类库。

Kafka 主要有两大应用场景：

1. **消息队列** ：建立实时流数据管道，以可靠地在系统或应用程序之间获取数据。
2. **数据处理：** 构建实时的流数据处理程序来转换或处理数据流。

 Kafka 相比其他消息队列主要的优势如下：

1. **极致的性能** ：基于 Scala 和 Java 语言开发，设计中大量使用了批量处理和异步的思想，最高可以每秒处理千万级别的消息。
2. **生态系统兼容性无可匹敌** ：Kafka 与周边生态系统的兼容性是最好的没有之一，尤其在大数据和流计算领域。

### 2.0 基本概念

> 参考：https://blog.csdn.net/a1774381324/article/details/125463522

#### 2.0.1 术语：

| **术语**                          | **术语**                   |
| --------------------------------- | -------------------------- |
| Topics（主题）                    | Kafka Cluster（Kafka集群） |
| Partition（分区）                 | Producers（生产者）        |
| Partition offset（分区偏移）      | Consumers（消费者）        |
| Replicas of partition（分区备份） | Leader（领导者）           |
| Brokers                           | Follower（追随者）         |

#### 2.0.2 发布/订阅的流程

1. 生产者向topic当中提交消息，Brokers将topic当中的数据在对应的分区当中依次保存；
2. 消费者向Brokers请求获取消息，Brokers向消费者提供偏移量，消费者根据偏移量要求获取消息。
3. 消费者排队的前提，消费者数量大于分区数量
4. 同一个消费者组内的消息不会重复消费。

#### 2.0.3 生产者和消费者之间的区别？

- 生产者：主要是消息提供者，根据业务需要往指定的topic推消息，一般也俗称为消息的上游。
- 消费者：
  - 要指定消费者的分组：默认情况下，分组是test
  - 消费者可以同时消费若干个topic：
    - 消息是已key-value格式进行发送
    - 每个key如果重复发送，其偏移量会递增
    - 新key的偏移量从0开始
    - 消费者要放在一个独立的线程当中，才能始终处于消费状态
  - Spring是没有办法直接给线程当中进行依赖注入的
  - 消费者的线程如果要通知其他的任务执行，需要从Spring的bean当中获取相关的业务对象

#### 2.0.4 kafka写消息的路由策略

1. 如果指定分区：直接使用分区进行路由
2. 指定了key，但是没有指定分区，那么会对key进行hash运算，通过运算的值得到一个分区
3. 如果都没指定，那么会轮询写入一个分区

#### 2.0.5 kafka写硬盘：

1. 传统写硬盘是随机写
2. kafka是顺序写硬盘，是随机写硬盘速度的6000倍
3. 写数据的流程
   1. 首先找到leader
   2. 将消息写入leader的日志文件
   3. Followers(包含ISR中的成员，也包含不在ISR中的成员)会同步leader当中的消息，同步完以后会向leader发送一个ACK确认。
   4. leader在接收到isr所有成员的ACK确认后，正式提交commit保存

#### 2.0.6 kafka的消息安全策略：

> 1. 默认是保证一定成功（同步）
> 2. 不重复发送，不保证成功（异步）

#### 2.0.7 kafka的备份：

1. 备份是由分区来创建的
2. 一个分区有1个leader和0-n个follower，只要leader不宕机，所有的follower都宕机了也不影响读写。follower只负责数据备份，不负责数据读写。

#### 2.0.8 Kafka的isr(容灾和判活机制)：

1. 同步备份：保证isr集合当中至少存活一个，如果leader不挂，正常提供服务，如果leader挂了，重新选leader然后提供服务；每个分区都有自己的isr备份的算法：

   1. 分区：分区编号，取余代理数量 （p_i mod b_num）

   2. 备份：分区编号 + 备份编号之和， 取余 代理数量（p_i+r_j） mod b_num

2. 判定存活：配置延时replica.log.max.messages，replica.log.time.max.ms来判定是否宕机
3. kafka如何解决zookeeper的压力的
   1. Kafka有容器机制
   2. 每一个代理会创建一个新的容器
   3. 容器负责维护leader的读写，和选举

4. 如果所有的ISR成员都死亡：
   1. 等待ISR成员任意一个苏醒，但是这个过程是不可控的
   2. 默认：只要有一个不是isr的成员存活，把这个作为新的leader。但是并不能保证这个成员是否数据和原本leader数据一致。

#### 2.0.9 topic的创建和删除流程：

- 创建topic，是首先获取代理的ids，然后将这些ids组成一个isr，作为一个新的容器

- 删除topic：

  1. 默认情况下delete.topic.enable=false；也就是被删除的节点会被移入zk的这个节点/admin/delete_topics

  2. 要彻底删除

     - delete.topic.enable=true：一旦删除，容器会清空在/admin/delete_topics节点上的监听

     - auto.create.topics.enable=false：自动创建主题，如果他为true，那么只要还有一个用户在往这个主题当中写消息，这个主题就不会真正被删除。即便是你已经删了，他依然还会创建一个出来。

#### 2.0.10 kafka的数据保存（日志）

1. Kafka的日志分为两种，一种是运行日志；还有一种是用于保存消息的日志；





###  2.1 Kafka 的消息模型知道吗？？

早期的 JMS 和 AMQP（当然，RabbitMQ提供了标准实现） 属于消息服务领域权威组织所做的相关的标准，但是这些标准的进化跟不上消息队列的演进速度，特别是在异步、分布式环境中。

他们（早期）采用的是消息队列模型：

1. 单队列：一条消息只能被一个消费者使用，未被消费的消息在队列中保留直到被消费或超时。 比如：我们生产者发送 100 条消息的话，两个消费者来消费一般情况下两个消费者会按照消息发送的顺序各自消费一半
2. 这存在很明显的问题，他只是进行了一个异步读取，但是效率并没有提起来。（甚至有点线程池的感觉）

**现在大家用的都是发布-订阅模型：（Kafka 中没有队列这个概念，与之对应的是 Partition（分区））**

- **几个核心角色：Producer、Consumer、Broker、Topic、Partition**
  - 比较难理解的是Partition：Partition(分区)是真正保存消息的地方。一个 Topic 可以有多个 Partition ，并且同一 Topic 下的 Partition 可以分布在不同的 Broker 上，这也就表明**一个 Topic 可以横跨多个 Broker 。**
  - Broker（代理） : 可以看作是一个独立的 Kafka 实例（所以他可以有多个，broker1、broker2）。多个 Kafka Broker 组成一个 Kafka Cluster。

PS：Kafka拥有持久化日志，这些日志可以被重复读取和无限期保留。这是它的一大优点！

### 2.2 Kafka 的多副本机制了解吗？带来了什么好处？（分布式的概念引入）

Kafka 为分区（Partition）引入了多副本（Replica）机制：

1. 分区（Partition）中的多个副本之间会有一个叫做 leader 的家伙，其他副本称为 follower。我们发送的消息会被发送到 leader 副本，然后 follower 副本才能从 leader 副本中**拉取**消息进行同步。（类比redis和mysql的主从机制）
2. 生产者和消费者只与 leader 副本交互。你可以理解为其他副本只是 leader 副本的拷贝，它们的存在只是为了保证消息存储的安全性。
3. leader出问题的时候，已经完成同步的follower可以重新选举一个新的leader出来

**Kafka 的多分区（Partition）以及多副本（Replica）机制优点：**

1. 多分区：Kafka 通过给特定 Topic 指定多个 Partition, 而各个 Partition 可以分布在不同的 Broker 上, 这样便能提供比较好的并发能力（负载均衡）。
2. Partition 可以指定对应的 Replica 数, 这也极大地提高了消息存储的安全性, 提高了容灾能力，不过也相应的增加了所需要的存储空间。

### 2.3 Zookeeper 在 Kafka 中的作用知道吗？（但是现在fakfa内部的分布式事务已经不用zookeeper了，自己实现了raft协议，更轻量化，避免了对zk的依赖）

ZooKeeper主要服务于分布式系统，Kafka主要使用ZooKeeper来**保存它的元数据、监控Broker和分区的存活状态，并利用ZooKeeper来进行选举**。（所以Zookeeper只在Kafka的分布式场景中才能使用，用于集群中不同节点之间通信）

1. Broker 注册 ：在 Zookeeper 上会有一个专门用来进行 Broker 服务器列表记录的节点。每个 Broker 在启动时，都会到 Zookeeper 上进行注册，即到 /brokers/ids 下创建属于自己的节点。每个 Broker 就会将自己的 IP 地址和端口等信息记录到该节点中去
2. Topic 注册 ： 在 Kafka 中，同一个Topic 的消息会被分成多个分区并将其分布在多个 Broker 上，这些分区信息及与 Broker 的对应关系也都是由 Zookeeper 在维护。比如我创建了一个名字为 my-topic 的主题并且它有两个分区，对应到 zookeeper 中会创建这些文件夹：`/brokers/topics/my-topic/Partitions/0`、`/brokers/topics/my-topic/Partitions/1`（类似于文件系统的Znode）
3. **负载均衡** ：上面也说过了 Kafka 通过给特定 Topic 指定多个 Partition, 而各个 Partition 可以分布在不同的 Broker 上, 这样便能提供比较好的并发能力。 对于同一个 Topic 的不同 Partition，Kafka 会尽力将这些 Partition 分布到不同的 Broker 服务器上。当生产者产生消息后也会尽量投递到不同 Broker 的 Partition 里面。当 Consumer 消费的时候，Zookeeper 可以根据当前的 Partition 数量以及 Consumer 数量来实现动态负载均衡。

#### 2.3.1 Kafka客户端如何找到对应的Broker？

直接回答问题：

- 流程：
  - 先根据主题和队列，在右边的树中找到分区对应的state临时节点，state节点中保存了这个分区Leader的BrokerID。
  - 拿到这个Leader的BrokerID后，再去左侧的树中，找到BrokerID对应的临时节点，就可以获取到Broker真正的访问地址了

其实这个问题的核心是，**Zookeeper的元数据缓存是什么东西？**

1. Kafka的客户端并不会直接连接ZooKeeper，它只会和Broker进行远程通信。ZooKeeper上的元数据是通过Broker中转给每个客户端的，在需要的时候，通过RPC请求去Broker上拉取它关心的主题的元数据，然后保存到客户端的元数据缓存中，以便支撑客户端生产和消费
2. Kafka在每个Broker中都维护了一份和ZooKeeper中一样的元数据缓存，并不是每次客户端请求元数据都去读一次ZooKeeper。（依赖于Zookeeper的Watcher监控机制，Kafka可以感知到ZK的元数据变化情况，并及时更新）

### 2.4、kafka如何保证信息不丢失？（参考上面1.3）

这里有三种情况，生产者丢失消息、消费者丢失消息、中间件丢失消息。

生产者丢失消息：

1. 问题：生产者(Producer) 调用send方法发送消息之后，消息可能因为网络问题并没有发送过去。
2. 解决：通过 get()方法获取调用结果（不推荐，多了一步，相当于变成了同步操作，发完要确认）；为 Producer 的retries （重试次数）设置一个比较合理的值，一般是 3 ，但是为了保证消息不丢失的话一般会设置比较大一点。设置完成之后，当出现网络问题之后能够自动重试消息发送，避免消息丢失。

消费者丢失消息：

1. 问题：当消费者拉取到了分区的某个消息之后，消费者会自动提交 offset。但是当消费者去操作这个消息的时候，突然挂掉了，消息实际上并没有被消费，但是 offset 却被自动提交了。
2. 解决：手动关闭自动提交 offset，每次在真正消费完消息之后再自己手动提交 offset 。（重复消费的问题。比如消费完消息之后，还没提交 offset，结果自己挂掉了，那么这个消息理论上就会被消费两次。）——在后面的`4.重复消费`有讲

中间件丢失消息：

1. 问题：假如 leader 副本所在的 broker 突然挂掉，那么就要从 follower 副本重新选出一个 leader ，但是 leader 的数据还有一些没有被 follower 副本的同步的话，就会造成消息丢失。
2. 解决：（核心就是冗余备份）
   - 配置 acks = all，代表则所有副本都要接收到该消息之后该消息才算真正成功被发送。
   - 为 topic 设置 replication.factor >= 3。这样就可以保证每个 分区(partition) 至少有 3 个副本。虽然造成了数据冗余，但是带来了数据的安全性。
   - 设置 min.insync.replicas> 1 ，这样配置代表消息至少要被写入到 2 个副本才算是被成功发送。min.insync.replicas 的默认值为 1 ，在实际生产中应尽量避免默认值 1。

### 2.5 Kafka读写策略——主写主读？存储不分离？

> 不支持读写分离？有时候这个问题的本意是上面的**主写主读**概念，认为把读和写放在不同的机器上，这个叫做“读写分离”。

在 Kafka 中，生产者写入消息、消费者读取消息的操作都是与 leader 副本进行交互的，从而实现的是一种**主写主读**的生产消费模型。（<font color='red'>所以kafka集群的产生**主要**是为了解决分布式的问题，并不是为了应对高并发读的情形，或者说也不是为了负载均衡</font>）

Kafka 并不支持主写从读，因为主写从读有 2 个很明显的缺点:

- **数据一致性问题**。数据从主节点转到从节点必然会有一个延时的时间窗口，这个时间 窗口会导致主从节点之间的数据不一致。某一时刻，在主节点和从节点中 A 数据的值都为 X， 之后将主节点中 A 的值修改为 Y，那么在这个变更通知到从节点之前，应用读取从节点中的 A 数据的值并不为最新的 Y，由此便产生了数据不一致的问题。
- **延时问题**。类似 Redis 这种组件，数据从写入主节点到同步至从节点中的过程需要经历`网络→主节点内存→网络→从节点内存`这几个阶段，整个过程会耗费一定的时间。**而在 Kafka 中，主从同步会比 Redis 更加耗时，**它需要经历`网络→主节点内存→主节点磁盘→网络→从节点内存→从节点磁盘`这几个阶段。对延时敏感的应用而言，主写从读的功能并不太适用。

#### 2.5.1 Kafka 是否有必要支持主写从读的功能呢?

> 看一下kafka的生产者/消费者模型与主从broker的关系

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220923105911.png)

1. 在 Kafka 集群中有 3 个分区，每个分区有 3 个副本，正好均匀地分布在 3个 broker 上，灰色阴影的代表 leader 副本，非灰色阴影的代表 follower 副本，虚线表示 follower 副本从 leader 副本上拉取消息。
2. 当生产者写入消息的时候都写入 leader 副本，对于上图中的情形，每个 broker 都有消息从生产者流入;
3. 当消费者读取消息的时候也是从 leader 副本中读取 的，对于上面的情形，每个 broker 都有消息流出到消费者。

我们很明显地可以看出，每个 broker上的读写负载都是一样的，这就说明 **Kafka 通过主写主读实现了和主写从读不一样的负载均衡。**

有以下几种 情况(包含但不仅限于)会造成一定程度上的负载不均衡：

- **broker 端的分区分配不均**。当创建主题的时候可能会出现某些 broker 分配到的分区数 多而其他 broker 分配到的分区数少，那么自然而然地分配到的 leader 副本也就不均。
  - Kafka 中相应的分配算法，在主题创建的时候尽可能使分区分配得均衡
    - **RangeAssignor 分配策略**：按照**消费者**总数和分区总数进行整除运算来获得一个跨度，然后分区按照跨度来进行平均分配，尽可能保证分区均匀的分配给所有的消费者。**对于每个 topic，**该策略会讲消费者组内所有订阅这个主题的消费者按照名称的字典顺序排序，然后为每个消费者划分固定过的区域，如果不够平均分配，那么字典排序考前的就会多分配一个分区。
    - **RoundRobinAssignor 分配策略**：按将消费者组内所有消费者及消费者订阅的所有主题的分区按照字典排序，然后通过轮询的方式分配给每个消费者。
- **生产者写入消息不均**。生产者可能只对某些 broker 中的 leader 副本进行大量的写入操 作，而对其他 broker 中的 leader 副本不闻不问。
  - 主写从读也无法解决
- **消费者消费消息不均**。消费者可能只对某些 broker 中的 leader 副本进行大量的拉取操 作，而对其他 broker 中的 leader 副本不闻不问。
  - 主写从读也无法解决
- **leader 副本的切换不均**。在实际应用中可能会由于 broker 宕机而造成主从副本的切换， 或者分区副本的重分配等，这些动作都有可能造成各个 broker 中 leader 副本的分配不均。
  - Kafka 提供了**优先副本**的选举来达到 leader 副本的均衡（就是broker中会维护一个副本的优先级），与此同时，也可以配合相应的 监控、告警和运维平台来实现均衡的优化。

#### 2.5.2 为什么kafka不支持储存-计算分离？

>kafka有非常成熟的本地数据保存方案，分区保证数据一致性的问题
>
>然后还有一块就是云原生带来的问题。**存储和计算分离是现代比较流行的操作，**其影响比较大的有两块，**一个是数据库，另外一个是消息队列**，接下来我会具体讲下这两块到底是怎么利用“计算和存储分离”的。
>
>不论是Kafka还是RocketMQ其设计思想都是利用本地机器的磁盘来进行保存消息队列，这样其实是有一定的弊端的：
>
>- 数据有限，使用者两个消息队列的同学应该深有感触，一般会服务器保存最近几天的消息，这样的目的是节约存储空间，但是就会导致我们要追溯一些历史数据的时候就会导致无法查询。
>
>- 扩展成本高，在数据库中的弊端在这里同样也会展现。（全量复制迁移的问题）

在Pulsar的架构中，数据计算和数据存储是单独的两个结构：

- 数据计算也就是Broker，其作用和Kafka的Broker类似，用于负载均衡，处理consumer和producer等，如果业务上consumer和producer特别的多，我们可以单独扩展这一层。

- 数据存储也就是Bookie，pulsar使用了Apache Bookkeeper存储系统，并没有过多的关心存储细节，这一点其实我们也可以借鉴参考，当设计这样的一个系统的时候，计算服务的细节我们需要自己多去思考设计，而存储系统可以使用比较成熟的开源方案。


Pulsar理论上来说存储是无限的，我们的消息可以永久保存，有人会说难道硬盘不要钱吗？当然不是我们依然要钱，在**Pulsar**可以进行**分层存储**，我们将旧的消息移到便宜的存储方案中，比如AWS的s3存储，而我们当前最新的消息依然在我们比较贵的SSD上。在这个模式下不仅是存储是无限，我们的计算资源扩展也是无限的，因为我们的计算资源基本上是无状态的，扩展是没有任何成本的，所以Pulsar也搞出了一个多租户的功能,而不用每个团队单独去建立一个集群，之前在美团的确也是这样的，比较重要的BG基本上都有自己的Mafka集群，防止互相影响。

### 2.6 Kafka 放弃 ZooKeeper

> Broker 是 Kafka 集群的骨干，负责从生产者（producer）到消费者（consumer）的接收、存储和发送消息。
>
> 在当前架构下，
>
> 1. **Kafka 进程在启动的时候需要往 ZooKeeper 集群中注册一些信息，**比如 BrokerId，并组建集群。
> 2. **ZooKeeper 为 Kafka 提供了可靠的元数据存储，**比如 Topic/分区的元数据、Broker 数据、ACL 信息等等。
> 3. 同时 ZooKeeper 充当 Kafka 的领导者，以更新集群中的拓扑更改，比如说扩容、分区迁移等等。
>
> 这就相当于，云心一个分布式中间件集群，还要依赖另一个大型项目做分布式管理。分区数增加，ZooKeeper 上需要存储的元数据就会增加，从而加大 ZooKeeper 的负载，给 ZooKeeper 集群带来压力，可能导致 Watch 的延时或丢失。

怎么实现新的架构（最重要的一点，就是拜托选主机制对于zookeeper的依赖）：

1. Quorum 控制器使用新的 KRaft 协议来确保元数据在仲裁中被精确地复制。这个协议在很多方面与 ZooKeeper 的 ZAB 协议和 Raft 相似。这意味着，仲裁控制器在成为活动状态之前不需要从 ZooKeeper 加载状态。当领导权发生变化时，新的活动控制器已经在内存中拥有所有提交的元数据记录。

2. 在架构改进之前，一个最小的分布式 Kafka 集群也需要六个异构的节点：三个 ZooKeeper 节点，三个 Kafka 节点。而一个最简单的 Quickstart 演示也需要先启动一个 ZooKeeper 进程，然后再启动一个 Kafka 进程。在新的 KIP-500 版本中，一个分布式 Kafka 集群只需要三个节点，而 Quickstart 演示只需要一个 Kafka 进程就可以。

   

## 3、消息中间件如何保证消息的消费顺序？不同中间件的实现（避免重复消费的问题）

⽹上更多的都是介绍binlog的同步，好像更多的场景就没了。但是我们现在是消息中间件部分（似乎redis没有顺序的概念？因为redis是单线程，处理完就走的那种）

问题很明确：对于大批量的事务，当他们按一定顺序执行，才能有效果（这里的核心不是异步，而是同步）。比如，在数据库同时对⼀个Id的
数据进⾏了增、改、删三个操作，但是你消息发过去消费的时候变成了改，删、增，这样数据就不对了。（①更改用户会员等级 ②根据会员等级计算订单价格。这两个顺序错了就会出现问题）

### 3.1 Kafka是用的每次添加消息到 Partition(分区) 的时候都会采用尾加法

注意：Kafka 中 Partition(分区)是真正保存消息的地方，我们发送的消息都被放在了这里。所以，**Kafka 只能为我们保证 Partition(分区) 中的消息有序。**

1. 消息在被追加到 Partition(分区)的时候都会分配一个特定的偏移量（offset）。Kafka 通过**偏移量（offset）**来保证消息在分区内的顺序性。

但是，**一个topic下有多个partition，我们怎么保证呢？**（总不能1 个 Topic 只对应一个 Partition吧，这虽然可以，但是极大地限制了中间件的作用）

- Kafka规定了发送 1 条消息的时候，可以指定 topic, partition, key,data（数据） 4 个参数。如果你发送消息的时候指定了 Partition 的话，所有消息都会被发送到指定的 Partition。并且，同一个 key 的消息可以保证只发送到同一个 partition，这个我们可以采用表/对象的 id 来作为 key 。

### 3.2 RocketMQ提供了MessageQueueSelector队列选择机制？

RocketMQ使用的是队列模型。⼀个topic下有多个队列，为了保证发送有序，RocketMQ提供了MessageQueueSelector队列选择机制。具体而言，对应了三个方法：`SelectMessageQueueByhash、SelectMessageQueueByMachineRoom、SelectMessageQueueByRandom`

1. 让同⼀个订单发送到同⼀个队列中，再使⽤同步发送，只有同个订单的创建消息发送成功，再发送⽀付消息。这样，我们保证了发送有序。
2. RocketMQ的topic内的队列机制,可以保证存储满⾜FIFO,剩下的只需要消费者顺序消费即可。
3. 但是，RocketMQ仅保证顺序发送，顺序消费由消费者业务保证!!!（当消费者是多线程的情况下，你消息是有序的给他们的，你能保证他是有序的处理的？还是⼀个消费成功了再发下⼀个**稳妥**。）



## 4、消息中间件如何保证消息不会被重复消费？

我们要有这样的概念，就是中间件的引入只是为了加快速度，他当然也有自己的鲁棒性设计，但是还是会有一些问题没有解决，这是他属性决定的（就像数据库和redis的一致性问题，他就是不能做到强一致性，因为破坏了中间件的核心）。**具体的设计要根据具体的业务场景去选择！**

先看一下场景：对于一个消息，`用户支付成功`，对应地，有多个服务都要修改，比如`用户积分、库存、优惠券、活动系统`。如果用户积分增加的系统出错了，最直观的操作就是要求中间件重发一下。问题出现，别的服务也在监听（订阅了支付成功的事件），你重发一下，别人会以为又来一个新的服务，这就造成消息被重复消费了。

核心——幂等性：同一个函数⽆论多次执⾏，其结果都是⼀样的。

### 4.1 强校验和弱校验去实现幂等性

强校验（强一致性）：

- 把`用户积分、库存、优惠券、活动系统`都放到一个事务中，成功⼀起成功失败⼀起失败。（基于RabbitMQ的2PC就是这么个实现）
- 建⽴⼀个消息表，拿到这个消息做数据库的insert操作。给这个消息做⼀个唯⼀主键（primary key）或者唯⼀约束，那么就算出现重复消费的情况，就会导致主键冲突，那么就不再处理这条消息。（RocketMQ就是这样去重的）

弱校验：

- ⼀些不重要的场景，⽐如给谁发短信啥的，我就把这个id+场景唯⼀标识作为Redis的key，放到缓存。大家都去redis中查找，有没有这条服务，如果已经有了，就直接return不要⾛下⾯的流程了，没有就执⾏后⾯的逻辑。

可以看一下kakfa是怎么解决的↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓

### 4.2 Kafka解决重复消费消息——offset

**kafka出现消息重复消费的原因：**

- 服务端侧已经消费的数据没有成功提交 offset（根本原因）。
  - 场景复现：Kafka 侧 由于服务端处理业务时间长或者网络链接等等原因让 Kafka 认为服务假死，触发了分区 rebalance。


**解决方案：**

- 消费消息服务做幂等校验，比如 Redis 的set、MySQL 的主键等天然的幂等功能。这种方法最有效。

- 将`enable.auto.commit`参数设置为 false，关闭自动提交，开发者在代码中手动提交 offset。那么这里会有个问题：

  **什么时候提交offset合适？**

  - 处理完消息再提交：依旧有消息重复消费的风险，和自动提交一样
  - 拉取到消息即提交：会有消息丢失的风险（因为服务端数据并不能保证数据已经被成功处理，只能保证曾经获取到）。允许消息延时的场景，一般会采用这种方式。然后，通过定时任务在业务不繁忙（比如凌晨）的时候做数据兜底。

## 5、消息中间件怎么保证数据不丢失的？

### 5.1、 为何消息会丢失？

要想保证消息只被消费一次，那么首先就得要保证消息不丢失。我们先来看看，消息从被写入消息队列，到被消费完成，这整个链路上会有哪些地方可能会导致消息丢失？我们不难看出，其实主要有三个地方：

- 消息从生产者到消息队列的过程。
- 消息在消息队列存储的过程。
- 消息在被消费的过程。

#### 5.1.1 消息在写到消息队列的过程中丢失

消息生产者一般就是业务系统，消息队列是单独部署了在独立的服务器上的，所以业务服务器和消息队列服务器可能会出现网络抖动，当出现了网络抖动，消息就会丢失。

一般这种情况，我们可以采用消息重传的方案，即当我们发现发送的消息超时后，我们就重新发送一次，但是不能一直无限制的重传消息。按照经验来说，如果不是消息队列本身故障，或者是网络断开了，一般重试个 2 到 3 次就行了。

但是，这种方案就有可能造成消息的重复，这样就会导致消费者消费到重复的消息。例如，消息发送到消息队列中，但是由于消息队列处理消息较慢或者网络抖动，这个时候，其实消息是写入成功的，但是对于生产端就认为超时了，那么生产者就会重传当前消息，则会出现消息重复。对于我们上面案例中，就是用户会收到两个红包。

#### 5.1.2 消息在消息队列中丢失

> 那就从消息队列本身的容灾性去考虑

即使消息发送到了消息队列，消息也不会万无一失，还是会面临丢失的风险。

我们以 Kafka 为例，消息在Kafka 中是存储在本地磁盘上的， 为了减少消息存储对磁盘的随机 I/O，

一般我们会将消息写入到操作系统的 Page Cache 中，然后在合适的时间将消息刷新到磁盘上。

例如，Kafka 可以配置当达到某一时间间隔，或者累积一定的消息数量的时候再刷盘，也就是所谓的**异步刷盘**。

不过，如果发生机器掉电或者机器异常重启，那么 Page Cache 中还没有来得及刷盘的消息就会丢失了。那么怎么解决呢？你可能会把刷盘的间隔设置很短，或者设置累积一条消息就就刷盘，但这样频繁刷盘会对性能有比较大的影响，而且从经验来看，出现机器宕机或者掉电的几率也不高，所以我不建议你这样做。

如果你的电商系统对消息丢失的容忍度很低，那么你可以考虑**以集群方式部署 Kafka 服务，通过部署多个副本备份数据，保证消息尽量不丢失**。

**那么它是怎么实现的呢**？Kafka 集群中有一个 Leader 负责消息的写入和消费，可以有多个 Follower 负责数据的备份。Follower 中有一个特殊的集合叫做 ISR（in-sync replicas），当 Leader 故障时，新选举出来的 Leader 会从 ISR 中选择，默认 Leader 的数据会异步地复制给 Follower，这样在 Leader 发生掉电或者宕机时，Kafka 会从 Follower 中消费消息，减少消息丢失的可能。

由于默认消息是异步地从 Leader 复制到 Follower 的，所以一旦 Leader 宕机，那些还没有来得及复制到 Follower 的消息还是会丢失。为了解决这个问题，Kafka 为生产者提供一个选项叫做“acks”，当这个选项被设置为“all”时，生产者发送的每一条消息除了发给 Leader 外还会发给所有的 ISR，并且必须得到 Leader 和所有 ISR 的确认后才被认为发送成功。这样，只有 Leader 和所有的 ISR 都挂了，消息才会丢失。

#### 5.1.3 在消费的过程中存在消息丢失的可能

> 这里其实比较难解决，让消费者确认自己使用完了数据在确认的方式，对性能影响较大，加上这里是非关键链路，其实不太好做

这里面接收消息和处理消息的过程都可能会发生异常或者失败，比如说，消息接收时网络发生抖动，导致消息并没有被正确的接收到；处理消息时可能发生一些业务的异常导致处理流程未执行完成，这时如果更新消费进度，那么这条失败的消息就永远不会被处理了，也可以认为是丢失了。

**所以，**在这里你需要注意的是，一定要等到消息接收和处理完成后才能更新消费进度，但是这也会造成消息重复的问题，比方说某一条消息在处理之后，消费者恰好宕机了，那么因为没有更新消费进度，所以当这个消费者重启之后，还会重复地消费这条消息。

**为了避免消息丢失，我们需要付出两方面的代价：一方面是性能的损耗；一方面可能造成消息重复消费。**

为了解决这个问题，就是要去解决重复消费，一般可以通过分布式事务，或者版本号控制的方法。当然还有一些系统化的设计，比如上面的`消息中间件如何保证消息不会被重复消费`。（本质上就是去保证消息的幂等性）



# 七、容器——Docker&K8s

Docker是一个容器化~~平台~~（其实更准确地说，它是一个引擎），它以容器的形式将您的应用程序及其所有依赖项打包在一起，以确保您的应用程序在任何环境中无缝运行。

## 1、Docker与虚拟机有何不同

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220324154912.png)

- 虚拟机需要模拟整个硬件，运行整个操作系统（完全隔离），体积和内存占用都很高。Docker不会模拟硬件，只为每一个应用提供安全隔离的运行环境。

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220324155525.png)

- docker底层技术：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220324155653.png)

## 2、docker

**dockerfile**

- 自动化脚本

**image（镜像）**

- 相当于虚拟机的快照，里面包含了要部署的应用程序已经它所关联的所有库，通过镜像，可以创建不同的Container（就像类和实例的关系）

**Container**

- 里面运行了应用程序，每个容器是独立运行的。

**常见命令：**

- docker pull:拉取或者更新指定镜像；
- docker push:将镜像推送至远程仓库；
- docker images:列出所有镜像；
- docker rmi:删除镜像； 
-  docker ps:列出所有容器；
- docker rm:删除容器

**使用流程：**

- 创建Dockerfile后，docker build创建容器的镜像；
- 推送或拉取镜像

### 2.1、docker-compose

使用 Docker Compose 可以轻松、高效的管理容器，它是一个用于定义和运行多容器 Docker 的应用程序工具。

- docker-compose.yml文件

```yml
version: "3"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]
```

在 docker-compose.yml 所在路径下执行该命令 Compose 就会自动构建镜像并使用镜像启动容器。

```
docker-compose up
docker-compose up -d  // 后台启动并运行容器
```

### 2.2、Docker Volume持久化

Docker的数据持久化主要有两种方式：

- bind mount
- volume

Docker的数据持久化即使数据不随着container的结束而结束，数据存在于host机器上——要么存在于host的某个指定目录中（使用bind mount），要么使用docker自己管理的volume（/var/lib/docker/volumes下）。

- **bind mount**

bind mount自docker早期便开始为人们使用了，用于将host机器的目录mount到container中。但是bind mount在不同的宿主机系统时不可移植的，比如Windows和Linux的目录结构是不一样的，bind mount所指向的host目录也不能一样。这也是为什么bind mount不能出现在Dockerfile中的原因，因为这样Dockerfile就不可移植了。

- **volume**

volume也是绕过container的文件系统，直接将数据写到host机器上，只是volume是被docker管理的，docker下所有的volume都在host机器上的指定目录下/var/lib/docker/volumes。





## 3、Docker和K8s的区别？

不是一个层面的东西！K8s用来解决分布式场景中，多节点协作的问题。原本docker中每个应用是在各个containers中，而这些containers还是在同一个机器上。k8s则是要把这些containers部署到不同的节点服务器上。

# 八、RPC框架

> RPC是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。RPC协议假定某些传输协议的存在，如TCP或UDP，为通信程序之间携带信息数据。在OSI网络通信模型中，**RPC跨越了传输层和应用层**。RPC使得开发包括网络分布式多程序在内的应用程序更加容易。
>
> 
>
> 注意一点，一般说RPC框架，那就说明你的项目是一个同步项目；如果是异步的话，一般就会问消息中间件；再其次，如果用到的是微服务的架构，那就要加入微服务的一些组件（限流、网关服务、负载均衡等）。

RPC（Remote Procedure Call Protocol）远程过程调用协议。一开始的理解就是**序列化和反序列化**，但是这是比较粗浅的。他背后蕴含的技术，是计算机网络协议+服务调用。

## 1、什么是RPC？

RPC是指远程过程调用，也就是说两台服务器A，B，一个应用部署在A服务器上，想要调用B服务器上应用提供的函数/方法，由于不在一个内存空间，不能直接调用，需要通过网络来表达调用的语义和传达调用的数据。

是一种进程间通信方式，他是一种**技术的思想，**而不是规范。

RPC两个核心模块：**通讯，序列化**。

![](https://img-blog.csdnimg.cn/20190825095214999.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODcxNzM0,size_16,color_FFFFFF,t_70)

- 首先，要**解决通讯**的问题：在客户端和服务器之间建立TCP连接，远程过程调用的所有交换的数据都在这个连接里传输。
- 第二，要**解决寻址**的问题：A服务器上的应用怎么告诉底层的RPC框架，如何连接到B服务器（如主机或IP地址）以及特定的端口，方法的名称名称是什么，这样才能完成调用。比如基于Web服务协议栈的RPC，就要提供一个endpoint URI，或者是从UDDI服务上查找。
- 第三，**参数序列化**：由于网络协议是基于二进制的，内存中的参数的值要序列化成二进制的形式，也就是序列化（Serialize）或编组（marshal），通过寻址和传输将序列化的二进制发送给B服务器。
- 第四，**反序列化**：对参数进行反序列化（序列化的逆操作），恢复为内存中的表达方式，然后找到对应的方法（寻址的一部分）进行本地调用，然后得到返回值。

## 2、RPC的应用场景

一开始，我就是理解为远程服务调用。偶然看到一个朋友，他做了一个项目，就是基于rpc框架的跨语言调用。发现目前RPC有两种路线：

1. **为了跨语言，**服务端可以用不同的语言实现，客户端也可以用不同的语言实现，不同的语言实现的客户端和服务器端可以互相调用。很显然，要支持不同的语言，需要基于那种语言实现相同协议的框架，并且协议设计应该也是跨语言的，其中比较典型的是 grpc,基于同一个IDL，可以生成不同语言的代码，并且语言的支持也非常的多。
2. **为了服务治理，**主要的精力放在服务发现、路由、容错处理等方面，主要围绕一个语言开发，可能也有一些第三方曲折的实现服务的调用和服务的实现，这其中的代表，也是比较早的开源的框架就是阿里巴巴的dubbo。**（这一部分是我在项目中使用的，我用到的组件是Feign）**。有些restful风格的rpc框架天然支持API gateway进行负载均衡。
   - **这里可以展开两个层级的问题（优化，高并发！）：**最简单的Feign是怎么设计的？它的流程是什么？那你知道怎么设计一个能够应对高并发场景的RPC框架吗（利用Feign）？(参考上文的`Feign原理简述`)

## 3、RPC定义

一个通俗的描述是：客户端在不知道调用细节的情况下，调用存在于远程计算机上的某个对象，就像**调用本地应用程序中的对象**一样。比较正式的描述是：一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。

我们可以注意到这么几个点：

1. RPC是协议（规范）：目前典型的RPC实现包括：Dubbo、Thrift、GRPC、Hetty等。实现了RPC协议的应用工具往往都会附加其他重要功能，例如Dubbo还包括了服务治等功能。
2. 网络协议和网络I/O模型对其透明：既然RPC的客户端认为自己是在调用本地对象。那么传输层使用的是TCP/UDP还是HTTP协议，又或者是一些其他的网络协议它就不需要关心了。
3. 信息格式对其透明：在本地应用程序中，对于某个对象的调用需要传递一些参数，并且会返回一个调用结果。至于被调用的对象内部是如何使用这些参数，并计算出处理结果的，调用方是不需要关心的。对于RPC而言，也是这样的一种情况，不需要关心参数是怎么传递的，只要协调好，要传入的参数就行了。（这里可以针对面试官的问题：“你的项目中需要涉及到项目分工的问题吗？”带一下，只用带一下就行了）
4. **应该有跨语言能力：**调用方实际上也不清楚远程服务器的应用程序是使用什么语言运行的。（这就是RPC的应用部分，跨语言的服务调用）

## 4、RPC主要组成部分

虽然说，客户不需要知道这么多的内部，那是因为它内部已经做好了封装，这就是封装的好处，只用对外提供API。

![img](https://img-blog.csdnimg.cn/img_convert/13b62a1239ebc5d13d3a6c0ad6ee2dc4.png)

- Client：RPC协议的调用方
- Server：在RPC规范中，这个Server并不是提供RPC服务器IP、端口监听的模块。而是远程服务方法的具体实现（在JAVA中就是RPC服务接口的具体实现）。其中的代码是最普通的和业务相关的代码，甚至其接口实现类本身都不知道将被某一个RPC远程客户端调用。（**这个很重要，Server只是个代码**）
- Stub/Proxy：RPC代理存在于客户端，因为要实现客户端对RPC框架“透明”调用，那么客户端不可能自行去管理消息格式、不可能自己去管理网络传输协议，也不可能自己去判断调用过程是否有异常。
- Message Protocol：在上文我们已经说到，一次完整的client-server的交互肯定是携带某种两端都能识别的，共同约定的消息格式。目前流行的技术趋势是不同的RPC实现，为了加强自身框架的效率都有一套（或者几套）私有的消息格式。
- Transfer/Network Protocol：传输协议层负责管理RPC框架所使用的网络协议、网络IO模型。例如Hessian的传输协议基于HTTP（应用层协议）；而Thrift的传输协议基于TCP（传输层协议）。传输层还需要统一RPC客户端和RPC服务端所使用的IO模型；
- **Selector/Processor：**存在于RPC服务端，用于服务器端某一个RPC接口的实现的特性（它并不知道自己是一个将要被RPC提供给第三方系统调用的服务）。所以在RPC框架中应该有一种“负责执行RPC接口实现”的角色。包括：管理RPC接口的注册、判断客户端的请求权限、控制接口实现类的执行在内的各种工作。**<font color='red'>（Feign！！！！！）</font>**
- IDL：实际上IDL（接口定义语言）并不是RPC实现中所必须的。但是需要跨语言的RPC框架一定会有IDL部分的存在。

## 5、影响RPC框架性能的因素

几个因素，也就是针对RPC框架的各个部分的优化！

- **使用的网络IO模型：**RPC服务器可以只支持传统的阻塞式同步IO，也可以做一些改进让RPC服务器支持非阻塞式同步IO，或者在服务器上实现对多路IO模型的支持。这样的RPC服务器的性能在高并发状态下，会有很大的差别。
- 基于的网络协议：一般来说您可以选择让您的RPC使用应用层协议，例如HTTP或者HTTP/2协议，或者使用TCP协议，让您的RPC框架工作在传输层。(没有采用UDP协议做为主要的传输协议的)
- 消息封装格式：选择或者定义一种消息格式的封装，要考虑的问题包括：消息的易读性、描述单位内容时的消息体大小、编码难度、解码难度、解决半包/粘包问题的难易度。
- **Schema 和序列化**（Schema & Data Serialization）：序列化和反序列化，是对象到二进制数据的转换，程序是可以理解对象的，对象一般含有 schema 或者结构，基于这些语义来做特定的业务逻辑处理。需要考虑以下几点：
  - 编码格式，是不是能直观看懂的，比如Json/protobuf，还是binary格式
  - 新老契约的兼容性 。比如 IDL 加了一个字段，老数据是否还可以反序列化成功。
  - 压缩算法的契合度，例如Gzip
  - **序列化、反序列化的时间，**序列化后数据的字节大小是考察重点。（常见的有 Protocol Buffers， Avro，Thrift，XML，JSON）——项目中用的是JSON序列化（Redis使用FastJson序列化）、（JsonFormat，去定义请求方法，操作类别，参数，错误信息等，`class R<T> implements Serializable`，响应主体实现了序列化的方法）
- **实现的服务处理管理方式：**在高并发请求下，如何管理注册的服务也是一个性能影响点。（<font color='red'>参考上文中`五、微服务 - 5.5 基于springcloud异步线程池、高并发请求feign的解决方案`</font>）
  - 可以让RPC的Selector/Processor使用单个线程运行服务的具体实现（这意味着上一个客户端的请求没有处理完，下一个客户端的请求就需要等待）——少
  - 也可以为每一个RPC具体服务的实现开启一个独立的线程运行（可以一次处理多个请求，但是操作系统对于“可运行的最大线程数”是有限制的）——目前的项目
  - 也可以线程池来运行RPC具体的服务实现（目前看来，在单个服务节点的情况下，这种方式是比较好的）——优化的项目
  - 还可以通过注册代理的方式让多个服务节点来运行具体的RPC服务实现。——多个服务器的情况，用不上。
    



# PS：一些伴随问题

## 2、分布式系统的演变？

- **单一应用架构：**当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简化增删改查工作量的数据访问框架(ORM)是关键。
- **垂直应用架构：**当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的几个应用，以提升效率。通过切分业务来实现各个模块独立部署，降低了维护和部署的难度，团队各司其职更易管理，性能扩展也更方便，更有针对性。（但是很多共用模块无法重复利用）
- **分布式服务架构：**将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。此时，用于提高业务复用及整合的分布式服务框架(RPC)是关键。
- **流动计算架构：**当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。此时，用于提高机器利用率的**资源调度和治理中心(SOA)**[ Service Oriented Architecture]是关键。

## 4、前后端分离？

- **核心思想**是前端html页面通过ajax调用后端的restuful api接口并使用json数据进行交互。

传统的JSP设计方法：在java后端都是分了三层（就是MVC思想），控制层（controller/action），业务层（service/manage），持久层（dao）。控制层负责接收参数，调用相关业务层，封装数据，以及路由&渲染到jsp页面。**然后jsp页面上使用各种标签（jstl/el/struts标签等）或者手写java表达式（<%=%>）将后台的数据展现出来**。

- 那这个时候，你的所有操作都是在一个服务器上，一旦用户量激增，就很容易出现宕机的问题！
- 另外，动态资源和静态资源全部耦合在一起，服务器压力大，因为服务器会收到各种http请求，例如css的http请求，js的，图片的等等。 一旦服务器出现状况，前后台一起玩完，用户体验极差。

**前后端分离：**

- 前端服务器使用nginx，前端/WEB服务器放的是css，js，图片等等一系列静态资源，**前端服务器负责控制页面引用&跳转&路由**，**前端页面异步调用后端的接口**；后端/应用服务器使用tomcat，加快整体响应速度。
- 页面逻辑，跳转错误，浏览器兼容性问题，脚本错误，页面样式等问题，全部由前端工程师来负责（展示类）。接口数据出错，数据没有提交成功，应答超时等问题，全部由后端工程师来解决（数据类）。
- 在大并发情况下，我可以同时水平扩展前后端服务器，比如淘宝的一个首页就需要2000+台前端服务器做集群来抗住日均多少亿+的日均pv。
- 减少后端服务器的并发/负载压力 除了接口以外的其他所有http请求全部转移到前端nginx上，接口的请求调用tomcat，参考nginx反向代理tomcat。
- 即使后端服务暂时超时或者宕机了，前端页面也会正常访问，只不过数据刷不出来而已。
- 页面显示的东西再多也不怕，因为是异步加载。

## 5、UUID怎么生成？

32位的UUID通用唯一识别码。

UUID由以下几部分的组合：

- 当前日期和时间，UUID的第一个部分与时间有关，如果你在生成一个UUID之后，过几秒又生成一个UUID，则第一个部分不同，其余相同。
- 时钟序列。
- 全局唯一的IEEE机器识别号，如果有网卡，从网卡MAC地址获得，没有网卡以其他方式获得。

UUID的唯一缺陷在于生成的结果串会比较长。

2. **相比较自动增长的int类型的主键有什么好处？**

当数据量多、登录用户量多、遇到高并发的时候，自增长不利于维护，也不利于拓展，而且也有可能出现几个人同时插入自增长的同一个id 比如自增长到了100，ABC三个用户同时插入的时候会不会出现同时插入101？或者覆盖101数据呢？这就是弊端。

**生成22位UUID 改造：**

- 由短域名想到uuid用64进制改造，把uuid生成的字符串去掉“-”，在补一个“0”，得到33位的16进制数，再用22个64进制数表示。
- 利用uuid生成的mostSigBits、leastSigBits来做位移，再通过base64的算法将16字节的2个long类型转换成字符。

## 6、MyBatis中#{}和${}的区别？

1. #将传入的数据都当成一个字符串，会对自动传入的数据加一个双引号。如：order by #user_id#，如果传入的值是111,那么解析成sql时的值为order by "111", 如果传入的值是id，则解析成的sql为order by "id".
2. #{} 的参数替换是发生在 DBMS（**预处理时，会把参数部分用一个占位符 ? 代替** ）中，而 ${} 则发生在动态解析过程中。
3. #方式能够很大程度防止sql注入。
4. 一般能用#的就别用$.MyBatis排序时使用order by 动态参数时需要注意，用$而不是#
