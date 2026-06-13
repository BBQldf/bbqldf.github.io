---
layout:     post
title:     Java Interview-02：Java底层与集合
subtitle:   每日小问——底层系列
date:       2022-03-01
author:     ldf
header-img: img/post-bg-interview01.jpg
catalog: true
tags:
    - java基础
    - 面试
    - code
---



# 负一、java关键字&java 类加载

51+2个保留字（const常量；goto跳转标签）=53个关键字(java的关键字都是小写的！！)

## 1、定义（修饰符，数据类型，类，接口，等）

1. 访问**修饰符**的关键字（共3个）
   - public：可跨包，（默认选择）
   - protected：当前包可用
   - private：当前类可用

2. 定义类、接口、抽象类和实现接口、继承类的关键字、实例化对象（共6个）
   - class：定义类（有实现的方法体）
   - interface：接口（有方法体，但是没有实现）
   - abstract：声明抽象（抽象类abstract class 、抽象方法）。介于类和接口之间，可以有也可以没有实现的方法
   - implements：用于类或接口的实现（public class A interface B(){}）
   - extends：继承类。
   - new：新建（实例化）对象
3. 包（package）关键字（2个）
   - import：引入包
   - package：定义包
4. 修饰方法、类、属性和变量（共9个）
   - **static：**属性和方法都可以用static修饰。只有内部类可以使用static关键字修饰；
   - **final：**方法和类都可以用final来修饰——**final修饰的类是不能被继承的、也不能被重写**；final修饰常量时，表示最终的不可被改变的
   - super：调用父类的方法
   - this：调用当前类中的方法
   - native：本地
   - strictfp：严格（没见过）
   - **synchronized：线程,同步**
   - transient：短暂
   - **volatile：易失**
5. 数据类型的关键字（共12个）

| 关键字  | 意思   | 备注，常用                                        |
| ------- | ------ | ------------------------------------------------- |
| byte    | 字节型 | 8bit                                              |
| char    | 字符型 | 16bit                                             |
| boolean | 布尔型 | --                                                |
| short   | 短整型 | 16bit                                             |
| int     | 整型   | 32bit                                             |
| float   | 浮点型 | 32bit                                             |
| long    | 长整型 | 64bit                                             |
| double  | 双精度 | 64bit                                             |
| void    | 无返回 | public void A(){}  其他需要返回的经常与return连用 |
| null    | 空值   |                                                   |
| true    | 真     |                                                   |
| false   | 假     |                                                   |

6. 条件循环（流程控制）（共12个）
7. 错误处理（共5个）
   - catch
   - try
   - finally
   - throw：抛出一个异常对象：String a; if(a == null), throw new exception("a为null"); 所以throw是一个抛出去的动作
   - throws：声明一个异常可能被抛出；把异常交给他的上级管理，自己不进行异常处理。
     - throws只用在一个方法的末端，表示这个方法体内部如果有异常，这抛给它的调用者。 如： public void add(int a, int b) throws Exception(); 这个方法表示，在执行这个方法的时候，可能产生一个异常，如果产生异常了，那么谁调用了这个方法，就抛给谁。（throws出现在方法函数头；而throw出现在函数体；）
8. 枚举与断言（2个）（不常用）
   - enum：枚举。饿汉模式会用到
   - assert：断言。

### 1.1 String类型的长度？

> 美团面试官问我**一个字符的String.length()**是多少，我说是1，面试官说你回去好好学一下吧。

String.length()方法的定义：

```java
    /**
     * Returns the length of this string.
     * The length is equal to the number of <a href="Character.html#unicode">Unicode
     * code units</a> in the string.
     *
     * @return  the length of the sequence of characters represented by this
     *          object.
     */
    public int length() {
        return value.length;
    }
```

看上面的翻译，即，返回字符串的长度，这一长度等于字符串中的 Unicode 代码单元的数目。

Java中 有内码和外码这一区分简单来说

- 内码：char或String在内存里使用的编码方式。
- 外码：除了内码都可以认为是“外码”。（包括class文件的编码）

而java内码：unicode（utf-16）中使用的是utf-16.所以上面的那句话再进一步解释就是：返回字符串的长度，这一长度等于字符串中的UTF-16的代码单元的数目。

**UTF-16 的 16 指的就是最小为 16 位一个单元（char类型正好也是16位），也即两字节为一个单元，UTF-16 可以包含一个单元和两个单元，对应即是两个字节和四个字节。**我们操作 UTF-16 时就是以它的一个单元为基本单位的。

**例证：**

```java
public class testStringLength {
    public static void main(String [] args){
        String B = "𝄞"; // 这个就是那个音符字符，只不过由于当前的网页没支持这种编码，所以没显示。
        String C = "\uD834\uDD1E";// 这个就是音符字符的UTF-16编码
        System.out.println(C);
        System.out.println(B.length());
        System.out.println(B.codePointCount(0,B.length()));
        // 想获取这个Java文件自己进行演示的，可以在我的公众号【程序员乔戈里】后台回复 6666 获取
    }
}

/**
输出：
 𝄞
 2
 1

*/
```

可以看到通过codePointCount()函数得知这个音乐字符是一个字符！

- codePointCount其实就是代码点数的意思，也就是一个字符就对应一个代码点数。

- 比如刚才音符字符（没办法打出来），它的代码点是U+1D11E，但它的代理单元是U+D834和U+DD1E，如果令字符串str = "u1D11E"，机器识别的不是音符字符，而是一个代码点”/u1D11“和字符”E“，所以会得到它的代码点数是2，代码单元数也是2。

- 但如果令字符str = "\uD834\uDD1E"，那么机器会识别它是2个代码单元代理，但是是1个代码点（那个音符字符），故而，length的结果是代码单元数量2，而codePointCount()的结果是代码点数量1.

### 1.2 抽象类与接口的区别？

1. **最重要的区别是，接口的方法不可以有实现，并且是绝对的抽象方法。抽象类可以有实例方法用来实现默认行为。**
2. 接口中的变量申明默认是final, 而抽象类中变量申明可以是非final。
3. **接口中的成员默认是public修饰，而抽象类中成员可以是private, protected等等。**
4. 接口通过关键字implements被其他类实现，而抽象类则是通过extends关键字被其他类扩展.
5. **接口可以扩展（extends）另外一个/或多个接口，抽象类可以扩展（extends）另一个Java类，并(或)实现（implements）多个Java接口.**
6. Java类可以实现多个接口但是只能扩展一个抽象类.
7. 接口是绝对抽象而且无法实例化，抽象类也无法实例化但是如果类中有main()方法是可以被调用的。
8. 与抽象类相比，接口更慢，因为它需要额外的间接寻址。（这可能就是抽象类的优势吧？）

**看一下例子：**

- 抽象类：

```java
abstract class Car {
   public void accelerate() {
      System.out.println("Do something to accelerate");
   }
   public void applyBrakes() {
      System.out.println("Do something to apply brakes");
   }
   public abstract void changeGears();
}

//继承抽象类，但是必须要实现changeGears()
class Alto extends Car {
   public void changeGears() {
      System.out.println("Implement changeGears() method for Alto Car");
   }
}
class Santro extends Car {
   public void changeGears() {
      System.out.println("Implement changeGears() method for Santro Car");
   }
}
```

- 接口：

```java
public interface Actor {
   void perform();
}
public interface Producer {
   void invest();
}
//使用接口
public interface ActorProducer extends Actor, Producer{
   // some statements
}
```

#### 1.2.1 何时使用抽象类

1. 如果我们使用继承概念，那么抽象类是个不错的选择。因为它为派生类提供了一个通用基类实现。
2. **当定义非public成员时，抽象类也是不错的选择，而接口内，所有的方法都必须是public的**
3. 在未来需要添加新方法时，选择抽象类更为合适(但是要在这个抽象类中把这个新方法实现，否则其他继承的类还是要自己去实现)。因为当我们在接口内添加新的方法时，所有实现该接口的类都需要添加新的方法

4. 如果想创建多版本的组件，那么使用抽象类。抽象类提供一个简单且方便的方式来版本转换我们的组件。通过更新基类，所有继承子类都自动被更新为新的。另一方面，若使用接口则一旦创建则无法修改。如果我们需要新版本的接口只能创建一个全新的接口。

5. 抽象类具有更佳的向后兼容性，我们可以添加新的行为而不影响原有代码，但是如果调用方使用的是接口，我们则无法修改它。

6. 如果想为组件的所有实现中提供公共的且已实现的功能，那么使用抽象类，它允许我们部分实现类。而接口不为任何成员提供任何实现。

#### 1.2.2 何时使用接口

1. 如果我们实现的功能是对迥然不同的对象都很有用，那么使用接口。抽象类主要用作紧密相关的对象之间，然而在给非相关的类提供通用方法时，接口是更好的选择。
2. 在我们认为API在短期内不会修改时，接口是更好的选择。

3. 当我们有一些相似的多重继承时考虑使用接口，因为我们可以实现多重接口。

4. 如果我们设计小巧而简单的功能时使用接口，如果我们设计庞大的功能单元，使用抽象类。

**总结：**

其实就是理念上的差别，抽象类是把接口进行了简化，后面的人继承这个抽象类，只需要把那些没实现的方法实现一下就行了，而那些已经实现的方法所有的孩子类就可以完美继承~



## 2、Java关键字的差别

### 2.1 extends和impl有什么区别？

1. **extends：**继承，继承的是类，并且只能继承一个类，继承的时候可以重写父类的方法，继承之后可以使用父类的非私有方法。`public class Dog extends Animal`（更像是定义属性， 狗是动物，如果动物类里面实现了跑，那狗也可以跑）

   - 只要那个类不是声明为final或者那个类定义为abstract的就能继承；
   - 特例：extends可以跟接口，一个接口可以继承多个接口。
   - 特例：大类上面不能一次性继承多个类，但是内部类可以慢慢地引入多个类。比如遗传：我**们即继承了父亲的行为和特征也继承了母亲的行为和特征。**可幸的是Java是非常和善和理解我们的,它提供了两种方式让我们**曲折**来实现多重继承：接口和内部类。

   ```java
   public class Son {
       
       /**
        * 内部类继承Father类
        */
       class Father_1 extends Father{
           public int strong(){
               return super.strong() + 1;
           }
       }
       
       class Mother_1 extends  Mother{
           public int kind(){
               return super.kind() - 2;
           }
       }
       
       public int getStrong(){
           return new Father_1().strong();
       }
       
       public int getKind(){
           return new Mother_1().kind();
       }
   }
   ```

2. **implements：**实现，实现的是接口，并且必须实现接口中所有的方法。`public class Dog implements Runner`(更像是实现一种功能，比如狗可以跑，猫也可以跑)，可以一次性实现多个接口。

   - 接口中的定义全部都要实现！！！
   - 如果只实现部分，就变成了一个抽象类了，然后这个类不能实例化，必须由其子类没有实现的逻辑。

### 2.2 可不可以只有接口，只去实现接口，不去继承类？

可以，但是最好不要！类和接口都是为了程序的可扩展性设计的。**接口就是一个特殊的抽象类。**

- 比如你已经写好了1个类。而且也已经写好了所有的方法，通俗点说就是实现了所有的功能。但现在如果又要多增加个功能。比如，狗可以叫，但是狗要喝水，你就可以考虑实现接口。接口里写要增加的喝水的方法。但是继承的话，就不太好弄
- 反过来，如果已经有了一个父类Animal，他已经实现了这两个接口，重写了相关方法，那Dog类只用去继承就可以了。（避免了冗余，结构也更清晰）

### 2.3 抽象类的作用？

先要弄清楚这三个概念：

- class：定义类（有实现的方法体）
- interface：接口（有方法体，但是没有实现）
- abstract：声明抽象（抽象类abstract class 、抽象方法）。

前面两个都好理解，也用的多。

**父类是将子类所共同拥有的属性和方法进行抽取，这些属性和方法中，有的是已经明确实现了的，有的还无法确定，那么我们就可以将其定义成抽象，在后日子类进行重用，进行具体化。**所以，他介于接口和实现类之间。

- 抽象类是为了把相同的但不确定的东西的提取出来，**为了以后在子类中重用。**（所以，一般你继承一个抽象类的时候，idea会自动要求你@Override相应的方法）

```
// 这就是一个抽象类
public abstract class Animal {
	String name;
	int age;
 
	// 动物会叫；这是一个抽象方法
	/*不确定动物怎么叫的。定义成抽象方法，来解决父类方法的不确定性。抽象方法在父类中不能实现，
	 * 所以没有函数体。但在后续在继承时，要具体实现此方法。*/
	public abstract void cry();
}
 
// 抽象类可以被继承
// 当继承的父类是抽象类时，需要将抽象类中的所有抽象方法全部实现。
class cat extends Animal {
	// 实现父类的cry抽象方法
	public void cry() {
		System.out.println("狗叫:");
 
	}
}
```

- 所以，抽象类中的方法是不完整的，也不能直接实例化。（当然，abstract类里面也可以没有abstract方法）
  - **特例：**可以利用内部子类实例化，即 利用匿名内部类可以实现抽象类的（相当于new一个抽象方法）；也可以通过abstract **父类的引用来指向子类的实例，**`子类A继承abstract 父类B，B aa=new A(“a”);`就实例化了一个B类对象aa。
- 如果子类没有实现父类的抽象方法，则必须将子类也定义为abstract类。（不定义idea会报错）
- 抽象方法必须为public或者protected（因为如果为private，则不能被子类继承，子类便无法实现该方法；也不能是final的，final类（如String 类）也是不能修改，不能被重写的情况；）

#### 2.3.1 abstract怎么进行接口实现？

抽象类实现接口时，不必实现接口中的所有方法，未实现的方法可以交由子类来实现。



### 2.4 普通属性、静态属性、静态代码块有什么区别？

其实这里，我们只要理解一个概念，什么是“静态”？（最直观的理解，静静地在哪里，一开始就存在，谁想用就直接用，也可以改动（不想被改就用final修饰，这就相当于变成了常量））

- static用来修饰成员变量和成员方法，也可以形成静态static代码块，但是Java语言中没有全局变量的概念。
- static修饰的成员变量和成员方法不依赖类特定的实例，被类的所有实例共享。
  - 用public修饰的static成员变量和成员方法本质是全局变量和全局方法，所有的类都能使用
  - static变量前可以有private修饰，表示这个变量可以在类的静态代码块中，或者类的其他方法中使用，但是不能在其他类中通过类名来直接引用

<font color='red'>重点区别：</font>

1. 对于静态变量在内存中只有一个拷贝(节省内存)，JVM只为静态分配一次内存，在加载类的过程中完成静态变量的内存分配；对于实例变量（普通变量），没创建一个实例，就会为实例变量分配一次内存，实例变量可以在内存中有多个拷贝，互不影响(灵活)。

#### 2.4.1 静态方法？

1. 静态方法可以直接通过类名调用，任何的实例也都可以调用，
2. 静态方法中不能用this和super关键字，不能直接访问所属类的实例变量和实例方法

#### 2.4.2 静态代码块？

static代码块是在类中独立于类成员的static语句块，可以有多个，位置可以随便放，它不在任何的方法体内，JVM加载类时会执行这些静态的代码块，如果static代码块有多个，JVM将按照它们在类中出现的先后顺序依次执行它们，**每个代码块只会被执行一次**。（非静态的代码块，每次new创建对象的时候，都要重新执行）

既然只执行一次？那它有什么作用？项目里面什么情况要用这个？用于执行那些只需要执行一次的代码，例如驱动加载等。说白了，静态代码块其实就是给类初始化的，而构造代码块是给对象初始化的。

```java
//class A 
package com.my.test;


class A {

    static {
        System.out.println("A1:父类静态代码区域");
    }

  
    

    {
        System.out.println("A2：父类非静态代码区域");
    }

    public A() {
        System.out.println("A3：父类构造器");
    }
}

//class B

package com.my.test;

public class B extends A {

    static {
        System.out.println("B1:子类静态代码区域");
    }

  
    

    {
        System.out.println("B2：子类非静态代码区域");
    }

    public B() {
        System.out.println("B3：子类构造器");
    }
}


// 测试类
package com.my.test;

public class Test {
    public static void main(String[] args) {
        B b1 = new B();
        System.out.println("====");
        B b2 = new B();
        

    }

}
/**输出：
A1:父类静态代码区域
B1:子类静态代码区域
A2：父类非静态代码区域
A3：父类构造器
B2：子类非静态代码区域
B3：子类构造器
====
A2：父类非静态代码区域
A3：父类构造器
B2：子类非静态代码区域
B3：子类构造器
*/
```

### 2.5 它们加载的顺序？

**执行顺序：**类内容（静态变量、静态初始化块——他们的顺序按代码编写的先后，建议静态变量写在最前面)—>非静态代码块（变量，方法）—>构造方法。

**区别：**<font color='red'>静态代码块是自动执行的，而静态方法是被调用的时候才执行的。</font>这句话很核心哦，饿汉式实现了懒汉式就是利用静态内部类是在执行是第一个加载来完成的。

**注意：**

- 静态代码块只能定义在类里面，它独立于任何方法，不能定义在方法里面。
- 静态代码块里面的变量都是局部变量，只在本块内有效。

#### 2.5.1 静态类可不可以访问非静态类？

- 不能。（和上面一样的情况）
- 静态类不能访问外部类的非静态成员。他只能访问外部类的静态成员。
- 如果一个类要被声明为static的，只有一种情况，就是**静态内部类**。如果在外部类声明为static，程序会编译都不会过。
- 静态内部类跟静态方法一样，只能访问静态的成员变量和方法，不能访问非静态的方法和属性，但是普通内部类可以访问任意外部类的成员变量和方法
- **例如，**我在Main类下创建一个内部类Entry，然后在主方法上要创建Entry的对象。如果Entry不是静态的话，我必须先创建一个Main对象，然后在该对象中创建Entry对象:

```java
Main ma=new Main();
//注意是ma.new Entry(),不是new ma.Entry().前者表示的是在ma中创建对象
Entry entry=ma.new Entry();
```

- 但是如果使用静态内部类的话:

  ```java
  Entry entry=new Main.Entry();
  //注意这里直接写Entry即可，甚至不需要Main.Entry来表示类名(测试还是要的)
  ```

- 场景：如在进行代码程序测试的时候，如果在每一个Java源文件中都设置一个主方法(主方法是某个应用程序的入口，必须具有)，那么会出现很多额外的代码。在这种情况下，就可以将主方法写入到静态内部类中，从而不用为每个Java源文件都设置一个类似的主方法。

#### 2.5.2 静态方法可不可以访问非静态方法？

- 在static方法内部不能调用非静态方法，反过来是可以的。（很好理解吧，非静态的方法，可能一开始都不存在，咋调？）
- 静态方法是属于类的，即静态方法是随着类的加载而加载的，在加载类时，程序就会为静态方法分配内存 非静态方法是属于对象的，对象是在类加载之后创建的 静态方法先于对象存在,所以如果静态方法调用非静态方法的话，可能会报空指针异常。

#### 2.5.3 静态代码块，和静态方法的区别？

- 如果有些代码必须在项目启动的时候就执行的时候,需要使用静态代码块,这种代码是主动执行的;
- 需要在项目启动的时候就初始化,在不创建对象的情况下,其他程序来调用的时候,需要使用静态方法,这种代码是被动执行的. 静态方法在类加载的时候 就已经加载 可以用类名直接调用。
- **两者的区别就是：静态代码块是自动执行的; 静态方法是被调用的时候才执行的.**

### 2.6 总结一下static关键字的一些属性：

1. 静态元素在类加载的时候就被初始化，创建的很早，那时还没有创建对象
2. 静态元素存储在静态元素区中，每一个类都有自己的一个单独的区域，与别的类不冲突（反复调用这个类也不会出现变化）
3. 静态元素区不能被GC管理，可以简单的认为静态元素常驻内存（不会被清理）
4. 静态元素只加载一次，供全部的类对象和类本身共享
5. 静态元素与对象没有关系，它属于类
6. 静态元素可以直接访问静态元素。
7. 非静态元素可以直接方法静态元素，但是静态元素不能直接方法非静态元素
8. 静态元素中不可以使用this，super关键字

### 2.7 内部类：静态内部类、实例内部类、局部内部类

> 首先要清楚匿名内部类定义的作用：不用去实现接口并且调用接口内部的方法，而是直接调用。

- 什么是内部类？

在类的内部又定义了一个新的类，被称为内部类

1. 静态内部类
2. 实例内部类
3. 局部内部类

内部类和外部类的访问有如下规则：

1. 内部类可以直接访问外部类的成员，包括私有成员
2. **外部类要访问内部类的成员（成员变量，成员方法），必须要建立内部类的对象**（这也是为什么匿名内部类需要new XXX的原因）

```java
package com.wang.IntoClass;

public class IntoClassTest {
    public static void main(String[] args) {
        //调用Mymath中的Mysum方法
        Mymath mymath=new Mymath();
        mymath.SUM(new ComputeImpl(),100,200);
    }
    //静态内部类，前面有Static
    static class Inner1{

    }
    //没有static叫做实例内部类
    class Inner2{

    }
    //方法
    public void doSome(){
        //局部内部类
        class Inner3{
        }
    }

    public void doOther(){
        //doSOme（）方法中的局部内部类INNer3在doOther（）中不能用;

    }
}
interface Compute{
    int sum(int a,int b);
}

class ComputeImpl implements Compute{
    public int sum(int a,int b)
    {
        return a+b;
    }
}
//常规的调用方法
class Mymath{
    //数学求和方法
    public void SUM(Compute C,int x,int y)
    {
        int Value=C.sum(x,y);
        System.out.println(x+"+"+y+"="+Value);
    }
}
//使用匿名内部类调用
    public static void main(String[] args) {
        //调用Mymath中的Mysum方法
        Mymath mymath=new Mymath();
        //这里表面看上去是对接口可以直接new了，实际上并不是接口可以直接new了，后面的{}是对接口的实现
        //不建议使用匿名内部类，因为一个类没有名字没有办法重复使用，另外代码太乱，可读性很差
        mymath.SUM(new Compute() {
            @Override
            public int sum(int a, int b) {
                return a+b;
            }
        },100,200);
    }
```

**匿名内部类是局部内部类的一种简化形式。（也就是没有名字，这是何必呢？也不会降低什么复杂度。）**<font color='red'>本质上是一个对象,是实现了该接口或继承了该抽象类的子类对象。</font>

- 所以，匿名内部类肯定是结合了某个场景来使用才比较好。比如：有时候有的内部类只需要创建一个它的对象就可以了，**以后再不会用到这个类**，这时候使用匿名内部类就比较合适，而且也免去了给它取名字的烦恼。

- 有这样几个规则：

  - **只用到类的一个实例**
  - 类在定义后马上用到
  - 类非常小（SUN推荐是在4行代码以下）
  - 给类命名并不会导致你的代码更容易被理解

- 也有这样几个规范：

  1. 匿名内部类不能有构造方法
  2. 匿名内部类不能定义任何静态成员、方法和类
  3. 匿名内部类不能是public,protected,private,static（也就是说不要对内部类进行规范）
  4. 只能创建匿名内部类的一个实例
  5. 一个匿名内部类一定是在new的后面，用其隐含实现一个接口或实现一个类
  6. 因匿名内部类为局部内部类，所以局部内部类的所有限制都对其生效（其实也就是上线的1~4）

- 比较常见的场景：

  1. 创建线程：

  ```
  //继承Thread
  new Thread(){
  	public void run(){
  		//do something
  	};
  }.start();
  //实现 Runnable接口
  new Thread(new Runnable() {
  	public void run() {
  		//do something
  	};
  }).start();
  ```

- 为什么要用？

  - 一般情况下，我们要继承一个父类的方法，比如Teacher类中有一个run的方法。我们在调用的时候需要先创建一个teacher对象，再调用这个对象的run方法；但是内部类可以直接覆写这个run方法

  ```java
  //People抽象类，类中有run()方法
  public abstract class People {
   	public abstract void run()；
   }
  //Teacher类继承People类，重写其run()方法
  public class Teacher extends People{
   	public void run(){
    		System.out.println("老师在跑步");
   	}
  }
  //调用它
  public class Demo {
   	public static void main(String[] args) {
    		Teacher t = new Teacher();
    		t.run();
   	}
  }
  
  //但是使用匿名内部类就会简化掉Teacher类；但是前提是这个Teacher类只运行一次
  public class Demo {
   	public static void main(String[] args) {
    		new People(){
     			public void run(){
      				System.out.println("老师在跑步");
     			}
    		}.run();
   	}
  }
  //如果运行多次，那就得不偿失了；比如当经常要用到某个类，上面需要多次输出System.out.println("老师在跑步")；这就是冗余的
  public class Demo {
   	public static void main(String[] args) {
    		new People(){
     			public void run(){
      				System.out.println("老师在跑步");
     			}
    		}.run();
    		new People(){
     			public void run(){
      				System.out.println("老师在跑步");
     			}
    		}.run();
    		new People(){
     			public void run(){
      				System.out.println("老师在跑步");
     			}
    		}.run();
   	}
  }
  ```

  

学习匿名内部类主要是以后在阅读别人代码的时候，能够理解，并不代表以后要这样写（最好不要这样写），匿名类有两个缺点：

1. 太复杂太乱，可读性差
2. 没有名字，以后想重复使用，不能用

```
语法格式：
    new 类名或者接口名(){
        重写方法;
    };
```

匿名内部的好处：

1. 经常作为参数，或返回值，使用比较方便
2. 利用多态来给匿名内部类命名，并实现调用

```
//1. 创建新对象，直接调用
public class MyTest {
    public static void main(String[] args) {
        //匿名内部类
        new AA() {
            @Override
            public void aa() {
                System.out.println("aaaaaaaaaaaaaaaa");
            }
 
            @Override
            public void hehe() {
                System.out.println("hehehehehehhehehhe");
            }
        };
        //利用多态进行命名
        AA aa = new AA() {
            @Override
            public void aa() {
                System.out.println("aaaaaaaaaaaaaa744444aa");
            }
 
            @Override
            public void hehe() {
                System.out.println("hehehehehe888877777hhehehhe");
            }
        };
        //这样就可以调用多个方法了
        aa.aa();
        aa.hehe();
 
    }  
}
abstract class AA {
    public abstract void aa();
 
    public abstract void hehe();
}

//2. 作为参数传进去
public class MyTest {
    public static void main(String[] args) {
        //匿名内部类，经常作为参数，使用比较方便。      
        test(new WW() {
            @Override
            public void hehe() {
                System.out.println("111111111");
            }
        });
    }
    //方法的形参要一个抽象类类型，传递一个该抽象类的子类对象.
    public static void test(WW ww) {
        ww.hehe();
    }
}
abstract class WW {
    public abstract void hehe();
}

//3. 作为返回值
public class MyTest {
 
    public static void main(String[] args) {
        BB bb = getBB();
        bb.bb();
 
        //匿名内部类，作为返回值，返回方便
    }
 
    //方法的返回值是一个抽象类 类型，返回一个该抽象类的子类对象。
    public static BB getBB2() {
        BB b2 = new BB() {
 
            @Override
            public void bb() {
                System.out.println("bbbbbbbbbbbbbbbbb222222222");
            }
        };
        //匿名内部类，作为返回值，返回方便
        return b2;
    }
}
 
abstract class BB {
    public abstract void bb();
}
```

### 2.8 lambda表达式

> "Lambda表达式"可以看做是一个匿名函数，是一种高效的类似于函数式编程的表达式。
>
> 一般而言，我们把他们当做是匿名内部类的进一步简化版；
>
> **要点：**lambda表达式只能与功能接口一起使用。

Lambda省去面向对象的条条框框，格式由3个部分组成：

- 参数部分
- 箭头
- 代码块

比如：`(参数类型 参数名称) -> { 代码语句 }`

A lambda表达式是实现函数接口的内联代码，不需要创建具体或匿名的类。 **Lambda表达式基本上是一个匿名的方法**。

**优势：**

1. 相比于匿名内部类，又进一步地减少了代码量
2. 减少匿名内部类的创建，节省资源。（匿名内部类在编译的时候会产生新的class文件，
3. 更高的效率-过使用Stream API（流方式）和lambda表达式，可以在批量操作集合的情况下获得更高的效率(并行执行)

**缺点：**

1. 不易于后期维护，必须熟悉lambda表达式和抽象函数中参数的类型
2. lambda函数中也无法添加日志信息
3. 只能扩展只有一个方法的接口
4. lambda表达式内使用外部的局部变量 必须是final类型的或者事实上是final类型的，虽然变量没强制必须用final修饰 ，但是也不允许在lambda表达式内被更改

2. 可读性差



#### 2.8.1 lambda表达式原理

1.  Lambda 表达式引用的局部变量必须是最终变量或实际上的最终变量，也就是说局部变量在被创建后不得被重新赋值（也就是说只能操作final变量）
2. Lambda表达式是一个语法糖，会被编译生成为当前类的一个私有方法
3. Lambda表达式内直接引用局部变量**本质是一种隐式传参，**编译时会自动将引用的局部变量放到参数列表中（Lambda方法多了个参数），而引用的实例变量并不需要放到参数列表，因为方法内可以直接引用。（理解了方法，再理解一下传参，基本就是lambda的实现过程了）
   - 造成直接引用的局部变量需要**final修饰**的原因应该和这种隐式传参有关。因为，Java中引用数据类型是由**引用变量**和**指向的实际对象**两部分组成的。在方法传参时，本质上是将实际对象的内存地址赋值给方法参数中的引用变量



#### 2.8.2 使用lambda表达式需要注意的点？——需要函数式编程接口

比如在Runnable接口上我们可以看到@FunctionalInterface注解（标记着这个接口**只有一个抽象方法**）

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/v2-f156e4060cc4c567bd6be8d0c7102eef_1440w.jpg)

我们使用Lambda表达式创建线程的时候，**并不关心接口名，方法名，参数名**。我们**只关注他的参数类型，参数个数，返回值**。

#### 2.8.3 常用的函数式编程接口（JDK原生）

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/v2-d4ff58a1bad0b37266d009e7047dec95_r.jpg)

编程实例可以参考：https://zhuanlan.zhihu.com/p/531120120

- java.util.function.Supplier\<T> 接口仅包含一个无参的方法：T get() 。用来获取一个泛型参数，可以指定类型的对象数据。
- java.util.function.Consumer\<T> 接口则正好与Supplier接口相反，它不是生产一个数据，⽽是消费一个数据，其数据类型由泛型决定。
- 用java.util.function.Predicate\<T>接口，对某种类型的数据进行判断，从而得到一个boolean值结果。（包含一个抽象方法：boolean test(T t)）
- java.util.function.Function<T,R>接口用来根据一个类型的数据得到另一个类型的数据，前者称为前置条件，后者称为后置条件。



#### 2.8.4 [lambda表达式-并行流，提升效率的利器？](https://blog.csdn.net/weigeshikebi/article/details/80030312)

> Lambda除了在写法上的不同，还有其它什么作用呢？当然有，就是数据并行化处理！它在某些场景下可以提高程序的性能。

Lambda实现并行编程很容易：

1. 集合可以通过parallelStream()方法获取拥有并行处理能力的Stream；

2. Stream可以通过parallel()方法标记你希望以并行的方式处理。

```
double sum = IntStream.range(1, 1000000)
    .asDoubleStream()
    .parallel()
    .map(x -> Math.sin(x) * Math.sin(x)  + Math.cos(x) * Math.cos(x))
    .sum();
```

举例：

```
// 流方式
        List<Person> newBoys = personList.stream().filter(p -> 1 == p.getSex()).collect(Collectors.toList());
// 并行流方式：找出所有男同学
		List<Person> newBoys = personList.parallelStream().filter(p -> 1 == p.getSex()).collect(Collectors.toList());        
```

##### 什么是并行？

Java支持多线程，可以同时开启多个任务。引入多线程的原因在于，线程可能会阻塞，CPU会主动切分**时间片**，只有分配到时间片的线程才会运行。而现代的处理器，几乎都是多核的，即多个CPU，如何才能更高效的利用硬件呢，多线程。

**区别：**

- 串行指一个步骤一个步骤地处理，也就是通常情况下，代码一行一行地执qub行
- 并发（concurrent）：在操作系统中，是指一个时间段中有几个程序都处于已启动运行到运行完毕之间，且这几个程序都是在同一个处理机（CPU）上运行；（利用CPU时间片技术）。
  - 打游戏和听音乐两件事情在同一个时间段内都是在同一台电脑上完成了从开始到结束的动作。那么，就可以说听音乐和打游戏是并发的。
- 并行（parallel）：有多个任务执行单元（强调有多个CPU），从物理上就可以多个任务一起执行

```
举例：
你吃饭吃到一半，电话来了，你一直到吃完了以后才去接，这就说明你不支持并发也不支持并行。

你吃饭吃到一半，电话来了，你停了下来接了电话，接完后继续吃饭，这说明你支持并发。

你吃饭吃到一半，电话来了，你一边打电话一边吃饭，这说明你支持并行。
```

注意：

1. 并行处理的Stream会自行处理锁的问题，所以不要在没有把握的时候自行加锁，尤其要注意有些方法自带synchronized，并行的效果反而没有顺序执行(sequential)好。
2. 并且，并行处理还可能会涉及到CPU线程切换，这也是有个耗时的地方，导致并行处理比较慢
3. 是并行流涉及数据切割与合并，所以只有在处理的数据量大且耗时高的场景才会体现优势



## 3、类/方法/变量加载的过程（举例分析）

先给出结论：

### 3.1 类初始化的过程：

1. 一个类要创建实例需要先加载并初始化该类。main()方法所在的类需要先加载并初始化。
2. 一个子类要初始化需要先初始化父类。
3. 一个类初始化就是执行\<clinit>()方法。\<clinit>()方法包括：静态变量和静态代码块，他们的加载顺序同执行顺序。且只执行一次。

类加载完成之后，才会进入到实际的创建实例的过程！

### 3.2 实例初始化的过程：

实例初始化就是执行\<init>()方法。（对比clinit）

1. \<init>()方法包括：非静态变量、非静态代码块、对应构造器（最后执行）。
2. 每次创建实例，调用对应构造器，就是去执行\<init>()方法。
3. 执行的时候也是先执行父类对应的内容，然后再执行子类。
4. 非静态变量初始化的时候还要考虑重写问题。**如果存在重写，是执行子类中的内容**。

另外，关于方法重写的内容，以下这些情况，方法是不能被重写的，包括： 

- final修饰的方法。
- 静态方法。
- private修饰的方法，导致子类中不可见，所以不可以被重写。

ok，这里都是宏观地去看，下面我们利用JVM的原理去看看，类是怎么一步步被ClassLoader加载到内存的！！

### 3.3 类的加载全流程——加载，验证，准备，解析，初始化，使用，卸载

#### 3.3.1 加载

1. Java虚拟机将.class文件（在cmd下使用javac 编译某一java文件则会产生.class文件）读入内存，并为之创建一个Class对象。放在**方法区**内。
2. 解析类的二进制流为方法区内的数据结构
3. 然后在**堆**中创建java.lang.Class**对象（记住，这只是一个对象！！）**，用来封装类在方法区的数据结构。作为方法区这个类的访问入口（堆中的对象引用到了方法区）

#### 3.3.2 链接——验证、准备、解析——一条龙解决~~

1. 验证阶段：主要是确保Class文件的**格式**正确，运行时不会危害虚拟机的安全（比如，看他的版本号是不是虚拟机所支持的；是否继承了final类；字节码有没有问题；符号引用对应的直接引用是否存在）
2. 准备阶段：负责为类的静态成员**分配内存**，并**设置默认初始值**（注意是默认值，不是设定的值比如static int i= 1；i一开始是0，后面再赋值为1）。如果类静态变量的字段属性表中存在**ConstantValue属性**，则直接执行赋值语句，比如这两种：
   1. 类静态变量为基本数据类型，并且被final修饰
   2. 类静态变量为String类型，被final修饰，并且以字面量的形式赋值

```java
public class Person {
    private static int age = 10;
    private static final int length = 160;
    private static final String name = "name";
    private static final String loc = new String("loc");
}
```

​			所以，length和name属性在准备阶段就会赋值为ConstantValue指定的值。而age和loc变量则会在后面初始化阶段

3. 解析阶段：将类的二进制数据中的符号引用替换为直接引用。那什么是符号引用？什么是直接引用呢？
   1. 举个例子来说，现在调用方法hello()，这个方法的地址是0xaabbccdd，那么hello就是符号引用，0xaabbccdd就是直接引用。再比如，上面的age，也要指向直接引用。
   2. 在解析阶段，虚拟机会把所有的类名，方法名，字段名这些符号引用替换为具体的内存地址或偏移量，也就是直接引用。

#### 3.3.3 初始化(就是上面说的\<clinit>()方法)

类的\<clinit>()方法：

- 初始化，则是为标记为常量值的字段赋值的过程。换句话说，只对static修饰的变量或语句块进行初始化。
- 如果初始化一个类的时候，其父类尚未初始化，则优先初始化其父类。
- 如果同时包含多个静态变量和静态代码块，则按照自上而下的顺序依次执行。

#### 3.3.4 使用（就是上面说的\<init>()方法）

实例的\<init>()方法：

- 调用父类的\<init>()方法
- 非静态成员变量赋值
- 执行构造代码块
- 执行构造函数

#### 3.3.5 最后就是卸载~~（即垃圾回收GC）

**垃圾收集不仅发生在堆中，方法区上也会发生。**但是对方法区的类型数据回收的条件比较苛刻。（因为方法区里面很多都是对象的引用，而方法区里面都是类信息，静态变量，常量）

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxNy85LzQvZGE3N2Q5MDE0Njc4NmMwY2IzZTE3MGI5YzkzNzZhZTQ_aW1hZ2VWaWV3Mi8wL3cvMTI4MC9oLzk2MC9mb3JtYXQvd2VicC9pZ25vcmUtZXJyb3IvMQ)

- 比如，上面的Simple类，只有堆中的Simple对象、代表Simple类的Class对象、ClassLoader对象都被回收了，方法区的Simple类的二进制数据结构才**有可能**被回收。

- 以看到对方法区的类型数据回收的条件比较苛刻，但是收效甚微，所以有些垃圾收集器不会对方法区的类型数据进行回收

## 4、类加载之“双亲委派”机制

> **双亲委派机制**是指当一个类加载器收到一个类加载请求时，该类加载器首先会把请求委派给父类加载器。每个类加载器都是如此，只有在父类加载器在自己的搜索范围内找不到指定类时，子类加载器才会尝试自己去加载。（先找双亲，双亲有的话，就直接加载）

![img](https://img2018.cnblogs.com/i-beta/1724418/201911/1724418-20191126152838337-942490000.png)

首先要搞清楚，到底**什么是类加载器？**为什么很重要？

类加载器是jre的一部分，负责动态将类添加到Java虚拟机。

### 4.1 双亲委派模型工作工程：

1. 当Application ClassLoader 收到一个类加载请求时，他首先不会自己去尝试加载这个类，而是将这个请求委派给父类加载器Extension ClassLoader去完成。 

2. 当Extension ClassLoader收到一个类加载请求时，他首先也不会自己去尝试加载这个类，而是将请求委派给父类加载器Bootstrap ClassLoader去完成。  

3. 如果Bootstrap ClassLoader加载失败(在<JAVA_HOME>\lib中未找到所需类)，就会让Extension ClassLoader尝试加载。 

4. 如果Extension ClassLoader也加载失败，就会使用Application ClassLoader加载。 

5. 如果Application ClassLoader也加载失败，就会使用自定义加载器去尝试加载。 

6. 如果均加载失败，**就会抛出ClassNotFoundException异常。**

**常见问题：**在自己的项目里新建一个java.lang包，里面新建了一个String类，能代替系统String吗？

不能，因为根据类加载的双亲委派机制，会将请求转发给父类加载器，父类加载器发现冲突了String就不会加载了。

所以，双亲委派模型的**好处**主要在于Java类随着它的类加载器一起具备了一种带有**优先级的层次关系**。

### 4.2 哪些操作是违背了“双亲委派”机制的？

> 在实际的应用中双亲委派解决了java 基础类统一加载的问题，但是却着实存在着一定的缺陷。
>
> 比如：**JDBC的驱动加载过程，Tomcat类加载器等**
>
> 破坏双亲委派的本质：有一个自己实现了的loadClass方法，并且重写了loadClass方法，让它不进行双亲委派。

**“双亲委派”机制的缺陷？**

- 我们系统里抽象的各个模块，往往有很多不同的实现方案，比如日志模块、xml解析模块、jdbc模块等方案。
- 面向对象的设计里，我们一般推荐模块之间基于接口编程，模块之间不对实现类进行硬编码。一旦代码里涉及具体的实现类，就违反了可拔插的原则，如果需要替换一种实现，就需要修改代码。（其实就是**继承**，**封装**和**多态**）。
- 为了实现在模块装配的时候能不在程序里动态指明，这就需要一种服务发现机制。**Java SPI**就是提供这样的一个机制：为某个接口寻找服务实现的机制。有点类似IOC的思想，就是将装配的控制权移到程序之外，在模块化设计中这个机制尤其重要。具体而言，就是4.2.1的`约定`

#### 4.2.1 SPI机制简介

> 全称叫Service Provider Interface，是基于接口的动态扩展机制；相当于Java里面提供了一套标准接口，然后第三方可以去实现这个接口，来完成功能的扩展和实现。（这里就是依赖倒置原则和开闭原则的体现；也是策略模式的体现）

**Java SPI的具体约定为：**

- 当服务的提供者提供了服务接口的一种实现之后，在jar包的META-INF/services/目录里同时创建一个以服务接口命名的文件，该文件里就是实现该服务接口的具体实现类。
- 而当外部程序装配这个模块的时候，就能通过该jar包META-INF/services/里的配置文件找到具体的实现类名，并装载实例化，完成模块的注入。（相当于把装配的控制权移到程序之前，完成标准和实现的解耦）

**举个例子：**

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220920105850.png)

在Java的SDK里面，提供了对数据库访问的标准接口`java.sql.Driver`，但是java中并没有提供这个接口的实现。这样，不同的数据库厂商会有不同的语法和实现，比如：oracle的是`oracle.jdbc.OracleDriver`，Mysql的是`com.mysql.jdbc.Driver`。

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220920110332.png)

有一个小问题：

- SPI每次都会去加载扩展接口的所有实现类并进行实例化，这就会有性能开销；并且还有一些不会用到的对象，也被实例化了，占用内存



**SPI机制带来的问题：**

- SPI 的接口由 Java 核心库来提供，而这些 SPI 的实现则是由各供应商来完成。终端只需要将所需的实现作为 Java 应用所依赖的 jar 包包含进类路径（CLASSPATH）就可以了。
- **问题在于SPI接口中的代码经常需要加载具体的实现类**：SPI的接口是Java核心库的一部分，是由启动类加载器来加载的；而SPI的实现类是由系统类加载器来加载的。启动类加载器是无法找到 SPI 的实现类的(因为它只加载 Java 的核心库)，按照双亲委派模型，启动类加载器无法委派系统类加载器去应用级的加载类（它只能往上加载，不能往下加载）。也就是说，类加载器的双亲委派模式无法解决这个问题。
- 所以，能不能在执行线程中抛弃双亲委派加载链模式，使程序可以逆向使用类加载器？



#### 4.2.2 JDBC怎么打破双亲委派？

但是呢，如果只有双亲委派机制的话，如果我们应用层需要对不同的父类进行加载，就会出现问题。

- 特别的，比如JDBC加载驱动，Class.forName(DriverName)，其实是需要BootStrap启动类去加载，但是我们是用的加载，BootStrap类加载器就加载不到这个驱动实现类。
- 线程上下类加载器就解决了这个问题。Java.lang.Thread中的方法 getContextClassLoader()和 setContextClassLoader(ClassLoader cl)用来获取和设置线程的上下文类加载器。如果没有通过 setContextClassLoader(ClassLoader cl)方法进行设置的话，线程将继承其父线程的上下文类加载器。
- 如果不做任何的设置，Java应用的线程上下文类加载器默认就是系统类加载器。因此，在 SPI 接口的代码中使用线程上下文类加载器，就可以成功的加载到 SPI 实现的类。

具体而言：

我们可以看一下`JDBC驱动注册的常用几种方式`：

方式一：Class.forName(“com.mysql.jdbc.Driver”)：这句话的作用是加载并初始化指定驱动。mysql jdbc正是在Driver初始化的时候完成注册

方式二：System.setProperty(“jdbc.drivers”,“com.mysql.jdbc.Driver”)：这种方式是通过系统的属性设置注册驱动，最终还是通过系统类加载器完成。

方式三：SPI服务加载机制注册驱动：这种方式与第一种方式唯一的区别就是经常写的Class.forName被注释掉了，但程序依然可以正常运行，这是为什么呢？

```
try {
    // Class.forName(driver);
    conn = (Connection)DriverManager.getConnection(url, user, passwd);
} catch (Exception e) {
    System.out.println(e);
}
```

其实就是要找到SPI在哪一步自动注册了mysql driver的？

- 重点就在DriverManager.getConnection()中。

```java
static {
    loadInitialDrivers();
    println("JDBC DriverManager initialized");
}
```

我们知道，调用类的静态方法会初始化该类，loadInitialDrivers()方法就是去上面的方式二和方式三。（方式一不会出问题），并且，看一下loadInitialDrivers()方法的源码，我们也知道了JDBC中的DriverManager加载Driver的步骤顺序依次是：

1. 通过SPI方式，读取 META-INF/services 下文件中的类名，使用线程上下文类加载器加载；
2. 通过System.getProperty(“jdbc.drivers”)获取设置，然后通过系统类加载器加载。

**ok，可以看一下SPI是怎么解决的：**

- 因为`Class.forName(DriverName, false, loader)`代码所在的类在java.util.ServiceLoader类中，而`ServiceLoader.class`又加载在BootrapLoader中，因此传给 forName 的 loader 必然不能是BootrapLoader(启动类加载器只能加载java核心类库)。
- 这时候只能使用线程上下文类加载器了：把自己加载不了的类加载到线程上下文类加载器中（通过Thread.currentThread()获取），而线程上下文类加载器默认是使用系统类加载器AppClassLoader。

```java
public static <S> ServiceLoader<S> load(Class<S> service) {
    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    return ServiceLoader.load(service, cl);
}
```

回头再看ServiceLoader.load(Class)的代码，的确如此：

- ContextClassLoader默认存放了AppClassLoader的引用，由于它是在运行时被放在了线程中，所以不管当前程序处于何处（BootstrapClassLoader或是ExtClassLoader等），在任何需要的时候都可以用`Thread.currentThread().getContextClassLoader()`取出应用程序类加载器来完成需要的操作。
- 到这儿差不多把SPI机制解释清楚了。直白一点说就是：我（JDK）提供了一种帮你（第三方实现者）加载服务（如数据库驱动、日志库）的便捷方式，只要你遵循约定（把类名写在/META-INF里），那当我启动时我会去扫描所有jar包里符合约定的类名，再调用forName加载。但我的ClassLoader是没法加载的，那就把它加载到当前执行线程的线程上下文类加载器里，后续你想怎么操作就是你的事了。

[for more information~](https://blog.csdn.net/xmtblog/article/details/118947643)



#### 4.2.3 Tomcat怎么打破双亲委派？（实现了代码多版本共存隔离）

- <font color='red'>**目的：**</font>不同的war包，要加载相同类名，但是不同jar包版本的**类**
- 众所周知，Tomcat是web容器，一个web容器可能需要部署多个应用程序。
- 但是，不同的应用程序可能会依赖同一个第三方库的不同版本，但是不同版本的库中某一个类的全路径名可能是一样的。
  - **比如，**有多个应用都要依赖dabin.jar，但是A应用需要依赖1.0.0版本，但是B应用需要依赖1.0.1版本。这两个版本中都有一个类是com.dabin.xxx
  - 上面这个可能不好理解，换到Spring中：一个Tomcat web容器部署了两个web应用程序（war包），然后他们都依赖Spring。但是第一个war包依赖4.x版本的Spring，第二个依赖5.x的Spring；假如这两个包依赖的spring中有相同的类（比如String）要加载，那按双亲委派的话，要么加载4.x，要么加载5.x（因为是从上往下加载），那另一个肯定就部署不了。
- 如果要用默认的双亲委派机制，他们其实加载的是同一个类（一个类的唯一性需要类加载器和类的全限定名）；
- 为了加载不同的类，Tomcat破坏了双亲委派原则，提供了隔离的机制，为每个web容器单独提供一个WebAppClassLoader加载器。
  - 当然，这里多说一句，虽然Tomcat允许这么操作去加载类，**但是一些核心类，比如object、String类等，即使我们重写了loadClass方法，java的沙箱机制会阻止我们去加载自定义的这些类。**（这同样适用于所有的自定义loadClass）
    - 比如，自己床架一个java.lang.String类，然后在里面写一个main方法。运行一下，就会报错~，提示找不到main方法。
- 每个应用都有自己的类加载器WebAppClassLoader，该加载器重写了loadClass方法，会优先加载当前应用目录下的类，加载不到时在交给WebAppClassLoader的父加载器ShardClassLoader去加载（反过来了）。

[**总结：**](https://blog.csdn.net/qq_33591903/article/details/120088757)

- 双亲委派机制，核心是子加载器委托父加载器，能够避免java核心类库被篡改，增加了安全性。
- 但发展会带来创新，创新就会带来变革，jdbc与tomcat打破了这个自古相传的机制。
- 在jdbc中，父加载器委托子加载器。即利用线程上下文类加载器，让启动类加载器得以委托应用类加载器，去加载jar中的数据库驱动。
- 在tomcat中，子加载器优先于父加载器加载。即为了实现各个webapp的隔离性，webappClassLoader会先于父加载器加载。



[具体的实现，b站视频](https://www.bilibili.com/video/BV1ip4y1n7Gn?share_source=copy_web)

## 5、什么是面向对象编程？

面向过程的编程语言包括：C、Fortran、Pascal、Basic等。

看一个简单的例子：

咱们以把大象放进冰箱为例，**面向过程**的方式分为三步：

1.打开冰箱

2.把大象放进冰箱

3.关闭冰箱

**面向对象**是指围绕数据或对象而不是功能和逻辑来组织软件设计，更专注于对象与对象之间的交互，对象涉及的方法和属性都在对象内部。本质上，**面向对象是一种依赖于类和对象概念的编程方式。**而面向过程更像是依赖于方法的编程方式。

**面向对象的编程语言包括：C++、Java、Python、C#以及JavaScript等。**

**类：**是相同种类对象的抽象，是它们的公共属性。

**对象：**对象是类的实例。

面向对象将一个事物描述为一个对象，这个对象包括各种属性和方法，例如上面把大象装进冰箱的例子，冰箱的属性包括：长、宽、高、温度等，方法有：打开、关闭、存储等，这些属性和方法都属于这个对象。

同样，也把大象放进冰箱为例进行说明，面向对象的方式为：

1. 冰箱是一个对象，大象也是一个对象。
2. 冰箱有自己的方法，打开、存储、关闭，有自己的属性：长、宽、高等。
3. 大象也有自己的方法，吃、走路等，有自己的属性：体重、高度、体积等。
4. 我们创建这两个对象，然后调用三个方法（打开冰箱，放进大象，关闭冰箱）

下面向过程和面向对象的区别，如下所示：

（1）安全性

面向对象比面向过程安全性更高，面向对象将数据访问隐藏在了类的成员函数中，而且，类的成员变量和成员函数都有不同的访问属性。而面向过程并没有合适的方法来隐藏程序数据。

（2）程序设计上

面向过程通常将程序分为一个个的函数，而面向对象编程中通常使用一个个对象来，函数通常是对象的一个方法。

（3）过程

面向过程通常采用自上而下的方法，而面向对象通常采用自下而上的方法。

（4）程序修改

面向对象编程更容易修改程序，更容易添加新功能。

## 6、说一下对象实例化的过程？

> 了解了类的加载过程，其实就应该了解对象实例化（创建）的过程。
>
> Java堆是被所有线程共享的一块内存区域，主要用于存放对象实例。
>
> 对于Java开发来说，在虚拟机内存管理的帮助下，**不需要为每个新的对象在代码层面分配内存**，回收内存，比如像C语言那样操作。所以在正常情况下，内存泄露和内存溢出等问题也不太容易出现。

简单的说`instance = new SingletonDemo5();`并不是一个原子操作，其实就是三步：

1. 先malloc，在堆内存中分配一个内存空间；
2. `ctorInstance(memory);`；初始化对象
3. 把新建的对象实例指向这个内存空间：

但是这个过程还是很复杂的：

1. JVM先检查目标对象是否已经被加载并初始化
2. 如果没有，就立刻去加载目标类，并调用目标类的构造器去完成初始化
3. 而目标类的加载是通过类加载器来实现的（主要就是把一个类加载到内存里面）
4. 然后是初始化的过程：主要是把类中的静态变量，成员变量，静态代码块进行初始化
5. 这个时候在方法区就已经可以找到类元信息和常量信息等；并且，目标对象的大小在类加载完成之后也确认了；
6. 然后为新创建的实例对象根据目标对象的大小，在堆内存中去分配内存空间。内存分配的方式有两种（根据内存片是否完整）：指针碰撞和空闲列表（**4-6：开辟空间，并且给一些基础类型变量赋上“零值”**）
7. JVM对对象头进行设置，包括：对象所属的类元信息，对象的GC分代年龄，hashcode，锁标记等**（到此，JVM对于新对象的创建工作已经结束）**
8. 然后是，Java就要开始执行目标对象内部生成的init方法，初始化成员变量的值，执行构造块，最后调用目标对象的构造方法（也包括父类的构造方法）完成对象的创建



### 6.1 为对象分配内存的两种方式

为对象分配内存就是把一块大小确定的内存从堆内存中划分出来，通常有指针碰撞和空闲列表两种实现方式：

1. 指针碰撞：**假设Java堆中内存时完整的**，已分配的内存和空闲内存分别在不同的一侧，通过一个指针作为分界点，需要分配内存时，仅仅需要把指针往空闲的一端移动与对象大小相等的距离。使用的GC收集器：Serial、ParNew，适用堆内存规整（即没有内存碎片）的情况下。

2. 空闲列表：事实上，**Java堆的内存并不是完整的，**已分配的内存和空闲内存相互交错，JVM通过维护一个列表，记录可用的内存块信息，当分配操作发生时，从列表中找到一个足够大的内存块分配给对象实例，并更新列表上的记录。**使用的GC收集器：CMS，适用堆内存不规整的情况下。**

   

### 6.2 [内存分配并发问题](https://blog.csdn.net/qq_44346427/article/details/116163250)

在创建对象的时候有一个很重要的问题，就是线程安全，因为在实际开发过程中，创建对象是很频繁的事情，作为虚拟机来说，必须要保证线程是安全的，通常来讲，虚拟机采用两种方式来保证线程安全：

1. CAS： CAS 是乐观锁的一种实现方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。虚拟机采用 <font color='red'>CAS + 失败重试</font>的方式保证更新操作的原子性
2. TLAB： **为每一个线程预先分配一块内存，**JVM在给线程中的对象分配内存时，首先在TLAB分配，当对象大于TLAB中的剩余内存或TLAB的内存已用尽时，再采用上述的CAS进行内存分配

# 负二、Java基础（基础中的基础！！！）

> 这一节梳理一下Java最基础的一些性质，也就是课本上的知识，比如Java特性、集合、链表等

## 1、Java的特性——封装、继承、多态

### 1.1 封装(Encapsulation) 

- 隐藏对象的**属性和实现细节**，仅仅对外公开接口。
  - 提高安全性（防止使用者错误修改系统属性）
  - 提高重用性
  - 将变化隔离

- 最直观的操作：
  - 将成员变量私有化，对外提供对应的set ， get方法对其进行访问。提高对数据访问的安全性。

### 1.2 继承

- 继承是从已有的类中派生出新的类， 新的类能吸收已有类的数据属性和行为，并能扩展新的能力。
- **一个需要注意的地方，**当子类继承父类时，类加载的顺序：
  1. 父类的静态代码块（静态变量）——子类的静态代码块（静态变量）——子类的主程序（静态方法）——父类静态方法
  2. 父类的有参构造函数——父类的方法——父类的无参构造——子类的非静态代码块——子类的有参构造——子类重写了父类的方法。

### 1.3 多态

- 在面向对象语言中， 多态性是指一个方法可以有多种实现版本，即“一种定义， 多种实现”
- 类的多态性表现为方法的多态性，方法的多态性主要有**方法的重载（不同的传参）和方法的覆写（override）**
- **instanceof**关键字是用来判断其左边对象是否为其右边的实例， 返回boolean类型的数据

### 1.4 继承和多态的区别？

继承和多态很像，他们本质上都是进行代码复用，提高代码的可扩展性和维护性。

- 继承的作用是体现多态

- 要有多态，首先得有继承，子类继承父类之后，才能对父类的方法进行重写
- 总结：**继承**是不同的类有相同的特性、相同的实现（所以子类要继承父类）等；**多态**是相似的类有不同的特性或同一特性有不同的实现（对同一个方法，进行不同的实现）

## 2、Java的集合——Collection（单列集合）+Map接口（双列集合key-value）

### 2.1 Collection 接口——List、Set、Queue

1. List元素按进入先后**有序**保存，**可重复**
   - ArrayList：数组， **随机访问**， 没有同步， 线程不安全
   - LinkedList：链表，**插入删除**，没有同步， 线程不安全
   - Vector：数组， 同步， **线程安全**
     - 和ArrayList有一样的性质
     - Stack 是Vector类的实现类
2. Set元素，仅接收一次，**无序，不可重复**（ 唯一性都是通过重写hashcode和equals方法），并可以做内部排序
   - HashSet：使用hash表（数组）存储元素
     - LinkedHashSet ：底层数据结构采用链表和哈希表共同实现，链表保证了元素的顺序与存储顺序一致，哈希表保证了元素的唯一性。
     - 可以放入null，但只能放入一个null。
   - TreeSet：底层实现为二叉树，**元素排好序**
     - 不允许放入null值
     - 在我们需要排序的功能时，我们才使用TreeSet（基本不怎么用）。
3. Queue
   - DeQueue(Double-ended queue)为接口：继承了Queue接口，创建双向队列，灵活性更强，可以前向或后向迭代，在队头队尾均可心插入或删除元素。
   - 它的两个主要实现类是ArrayDeque和**LinkedList**。
   - PriorityQueue：
     - 底层用**数组**实现堆的结构
     - PriorityQueue不是一个线程安全的类，如果要在多线程环境下使用，可以使用 PriorityBlockingQueue 这个优先阻塞队列。其中add、poll、remove方法都使用 ReentrantLock 锁来保持同步，take() 方法中如果元素为空，则会一直保持阻塞。

### 2.2Map接口——Hashtable、HashMap、TreeMap、~~IdentifyHashMap~~

Map数据结构，都必须实现hashCode和equals方法！！

**它的底层实际上由数组和链表来组成的**：举个例子，使用put方法时,先查数组位置是否为对象,通过key.hashcode对数组长度取余; 存在,则把里面的链表拿出来,判断链表里面是否存在key值相互匹配的对象, 如果存在就将查到的key值对应的value替换,不存在则通过链表的add()方法直接加在链表后面

1. Hashtable 接口实现类， **同步， 线程安全**

   - hashtable的key、value不可以为null

2. HashMap 接口实现类 ，没有同步， **线程不安全**

   - HashMap对象的key、value值均可为null

   - **LinkedHashMap：双向链表和哈希表实现**
   - WeakHashMap

3. TreeMap：红黑树，对所有的key进行排序
4. IdentifyHashMap：它允许"自己"相同的key保存进来，因此又一个相同二字。他和HashMap的区别：
   - 对于要保存的key，k1和k2，当且仅当k1== k2的时候，IdentityHashMap才会相等，而对于HashMap来说，相等的条件则是：对比两个key的hashCode等
   - dentityHashMap**不是Map的通用实现**，它有意违反了Map的常规协定。并且IdentityHashMap允许key和value都为null。
   - 该类不是线程安全的，如果要使之线程安全，可以调用**Collections.synchronizedMap(new IdentityHashMap(…))**方法来实现。（这里面试官可能会问？那怎么实现一个线程安全的identityHashMap(HashMap)？）

用的最多的就是1和2，4可以作为扩展说一下。



### 2.3 ArrayList和LinkedList的区别？（补充一个和Vector的区别）

- Arraylist：
  - 优点：ArrayList是实现了基于动态数组的数据结构,因为地址连续，一旦数据存储好了，查询操作效率会比较高（在内存里是连着放的）。
  - 缺点：因为地址连续， ArrayList要移动数据,所以插入和删除操作效率比较低。
- LinkedList：
  - 优点：LinkedList基于链表的数据结构,地址是任意的，对于新增和删除操作add和remove，LinedList比较占优势。LinkedList 适用于要头尾操作或插入指定位置的场景
  - 缺点：因为LinkedList要移动指针,所以查询操作性能比较低。

这里可以补充一个**ArrayList与Vector的区别和适用场景**。因为ArrayList和Vector都是用数组（即，都是list）实现的，主要有这么三个区别：

- Vector是多线程安全的，这个可以从源码中看出，Vector类中的方法很多有synchronized进行修饰，这样就导致了Vector在效率上无法与ArrayList相比（但是Vector仍然是连续空间）
- 两个都是采用的线性连续空间存储元素，但是当空间不足的时候，两个类的**增加方式不同**。（怎么增加？vector增长率为目前数组长度的100%,而arraylist增长率为目前数组长度 的50%）
- 在集合中使用数据量比较大的数据，用Vector有一定的优势。
- Vector可以设置增长因子，而ArrayList不可以。

### 2.4 ArrayList的扩容机制？怎么扩？

这种问题，都是两步走！先分析初始化的情况，然后分析扩容时候的情况。 

第一次创建的时候：

- 如果第一次添加数据的话那么数组扩容长度为DEFAULT_CAPACITY=10（默认）
- 可以生成一个带数据的ArrayList这边不再赘述
- 自定义初始容量：可以初始化为0，出现在需要用到空数组的地方。（很少）

有添加数据的操作上面都要需要判断当前数组容量是否足以容纳新的数据，如果不够的话就需要进行扩容！！！

ArrayList扩容的核心方法grow()，下面将针对三种情况对该方法进行解析：

- 默认为10，然后一个原来数组的长度加上原来数组的长度0.5进行扩容（**通过原来的数组长度加上一个右移运算符“oldCapacity >> 1”**），对于扩容的源码就是ensureExplicitCapacity这个方法中的**grow**；
- 初始化为0，前4次都是+1，第五次才1.5倍扩容
- 当扩容量（newCapacity）大于ArrayList数组定义的最大值后会调用hugeCapacity来进行判断。如果大于ArrayList允许的最大容量（-2^31~2^31-1）,内存溢出报错

<font color='red'>**注意：**</font>

Java容器有一种**保护机制，**能够防止多个进程同时修改同一个容器的内容。如果你在迭代遍历某个容器的过程中，另一个进程介入其中，并且插入，删除或修改此容器的某个对象，就会立刻抛出ConcurrentModificationException。**(多线程安全)**

**哪里会出现？**

- 在迭代遍历（使用迭代器Iterator(ListIterator)或者forEach（注意这里不是for循环的意思）语法）的过程中都调用了方法checkForComodification来判断当前ArrayList是否是同步的
- 所谓的不同步指的就是，如果你在遍历的过程中对ArrayList集合本身进行add,remove等操作时候就会发生。

**面试：如何保证ArrayList在多线程环境下的线程安全性？**

- 如果你在多线程中使用CopyOnWriteArrayList就可以避免了。
- 用Iterator那么使用它的remove是允许的因为此时你直接操作的不是ArrayList集合而是它的Iterator对象。
- 使用synchronized来同步所有的ArrayList操作方法（List<String> list = new Vector<>();用Vector类来实现，它的add()方法加了synchronized关键字修饰，所以能保证线程安全。）
- Collections.synchronizedList()方法其实底层也是在集合的所有方法之上加上了synchronized。（List<String> list = Collections.synchronizedList(new ArrayList<>());）

### 2.5 什么是CopyOnWriteArrayList？为什么能保证ArrayList的线程安全

Copy On Write 也是一种重要的思想，在写少读多的场景下，为了保证集合的线程安全性，我们完全可以在当前线程中得到原始数据的一份拷贝，然后进行操作（如果不是读多写少，对于更新操作，开销还是很大的）。`List<String> list = new CopyOnWriteArrayList<>();`

- CopyOnWriteArrayList和CopyOnWriteSet都是线程安全的集合，其中所有可变操作（add、set等等）都是通过对底层数组进行一次新的复制来实现的。
- 它适合使用在读操作远远大于写操作的场景里，比如缓存。它不存在“扩容”的概念
- 写时复制，能够做到读写分离，保证写的线程安全且支持并发读。

其实这个问题，本质上只要理解了什么是“[写时复制”就行了。](https://blog.csdn.net/weixin_42146366/article/details/88016527)（里面有java操作代码）

- 每次写操作（add or remove）都要copy一个副本，在副本的基础上修改后改变array引用，所以称为“CopyOnWrite”，因此**在写操作是加锁**，并且对整个list的copy操作时相当耗时的，过多的写操作不推荐使用该存储结构。**读操作无锁**。

CopyOnWriteArrayList存在的问题：

1. 由于写操作的时候，需要拷贝数组，会消耗内存，如果原数组的内容比较多的情况下，可能导致young gc或者full gc。
2. 不能用于实时读的场景，像拷贝数组、新增元素都需要时间，所以调用一个set操作后，读取到数据可能还是旧的,虽然CopyOnWriteArrayList 能做到**最终一致性**（底层用到是可重入锁REentrantLock，volatile关键字，强迫线程将变化值回写到共享内存）,但是还是没法满足实时性要求

## 3、Java的构造方法有什么用

> 之前一直对这个概念比较模糊，[这篇文章讲得很清晰](https://zhuanlan.zhihu.com/p/485785681)
>
> 先直观地理解一下，构造方法的作用：**初始化对象的内部状态**。也就是给对象的各个属性赋初始值（无参构造就相当于赋null和0值）。
>
> 那一般什么时候用到呢？一般是new 创建一个对象的时候。

### 3.1构造方法的三个特点

1. 构造方法的名称必须与类的名称相同。比如类的名称叫A，那么它的构造方法必须也叫A。
2. 构造方法的前面**不能声明返回值类型**，即便是void也不行。只有满足了这两个条件，编译器才会认定这个方法是构造方法。
   - 因为普通的方法也可以和类同名，但是它是有返回值的
3. 如果程序员没有在类中定义构造方法，那么在编译阶段，编译器会“免费赠送”给这个类一个构造方法，也就是说，编译器会在编译阶段在字节码文件中补充添加一个构造方法。

```java
//一个常规的无参构造
class Person
{
	String name;
	int age;
	void printInfo(){
		System. out.println("我叫"+name+" ,我今年" +age+"岁");
	}
}
/*
输出：我叫null，我今年0岁
*/

//一个常规的有参构造
class Person
{
	String name;
	int age;
	//定义一一个构造方法并且在构造方法中给属性赋值
	public Person(){
	name="张三";
	age = 20;
	}
	void printInfo(){
		System. out.println("我叫"+name+" ,我今年" +age+"岁");
	}
}
/*
输出：我叫张三，我今年20岁
*/
```

但是问题来了：按照这样的写法，我们所创建的每个Person对象，name属性都被赋值为“张三”，而age属性都被赋值为20。这说明我们创建的对象是“千篇一律”的，并且从情理上也说不通，毕竟每个人都有属于自己的名字和年龄，不可能每个人都叫张三年龄20岁。（也就是失去了父类的扩展的意义）

并且，一旦定义了有参构造方法，默认的无参构造就会失效。

### 3.2 this和super方法

父类定义了有参构造方法，并且允许子类通过this方法注入：

```java
//父类的有参构造
public Person(String name,int age){
	this.name = name;
	this.age = age;
}
//子类的调用方式
Person p1 = new Person("张三" ,20);
Person p2 = new Person("李四" ,21);
```

这样，我们就能够按照我们的意愿，创建出有自己个性化的对象了。所以，**构造方法的作用是给对象的各个属性赋上合理的初始值，从而使得我们所创建的对象不再是“千篇一律”，而是“千姿百态”。**

思考两个问题：

1. **构造方法可以像普通方法那样实现重载吗？**没有问题。可以在一个类中定义多个构造方法，只要这些构造方法参数不同，就构成了重载。
2. 在一个构造方法中，可以调用另外一个构造方法吗？没有问题。但调用的时候，需要注意：不能像调用普通方法那样，通过类名去调用，而是需要用一个关键字this。也有两个条件：
   1. 调用构造构造方法的语句必须放在第一行
   2. 两个构造方法不能形成相互调用关系。为了方便表述，我们把一个类中的两个构造方法代称为X和Y。如果我们在X中调用了Y，那么就不能在Y中去调用X了。否则就会形成循环依赖关系

```java
class Person
{
    String name;
    int age;
    public Person(String name){//-个参数的构造方法
    this.name = name;
    }
    public Person(String name,int age){/两个参数的构造方法
    this(name);//调用另一个构造方法,调用语句写在方法第一行
    this.age = age;
    void printInfo({
    System. out.println("我叫" + name+" ,我今年" +age+ "岁");
}
```

我们在一个构造方法中调用了另一个构造方法，调用的时候，需要用到this关键字，并且把调用语句写到第一行，这样才能顺利通过编译。

#### super()

> 如果我们创建的是一个子类的对象，那么在创建这个子类对象的时候，虚拟机会先调用父类的构造方法，之后才去调用子类的构造方法。

一下子类调用父类构造方法的语法细节：

1. 子类调用父类构造方法的时候，不能通过构造方法本身的名称来调用，必须使用super关键字
2. 子类在它的普通方法中不能调用父类的构造方法，只能在它自身的构造方法中才能调用
3. 子类调用父类构造方法的语句，必须写在自身构造方法的第一行

问题：

1. 父类如果有好几个构造方法，编译器会自动调用哪一个呢？这里必须明确：编译器只会调用那个无参数的构造方法，不会调用有参数的构造方法。
2. 这个规则又会引发一个新的问题，那就是：如果父类中压根就没有无参数的构造方法，那怎么办？在这种情况下，编译器就会强制子类定义一个构造方法（写明是super(xxx,yyy)），并且在它的构造方法中，通过手动调用的形式去调用父类的有参构造方法



# 一、HashMap

考HashMap，一定要复习到与之相似的另外两个数据结构：HashTable、ConcurrentHashMap。

## 1、原理

**HashTable**

- 底层数组+链表实现，无论key还是value都**不能为null**，线程**安全**，
- **实现线程安全的方式是Synchronized锁。**在修改数据时锁住整个HashTable，效率低，ConcurrentHashMap做了相关优化
- 初始size为**11**，扩容：newsize = olesize*2+1
- 计算index的方法：index = (hash & 0x7FFFFFFF) % tab.length

**HashMap**

- 底层**数组+链表**实现，可**以存储null键和null值**，线程**不安全**
- 初始size为**16**，扩容：newsize = oldsize*2，size一定为2的n次幂
- **hash冲突时**：先使用链表解决冲突--->链表的长度到8，则使用红黑树
- 扩容针对整个Map，每次扩容时，原来数组中的元素依次重新计算存放位置，并重新插入
- 插入元素后才判断该不该扩容，有可能无效扩容（插入后如果扩容，如果没有再次插入，就会产生无效扩容）
- 当Map中元素总数超过Entry数组的**75%（这个就是默认的负载因子）**，触发扩容操作，为了减少链表长度，元素分配更均匀
- 计算index方法：index = hash & (tab.length – 1)；
  - 这里有一个题目？为什么要用按位与操作，其实是用为当length为2^n时，按位与就相当于hash%length操作，而且与操作就是计算机的低层操作。

**ConcurrentHashMap**

- 底层采用分段的数组+链表实现，线程**安全**
- **通过把整个Map分为N个Segment（字段，是一个可重入锁），**可以提供相同的线程安全，但是效率提升N倍，默认提升16倍。(读操作不加锁，由于HashEntry的value变量是 volatile的，也能保证读取到最新的值。)
- **Hashtable的synchronized是针对整张Hash表的，即每次锁住整张表让线程独占，ConcurrentHashMap允许多个修改操作并发进行，其关键在于使用了锁分离技术**
- 有些方法需要跨段，比如size()和containsValue()，它们可能需要锁定整个表而而不仅仅是某个段，这需要按顺序锁定所有段，操作完毕后，又按顺序释放所有段的锁
- 扩容：段内扩容（段内元素超过该段对应Entry数组长度的75%触发扩容，不会对整个Map进行扩容），插入前检测需不需要扩容，有效避免无效扩容

问题：

## 2、为什么HashMap使用高16位异或低16位计算Hash值？

主要原因是保留高16位与低16位的特性，增大散列程度；

hash值其实是一个int类型，二进制位为32位，而HashMap的table数组初始化size为16，取余操作为hashCode & 15 ==> hashCode & 1111 。这将会存在一个巨大的问题，1111只会与hashCode的低四位进行与操作，也就是hashCode的高位其实并没有参与运算，会导很多hash值不同而高位有区别的数，最后算出来的索引都是一样的

## 3、hashMap的put操作流程？

首先就是寻址算法嘛（就是key的hashcode的高低位异或之后，和数组长度-1进行按位与，得到一个槽位（slot）下标

根据这个slot的状态，有四种情况：

 	1. slot==null，没有hash冲突，直接插入（把key，value包装成一个node对象，放进去）
 	2. slot为单节点：hash冲突，变成链表（尾插法）；没冲突（slot的key和新的node的key完全相等，直接替换）
 	3. slot为链表，hash冲突，在链表的尾部追加节点
 	4. slot为红黑树，将节点插入红黑树

## 4、ConcurrentHashMap 的锁分离技术（为什么不用HashTable的原因）
ConcurrentHashMap是使用了锁分段技术来保证线程安全的。

- 首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。

  - Hashtable中采用的锁机制是一次锁住整个hash表（Synchronized锁），从而在同一时刻只能由一个线程对其进行操作；而ConcurrentHashMap中则是一次锁住一个桶，它的锁粒度更细，本质上用的是CAS操作+Synchronized锁（jdk1.6之后增加了锁升级机制）。

- ConcurrentHashMap默认将hash表分为16个桶，诸如get、put、remove等常用操作只锁住当前需要用到的桶。这样，原来只能一个线程进入，现在却能同时有16个写线程执行，并发性能的提升是显而易见的。

- 其实可以看出JDK1.8版本的ConcurrentHashMap的数据结构已经接近HashMap，**相对而言，ConcurrentHashMap只是增加了同步的操作来控制并发**


### 5.1 ConcurrentHashMap的锁升级机制？

> ConcurrentHashMap的锁只会锁住目前获取到的那个Entry所在的那个节点，上锁的时候用的是CAS操作+Synchronized锁，在加上JDK1.6之后引入的锁升级机制，对Synchronized锁进行了一个优化，效率更高，并发度更高。
>
> 本质就是JDK1.6之后引入的**锁升级机制**，这个其实涉及到了并发编程部分，可以参考本博客的`二、并发编程 - 7、锁升级机制`。



## 5、ConcurrentHashMap的分段hash？

第一次Hash定位到Segment，第二次Hash定位到元素所在的链表的头部。

这一种结构的带来的副作用是Hash的过程要比普通的HashMap要长。

写操作的时候可以只对元素所在的Segment进行加锁即可，不会影响到其他的Segment，这样，在最理想的情况下，ConcurrentHashMap可以最高同时支持Segment数量大小的写操作(刚好这些写操作都非常平均地分布在所有的Segment上)。

## 6、JDK1.8之后的ConcurrentHashMap的改变？

1. 数据结构：取消了Segment分段锁的数据结构（Segment 通过继承 ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 Segment），取而代之的是数组+链表+红黑树的结构。

2. **保证线程安全机制：**JDK1.7采用segment的分段锁机制实现线程安全，其中segment继承自ReentrantLock。JDK1.8采用CAS+Synchronized保证线程安全。

3. **锁的粒度：**原来是对需要进行数据操作的Segment加锁，现调整为对每个数组元素加锁(Node)。

4. 链表转化为红黑树：定位结点的hash算法简化会带来弊端,Hash冲突加剧,因此在链表节点数量大于8时，会将链表转化为红黑树进行存储。查询时间复杂度从原来的遍历链表O(n)，变成遍历红黑树O(logN)。

对比HashMap的优势：

1. **更优的扩容：**ConcurrentHashMap引入了多线程并发扩容的机制：简单而言，就是多个线程对于原数组进行分片（得益于更细的锁粒度）。分片之后，每个线程去分别进行数据迁移。

2. **更快的size()计算：**原始的size()用来计算总的元素个数，在多线程场景中，因为要保证线程安全，就是在你计算 size 的时候，它还在并发的插入数据，可能会导致你计算出来的 size 和你实际的 size 有差距。
   1. **JDK1.7 和 JDK1.8 对 size 的计算是不一样的。** 1.7 中是先不加锁计算三次，如果三次结果不一样在加锁。
   2. JDK1.8 size 是通过对 baseCount 和 counterCell 进行 **CAS 计算，**最终通过 baseCount 和 遍历 CounterCell 数组得出 size。(volatile类型的变量**baseCount记录元素的个数**；初始化时counterCells为空，在并发量很高时，如果存在两个线程同时执行CAS修改baseCount值，则失败的线程会继续执行方法体中的逻辑，**使用CounterCell记录元素个数的变化；**)——一个是个数，一个是个数的变化，这个不难理解吧？？——网上有一些教程，说根据线程竞争激不激烈来判断到底是CAS还是CountCells中取值，这个是不准确的！！因为你判断不了，我想他的意思是，CAS操作反映了一部分的竞争激不激烈？（激烈的时候，肯定就有大量的线程在自旋！）
   3. JDK 8 推荐使用mappingCount 方法，因为这个方法的返回值是 long 类型，不会因为 size 方法是 int 类型限制最大值。



## 8、HashMap的扩容机制？

扩容是懒加载类型的：put的时候确定是否扩容，扩容时候直接生成一个容量为2倍的数组，然后进行数据迁移。

影响发生Resize的因素有两个**（要同时成立）:**

- 当前数据存储的数量（HashMap.size） >= Capacity*LoadFactor 时，HashMap**可能会**进行Resize。
- 当前加入的数据发生了hash冲突

考虑如下情况：

1）就是hashmap在存值的时候（默认大小为16，负载因子0.75，阈值12），可能达到最后存满16个值的时候，再存入第17个值才会发生扩容现象，因为前16个值，每个值在底层数组中分别占据一个位置，并没有发生hash碰撞。

2）当然也有可能存储更多值（超过16个值，最多可以存26个值）都还没有扩容。原理：前11个值全部hash碰撞，存到数组的同一个位置（这时元素个数小于阈值12，不会扩容），后面所有存入的15个值全部分散到数组剩下的15个位置（这时元素个数大于等于阈值，但是每次存入的元素并没有发生hash碰撞，所以不会扩容）

- <font color='red'>hashmap还有一种情况会导致扩容,就是链表长度大于6,且这个数组长度小于64,这时候不转为红黑树,触发扩容</font>

但是可以看到扩容机制对于效率影响还是很大的，所以有一种思路就是在初始化时，就设定数组的容量 ，以免反复扩容。

## 9、HashMap扩容时候的数据迁移？

这个根据HashMap的slot的格式（链表或者红黑树）内容，迁移规则有点不一样。

- slot为链表：说明存在hash冲突，就要把这个表拆成低位链和高位链，扩容位置也是不一样的。（高位链存放的：老表的位置+新表的长度=新位置；低位链就是老表的位置）——比如，121和131的低2位是不一样的，121为0，131为1，于是我们将高位为1的节点称为高位节点
- slot为红黑树：红黑树是TreeNode结构，他依然保留了next字段（在插入和删除的时候会用到；查询用不到）。为的就是方便去拆分这棵树，他也是把树按高位和低位去拆分成高低位。如果拆分出来的长度是<=6的，就是普通链表；如果大于6，就要重新生成红黑树。（也是用高低位来加速迁移，也不是说加速，其实也就是一个规则）

### 9.1 为什么HashMap采用尾插法？头插法为什么会产生循环的问题？尾插法为什么没有？

首先，这个问题的发生是有条件的，场景限制：

1. 我们是在并发执行的时候（也是因为HashMap线程不安全导致）
2. 同时数组出现扩容，并且扩容以后，原来链表上的值有映射到同一个数组位置时
3. 并且后面调用get()方法时可能陷入死循环。

过程：（用例子说明）

1. 假设有T1，T2两个线程同时对一个HashMap进行put操作，刚好，HashMap达到了扩容的条件， 这是两个线程都会去对这个HashMap进行扩容。
2. 链表A-->B->C-D，假设扩容后，计算的index依然相同，那么他们还会存放在同一链表中
3. 一开始是A->B->C-D,第一个线程T1获取了node=A，获取了下个节点next = A.next是B做下次循环，此时线程被挂起（但是node、next节点它还保存着）——这个时候T1还没任何操作
4. 第二线程哒哒跑完了整个流程，此时链表变成了D->C->B->A(假设扩容后都在同个桶里)
5. 接着第一个线程继续跑，A跑完，A->NULL（因为按头插法，A现在就应该在结尾）,下一个之前获取是B（~~本来这时候A下一步应该是Null,但是链表已经被上一个线程修改了，但是B是改之前获取~~）
6. T1对B跑完，此时变成B->A->NULL,b.next就是A了（重点）
7. 然后T1，会接着B->A->NULL前进，指向A。根据头插法，A把next指针又指向了B，此时变成A->B->A，形成了环。然后根据头插法，插完A之后，也有推出了
8. 最后有新的值，进来get()，如果访问到环，就死循环了。



产生循环链表后带来的问题是什么?

- 环形链表已经产生了, 当我们调用get(3)或者get(1)不会产生问题,
  但是如果get(5), 并且5在数组中的下标和1,3的一致的话, 由于链表中没有5, 所以就会一直在链表中寻找, 但是链表没有尽头, 就导致程序卡在get(5)处了

为什么尾插法不出现问题？

1. 是扩容后旧数组往新数组迁移的时候，由于头插法导致了新数组的链表顺序和原来是相反的，线程在做链表循环的时候又循环到前面的值，形成了一个环；
2. 而尾插法不会改变原有的next指向关系，所以他不会出现环



## 10、HaspMap的空值问题？

- key可以为null，但是只能有一个；
- value也可以为null，个数无限制。

PS：TreeMap是一个有序的key-value集合，它是通过红黑树实现的。

- 基于红黑二叉树的NavigableMap的实现，**线程非安全，不允许null，key不可以重复，value允许重复**，存入TreeMap的元素应当实现Comparable接口或者实现Comparator接口，会按照排序后的顺序迭代元素，两个相比较的key不得抛出classCastException。

- 主要用于存入元素的时候对元素进行自动排序，迭代输出的时候就按排序顺序输出




## 11、为什么链表到8才扩容为红黑树？小于6又退化成链表？

- 通常如果 hash 算法正常的话，那么链表的长度也不会很长，那么红黑树也不会带来明显的查询时间上的优势，反而会增加空间负担。所以通常情况下，并没有必要转为红黑树，所以就选择了概率非常小，小于千万分之一概率，也就是长度为 8 的概率，把长度 8 作为转化的默认阈值。（一般达不到）
- 所以，链表长度超过 8 就转为红黑树的设计，更多的是为了防止用户自己实现了不好的哈希算法时导致链表过长，从而导致查询效率低，而此时转为红黑树更多的是一种保底策略，用来保证极端情况下查询的效率。（避免自己设计的hashcode不够随机）

## 12、红黑树

### 12.1 五大基本定义

1. 结点必须是红色或者黑色。（父子不能同为红）
2. 根节点必须是黑色。
3. 叶节点(NIL)必须是黑色(NIL节点无数据，是空节点)。（为了简单起见，一般会省略该节点）
4. 新插入节点默认为红色，插入后需要校验红黑树是否符合规则，不符合则需要进行平衡。
5. 从任一节点出发到其每个叶子节点的路径，黑色节点的数量必须相等。（平衡的关键）

本质上：

- ​	所以红黑树是一个**大致平衡的二叉树。**跟AVL树不同，红黑树并不是严格平衡的，而AVL树却是严格平衡的。这5个性质决定了从根节点到叶子节点的最长路径不可能大于最短路径的2倍。

- 红黑是用非严格的平衡来换取增删节点时候旋转次数的降低，任何不平衡都会在三次旋转之内解决，而AVL是严格平衡树，因此在增加或者删除节点的时候，根据不同情况，旋转的次数比红黑树要多。（为什么不用平衡二叉树的原因？？？）

TreeNode={left，right， 颜色，next，parent}——next节点一般大家都不去管，它是用来横向寻找到，相当于是一个链式。



### 12.2 红黑树的插入操作

找到合适的插入点，以及他的父节点（https://www.cnblogs.com/gejuncheng/p/9081886.html）

首先约定插入的新节点的颜色都为红色。然后将该节点插入的按二叉查找树的规则插入到树中。这个节点后文称为N

1. 根节点为空。这种情况，将N的颜色改为黑色即可。

2. N的父节点为黑色。这种情况不需要做修改。

3. N的父节点为红色（根据性质3，N的祖父节点必为黑色）。
   - N的叔父节点为红色。这种情况，将N的父节点和叔父节点的颜色都改为黑色，若祖父节点是跟节点就将其改为黑色，否则**将其颜色改为红色**，并以祖父节点为插入的目标节点从情况1开始递归检测。**（改颜色）**
   - N的叔父节点为黑色， 
     - 且N和N的父节点在同一边。以父节点为祖父节的左儿子为例，将父节点改为黑色，祖父节点改为红色，然后以祖父节点为基准**右旋。**（同边右旋）
     - 切N和N的父节点不在同一边（如父节点为祖父的左儿子时，N是父节点的右儿子。）。以父节点为基准，进行**左旋，**然后以父节点为目标插入节点进入情况3的b情况进行操作。（异边左旋）

### 12.3 红黑树的删除操作

- 在删除带有两个非叶子儿子的节点的时候，我们找到要么在它的左子树中的最大元素、要么在它的右子树中的最小元素，并把它的值转移到要删除的节点中。

所有情况都可以转化为删除只有一个儿子的节点的情况，我们约定这个要删除的节点为N：

1. N为红色节点时。直接删除N，用它的黑色儿子代替它的位置。
2. N为黑色节点
   - 且父节点为红色。直接删除N，用它的儿子节点代替它的位置，并将该儿子节点改为黑色。
   - 且父节点为黑色。直接删除N，用它的儿子节点N'代替它的位置。
     - N’的兄弟节点和兄弟节点的2个儿子都为黑色。交换兄弟节点和父节点的颜色即可。
     - N‘的兄弟节点为黑色、且兄弟节点的红色儿子和兄弟节点在一边(如兄弟节点为右儿子时，右儿子的儿子也是红色)，将祖父节点和兄弟节点的颜色互换，并将红色右儿子的颜色改为黑色，然后以祖父节点为基准**左旋**。（同侧左旋，反过来了；但是这是基于右右的情况，左左是右旋；）
     - N‘的兄弟节点为黑色、且兄弟节点的红儿子和兄弟节点不在一边，将兄弟节点和它的红色儿子的颜色互换，然后以兄弟节点为基准**右旋**。此时对于N’来说就进入了上文b情况。
     - N‘的兄弟节点为红色。以兄弟节点为右儿子为例，将父节点和兄弟节点的颜色互换，然后以父节点为基准左旋
     - N‘的兄弟节为黑色，父节点也为黑色。此时将兄弟节点的颜色改为红色。然后以父节点为目标插入节点从头开始依次判断。

**for more information**↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓

```
*旋转规则：https://blog.csdn.net/qq_38685503/article/details/103425572
```

```
//红黑树（Java实现）:https://blog.csdn.net/weixin_45567738/article/details/116567192
//java实现（有更详细的解析）：https://baijiahao.baidu.com/s?id=1687524393031712762&wfr=spider&for=pc
//Java实现红黑树的删除：https://www.cnblogs.com/steveshao/p/15045521.html
```

### 12.4、hashmap中用红黑树不用其他树？（二叉排序树、平衡二叉树、B/B+树？）

首先，HashMap一开始并没有红黑树。HashMap在jdk1.8的时候才引入红黑树。（当链表长度满足8，或者校验数组长度满足64，调用treeifyBin()的方法，转成红黑树）

1. 对比二叉排序树（左子树上所有结点的值 均小于 它的根结点的值；右子树上所有结点的值均大于它的根结点的值）。
   - 在二叉排序树在添加元素的时候极端情况下会出现线性结构
2. 对比平衡二叉树（一种二叉查找树，且两个子树的高度差不超过1）
   - 红黑树不追求"完全平衡"，即不像AVL那样要求节点的 |balFact| <= 1。增加一个颜色标记要求，任何不平衡都会在三次旋转之内解决，而AVL是严格平衡树，因此在增加或者删除节点的时候，根据不同情况，旋转的次数比红黑树要多。
     - 插入的时候，AVL和RBTree一样，都是最多两次
     - 删除失衡，AVL需要维护从被删除节点到根节点root这条路径上所有节点的平衡，旋转的量级为O(logN)，红黑树只需要3次。
3. **对比B/B+树**（这两种数据结构的特点就是树比较矮胖，每个结点存放一个磁盘大小的数据，这样一次可以把一个磁盘的数据读入内存，减少磁盘转动的耗时，提高效率。）——这一块很少人比较，面试可以额外提一下！
   - 红黑树多用于内存中排序，也就是内部排序。
   - 如果用B+树的话，**在数据量不是很多的情况下，**数据都会“挤在”一个结点里面。这个时候遍历效率就退化成了链表。（所以大家的场景不太一样）

同样的问题，我们也要问：为什么Mysql里面要用B+树，不用B树、二叉树、Hash表、红黑树呢？[请参考](https://www.jianshu.com/p/99aabf9611a3)，在本博客中，也有解答！（后面加入引用~）

### 12.5、红黑树的应用场景

1. C++中的map，set
2. Java中的HashMap，TreeMap，TreeSet
3. 用于实现Linux中的CPU调度，用红黑树管理进程控制块。这样在查找的PCB的时候，更快。
4. Unix的网络I/O，epoll在把文件描述符fd拷贝到内核时，也是红黑树





# 二、Java并发编程：volatile、synchronized、ThreadLocal、ReentranLock

Java 关键字volatile 与 synchronized 作用与区别

> 一个面试题：如何在不加锁的情况下解决线程安全问题？
>
> 这其实是一个伪命题，但是考察的知识点要清楚：
>
> 1. 什么是线程安全问题？本质上是指多个线程对于某个共享资源的访问导致的变量的原子性、可见性、有序性的问题；这些问题就会导致共享数据存在一个不可预测性。
> 2. 一般情况下，解决线程安全的问题是增加同步锁，常见的如volatile，Synchronized，Lock这种；本质上就是同一时刻只允许一个线程进行数据操作，这样虽然安全了，但是会有一个性能的问题，因为加锁/释放锁都是需要时间的，会涉及到用户空间到内核空间的一个转换，以及上下文切换
> 3. 在性能和安全性上去做平衡就是“无锁并发”的概念。
>    1. 自旋锁：即线程在没有抢占到资源的情况下，先自旋指定的次数；
>    2. 加Seq：给每个数据增加版本号（其实也就是版本控制的思想），进一步地可以引出CAS操作
> 4. 最后，其实最好减少对共享数据的操作，从业务上去实现隔离，避免并发。

## 1、volatile（悲观锁）

1. 它所修饰的变量不保留拷贝，直接访问主内存中的。
2. 在Java内存模型中，有main memory，每个线程也有自己的memory (例如寄存器)。为了性能，一个线程会在自己的memory中保持要访问的变量的副本。这样就会出现同一个变量在某个瞬间，在一个线程的memory中的值可能与另外一个线程memory中的值，或者main memory中的值不一致的情况。
3. 一个变量声明为volatile，就意味着这个变量是随时会被其他线程修改的，因此不能将它cache在线程memory中。

### 1.1 既然讲到了volatile，就要提一下**CAS操作**（乐观锁）：

- CAS是一种乐观锁。(CAS的语义是“我认为V的值应该为A，如果是，那么将V的值更新为B，否则不修改并告诉V的值实际为多少”)
- 多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。**自旋锁**就是循环执行失败的线程，直到成功为止的过程（这个过程就叫自旋），就叫自旋锁。
- 比较传入的旧值是否与存放地址上的值相同，如果相同，则将新的值替换存放地址上的值；如果传入的旧值与存放地址上的值不相同，那么继续循环这个比较并替换操作，直到成功！（**也就是说他要保证一个位置上的数据新值和预期原值是一致的**）

举个例子：

1. 在内存地址V当中，存储着值为10的变量。
2.  此时线程1想要把变量的值增加1。对线程1来说，旧的预期值A=10，要修改的新值B=11。
3. 在线程1要提交更新之前，另一个线程2抢先一步，把内存地址V中的变量值率先更新成了11。
4. 线程1开始提交更新，首先进行A和地址V的实际值比较（Compare），发现A不等于V的实际值，提交失败。
5. 线程1重新获取内存地址V的当前值，并重新计算想要修改的新值。此时对线程1来说，A=11，B=12。这个重新尝试的过程被称为自旋。
6. 这一次比较幸运，没有其他线程改变地址V的值。线程1进行Compare，发现A和地址V的实际值是相等的。
7. 线程1进行SWAP，把地址V的值替换为B，也就是12。

**缺点：**

1. CPU可能开销较大：在并发量比较高的情况下，如果许多线程反复尝试更新某一个变量，却又一直更新不成功，循环往复，会给CPU带来很大的压力。
2. 不能保证代码块的原子性：**CAS机制所保证的只是一个变量的原子性操作**（只能保证一个共享变量，多个也不行），而不能保证整个代码块的原子性。比如需要保证3个变量共同进行原子性的更新，就不得不使用悲观锁了。
3. ABA问题：CAS的核心思想是通过比对内存值与预期值是否一样而判断内存值是否被改过，但这个判断逻辑不严谨，假如内存值原来是A，后来被一条线程改为B，最后又被改成了A，则CAS认为此内存值并没有发生改变，但实际上是有被其他线程改过的，这种情况对依赖过程值的情景的运算结果影响很大。解决的思路是引入版本号，每次变量更新都把版本号加一。
4. 其实CAS也算是有锁操作，只不过是由CPU来触发（Unsafe类中的compareAndSwapObject方法，LOCK_IF_MP也有锁指令实现的原子操作），比synchronized性能好的多。

### 1.2 CAS锁的ABA问题及解决

- 在CAS的核心算法中，通过死循环不断获取最新的E。如果在此之间，V被修改了两次，但是最终值还是修改成了旧值V，这个时候，就不好判断这个共享变量是否已经被修改过。

- 为了防止这种不当写入导致的不确定问题，原子操作类提供了一个**带有时间戳（版本号）**的原子操作类。之前用的`AtomicReference.compareAndSet`操作
- AtomicStampedReference的compareAndSet函数（四个参数：`compareAndSet(V expectedReference, V newReference, int expectedStamp, int newStamp)。`），这个类的compareAndSet方法的作用是首先检查当前引用是否等于预期引用，并且检查当前的标志是否等于预期标志，如果全部相等，则以原子方式将该应用和该标志的值设置为给定的更新值。
- ~~AtomicMarkableReference，它不是维护一个版本号，而是维护一个boolean类型的标记，用法没有AtomicStampedReference灵活。~~

### 1.3 Volatile结合CAS实现原子性：

- CAS（CompareAndSwap）比较交换原则结合volatile，就能够实现基本的线程安全，典型的应用就是concurrent包下的Atomic类，上述例子如果用AtomicInteger代替int，就能够实现自增情况下的线程安全。

### 1.4 compareAndSet, compareAndSwap 区别

- compareAndSet是API
- compareAndSwap是底层（sun包或者native c++）实现

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220408153523.png)





## 2、synchronized（悲观锁）

1. 当它用来修饰一个方法或者一个代码块的时候，能够保证在同一时刻最多只有一个线程执行该段代码。

**区别：**

1. volatile是变量修饰符，而synchronized则作用于一段代码或方法。

2. volatile只是在线程内存和“主”内存间同步某个变量的值；而synchronized通过锁定和解锁某个监视器同步所有变量的值。显然synchronized要比volatile消耗更多资源。

### 2.1. synchronized的底层

**每个对象有一个监视器锁（monitor）**的标志位（还有一个偏向锁标志位）。当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权，过程如下：

1、如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者。

2、如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1.

3.如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权。

**偏向锁**偏向于第一个获得它的线程，如果在接下来的执行过程中，该锁没有被其他的线程获取，那么持有偏向锁的线程无需再进行同步。

### 2.2.synchronized和volatile比较

- synchronized既能保证共享变量可见性，也可以保证锁内操作的原子性；

- volatile只能保证可见性；volatile不需要同步操作，所以效率更高，不会阻塞线程，但是适用情况比较窄

### 2.3. wait()、notify()、notifyAll()的区别

- 如果对象调用了**wait方法**就会使持有该对象的线程把该对象的控制权交出去，然后处于等待状态。
- 如果对象调用了**notify方法**就会通知某个正在等待这个对象的控制权的线程可以继续运行。
- 如果对象调用了**notifyAll方法**就会通知所有等待这个对象控制权的线程继续运行。

区别：

- 当线程执行wait()时，会把当前的锁释放，然后让出CPU，进入等待状态（等被唤醒后会恢复上下文，继续执行）。（如果不被唤醒，就会一直在内存中等待，导致程序不能结束）

- 当执行notify/notifyAll方法时，会唤醒一个处于等待该 对象锁 的线程，然后继续往下执行，直到执行完退出对象锁锁住的区域（synchronized修饰的代码块）后再释放锁。（当然，再次重入，执行到wait，也可以做到释放锁~）

<font size='5' color='red'>注意：</font>

1. **Java中wait()方法为什么要放在同步块中？**
   - 因为，不放在同步块中，JVM会优化线程执行顺序，当发生了上下文切换的时候，生产者消费者的内部操作可能会不一样。导致另一个线程还没阻塞（wait()），就被唤醒（notify()），这个唤醒就没有效果，导致线程还是阻塞！**lost wake up**问题。
   - 检查机制会报错**IllegalMonitorStateException**。
   - **所以，**Lost Wake-Up这个主要是解释生产者消费者中为什么要把这些操作用锁包起来，因为要保证原子性。为什么wait方法本身需要放在同步块中，synchronized是通过获取monitor对象来实现的，这个对象里面有owner，entryList，waitSet，只有拿到对应的monitor对象才能释放他，添加到waitSet里面。



## 3、ThreadLocal的原理？[场景？](https://cloud.tencent.com/developer/article/1442006)

> 首先，他放在这一节主要是因为他可以解决并发编程的问题，**但是它本身不是锁。**
>
> 他其实是一个线程隔离机制，也是为了解决多线程访问某个共享变量的安全性问题；（比如，有一个实现类是为了把一个数加10，然后并发，这个时候输出可能是：
>
> ```
> 0 28
> 3 28
> 7 28
> 11 28
> 15 28
> ```
>
> 这就是并发导致的。因为线程切换的原因，线程陆续将addNum值设置为0 ，3，7但是都没有执行完（没有执行到return addNum+10这一步）就被切换了，当其中一个线程将addNum值设置为18时，线程陆续开始执行addNum+10这一步，结果都输出了28。）
>
> 场景：
>
> 1. 对数据库链接的隔离
> 2. 对客户端请求会话的一些隔离

### 3.1 理解

ThreadLocal**不是为了解决多线程访问共享变量，**而是为每个线程创建一个单独的变量副本，提供了保持对象的方法和避免参数传递的复杂性。顾名思义它是local variable（线程局部变量）。它的功用非常简单，就是为每一个使用该变量的线程都提供一个变量值的副本，是每一个线程都可以独立地改变自己的副本，而不会和其它线程的副本冲突。



这里有几点需要注意：

- 因为每个 Thread 内有自己的实例副本，且该副本只能由当前 Thread 使用。这是也是 ThreadLocal 命名的由来。
- 既然每个 Thread 有自己的实例副本，且其它 Thread 不可访问，那就不存在多线程间共享的问题。

**原理：**

- 一个线程内可以存在多个 ThreadLocal 对象，所以其实是 ThreadLocal 内部维护了一个 **Map** ，这个 Map 不是直接使用的 HashMap ，而是 ThreadLocal 实现的一个叫做 ThreadLocalMap 的静态内部类。
- 只有当线程第一次调用ThreadLocal的set或者get方法的时候才会创建他们。

**ThreadLocal内存泄漏问题**：

- ThreadLocal 没有被外部强引用的情况下，在**垃圾回收的时候**会被清理掉的，这样一来 ThreadLocalMap中使用这个 ThreadLocal 的 key 也会被清理掉。但是，value 是强引用，不会被清理，这样一来就会出现 key 为 null 的 value。
- ThreadLocalMap实现中已经考虑了这种情况，在调用 set()、get()、remove() 方法的时候，会清理掉 key 为 null 的记录。如果说会出现内存泄漏，那**只有在出现了 key 为 null 的记录后，没有手动调用 remove() 方法，并且之后也不再调用 get()、set()、remove() 方法的情况下。**
- 建议回收自定义的ThreadLocal变量，尤其在线程池场景下，线程经常会被复用，如果不清理自定义的 ThreadLocal变量，可能会影响后续业务逻辑和造成内存泄露等问题。尽量在代理中使用try-finally块进行回收。

如上文所述，ThreadLocal 适用于如下两种**场景**：

- 每个线程需要有自己单独的实例
- 实例需要在多个方法中共享，但不希望被多线程共享

### 3.1 ThreadLocal(线程内安全）与synchronized（线程间）的区别:

- ThreadLocal 是通过让每个线程独享自己的副本，避免了资源的竞争。
- synchronized 主要用于临界资源的分配，在同一时刻限制最多只有一个线程能访问该资源。
- ThreadLocal 并不是用来解决共享资源的多线程访问的问题，因为每个线程中的资源只是副本，并不共享。因此ThreadLocal适合作为线程上下文变量，简化**线程内传参**。

```
将数字加10的工具类：
public class NumUtil {

    public static int addNum = 0;

    public static int add10(int num) {
        addNum = num;
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return addNum + 10;
    }
}

以打印为例：
public static void main(String[] args) {

    ExecutorService service = Executors.newFixedThreadPool(20);

    for (int i = 0; i < 20; i++) {
        int num = i;
        service.execute(()->{
            System.out.println(num + " " +  NumUtil.add10(num));
        });
    }
    service.shutdown();
}
变量
代码输出：
0 28
3 28
7 28
11 28
15 28
如果没有ThreadLocal，多个线程会共享到i，导致出现重复打印；（主要是因为线程切换的原因，线程陆续将addNum值设置为0 ，3，7但是都没有执行完（没有执行到return addNum+10这一步）就被切换了，当其中一个线程将addNum值设置为18时，线程陆续开始执行addNum+10这一步，结果都输出了28。）

解决方案1：每次来都new新的，空间浪费比较大；

解决方案2：这里也可以用synchronized(i)来做，把i作为临界资源，但是并发上不去。
解决方案3：用ThreadLocal，一个线程一个SimpleDateFormat对象
private static ThreadLocal<DateFormat> threadLocal = ThreadLocal.withInitial( ()-> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));		//创建
这里就是private static ThreadLocal<Integer> addNumThreadLocal = new ThreadLocal<>(()->{
            System.out.println(num + " " +  NumUtil.add10(num));
        });		//记得传入泛型

当多个线程同时读写同一共享变量时存在并发问题，如果不共享不就没有并发问题了，一个线程存一个自己的变量，类比原来好几个人玩同一个球，现在一个人一个球，就没有问题了，如何把变量存在线程上呢？其实Thread类内部已经有一个Map容器用来存变量了。
```



## 4、互斥锁和读写锁的区别

相交进程之间的关系主要有两种，同步与互斥。所谓互斥，是指散布在不同进程之间的若干程序片断，当某个进程运行其中一个程序片段时，其它进程就不能运行它们之中的任一程序片段，只能等到该进程运行完这个程序片段后才可以运行。所谓同步，是指散布在不同进程之间的若干程序片断，它们的运行必须严格按照规定的某种先后次序来运行，这种先后次序依赖于要完成的特定的任务。

**读写锁特点：** （pthread_rwlock_init()）(ReadWriteLock；写锁可重入读锁，读锁不能重入写锁；state（0/1/2/...）表示已读状态。)

- 多个读者可以同时进行读
- 写者必须互斥（只允许一个写者写，也不能读者写者同时进行） 
- 写者优先于读者（一旦有写者，则后续读者必须等待，唤醒时优先考虑写者）

**互斥锁特点：**  （pthread_mutex_init()）

- 一次只能一个线程拥有互斥锁，其他线程只有等待

**还有一个条件锁：**（pthread_cond_init）（条件满足才执行）

- 某一个线程因为某个条件为满足时可以使用条件变量使改程序处于阻塞状态。条件锁强调的是条件等待而不是互斥，**条件锁会阻塞当前线程，直到某个条件成立才会继续向下执行**。
- **互斥量用于上锁，条件变量则用于等待，**并且条件变量总是需要与互斥量一起使用，运行线程以无竞争的方式等待特定的条件发生。

### 4.1 自旋锁：

- 读写锁实际是一种特殊的自旋锁。
- 从实现原理上来讲，**Mutex（互斥锁）属于sleep-waiting类型的锁**。例如在一个双核的机器上有两个线程（线程A和线程B）,它们分别运行在Core0和Core1上。假设线程A想要通过pthread_mutex_lock操作去得到一个临界区的锁，而此时这个锁正被线程B所持有，那么线程A就会被阻塞，Core0会在此时进行**上下文切换(Context Switch)**将线程A置于等待队列中，此时Core0就可以运行其它的任务而不必进行忙等待。
- 而**Spin lock（自旋锁）**则不然，它属于**busy-waiting类型的锁**，如果线程A是使用pthread_spin_lock操作去请求锁，那么线程A就会一直在Core0上进行忙等待并不停的进行锁请求，直到得到这个锁为止。因为自旋锁不会引起调用者睡眠，所以自旋锁的效率远高于互斥锁。（一直请求，没有上下文切换；但是他会一直占有CPU，如果不能在短时间内获取到，CPU效率会降低）
- 场景：内核可抢占，或者多处理器；在单CPU且不可抢占（单处理器来说，非抢占的话，自旋锁退化为 关开中断），自旋锁的所有操作都是空操作。抢占来说，自旋锁变成 禁止/打开抢占+关开中断。
- **死锁场景：**（也会出现死锁）递归程序、或者带有睡眠的程序。自旋锁内调用kmalloc或者copy_to_user之类的接口。这类函数的实现内有睡眠操作，睡眠时产生了进程调度,新的进程内如果也使用了该自旋锁，就会导致死锁。（比如，t1锁住o1，开始休眠3s；t2锁住o2，开始休眠3s；t1先醒来，准备拿o2，发现o2加锁，开始自旋等待；t2醒来，拿o1，发现o1加锁，也开始自旋等待，造成死锁）（这个时候，考虑使用互斥锁或者资源计数器）



## 5、ReentrantLock和Synchronized的区别和原理

**相似点：**

- 两个都是可重入锁
- 它们都是加锁方式同步
- 而且都是**阻塞式的同步**，也就是说当如果一个线程获得了对象锁，进入了同步块，其他访问该同步块的线程都必须阻塞在同步块外面等待，而进行线程阻塞和唤醒的代价是比较高的（操作系统需要在用户态与内核态之间来回切换，代价很高，不过可以通过对锁优化进行改善）。

**不同点：**

- 构成：Synchronized锁是java关键字，需要jvm实现（Synchronized的使用比较方便简洁，由编译器去保证锁的加锁和释放，很方便）；ReentrantLock是API层面（实现了 Lock接口）的互斥锁（需要lock()和unlock()方法配合try/finally语句块来完成**手动释放**）；
- 范围：Synchronized锁的是整个方法或者synchronized块部分；而ReentrantLock可以**跨方法调用。**
- 等待与中断问题：Synchronized锁一旦进入就不能被中断（要么执行完，正常释放；要么抛出异常）；而ReentrantLock锁可以**设置超时方法 tryLock(long timeout, TimeUnit unit)**，时间过了就放弃等待；或者**调用interrupt()方法去主动中断**
- 公平性：SynChronized锁是非公平的，释放完后，锁可以抢占；而ReentrantLock锁两者都可以，默认公平锁，构造器可以传入boolean值，true为公平锁，false为非公平锁。
- 适用情况：资源竞争不是很激烈的情况下，偶尔会有同步的情形下，synchronized是很合适的。原因在于，编译程序通常会尽可能的进行优化synchronize，另外可读性非常好；但是当同步非常激烈的时候，synchronized的性能一下子能下降好几十倍。而ReentrantLock确还能维持常态。

### 5.1 ReentrantLock加锁释放锁具体操作：

> AQS框架下的锁则是先尝试CAS乐观锁去获取锁，获取不到才会转换为悲观锁，如：ReentrantLock。

1. A线程准备进去获取锁，首先判断了一下state状态，发现是0，所以可以CAS成功，并且修改了当前持有锁的线程为自己。
2. 这个时候B线程也过来了，也是一上来先去判断了一下state状态，发现是1，那就CAS失败了，真晦气，只能乖乖去等待队列，等着唤醒
3. A准备释放掉锁，所以改了state状态，抹掉了持有锁线程的痕迹，准备去叫醒B。
4. 这时候有新的线程C近来，发现state是0，直接CAS操作修改为1，还修改了当前持有锁的线程为自己。
5. B线程被A叫醒准备去获取锁，发现state居然是1，CAS就失败，继续等待。





## 6、公平锁和非公平锁的区别？

公平锁：多个线程按照申请锁的顺序去获得锁，线程会直接进入队列去排队，永远都是队列的第一位才能得到锁。

- 优点：所有的线程都能得到资源，不会饿死在队列中。
- 缺点：吞吐量会下降很多，队列里面除了第一个线程，其他的线程都会阻塞，cpu唤醒阻塞线程的开销会很大。

非公平锁：多个线程去获取锁的时候，会直接去尝试获取，获取不到，再去进入等待队列，如果能获取到，就直接获取到锁。

- 优点：可以减少CPU唤醒线程的开销，整体的吞吐效率会高点，CPU也不必取唤醒所有线程，会减少唤起线程的数量。
- 缺点：你们可能也发现了，这样可能导致队列中间的线程一直获取不到锁或者长时间获取不到锁，导致饿死。

## 7、锁升级（膨胀）过程（偏向锁/轻量级锁/重量级锁）

> 回答这个问题的时候，[首先要介绍引入一下锁的类型（乐观/悲观），然后是线程阻塞的代价，关键点的引入（markword），改进过后的操作模式（锁获取的流程）。](https://blog.csdn.net/lp284558195/article/details/115547269)
>
> **这个问题的场景引入：**ConcurrentHashMap的锁只会锁住目前获取到的那个Entry所在的那个节点，上锁的时候用的是CAS操作+Synchronized锁，在加上JDK1.6之后引入的锁升级机制，对Synchronized锁进行了一个优化，效率更高，并发度更高。

乐观锁（CAS操作）和悲观锁（volatile、Synchronized、ReentrantLock）上面章节已经做了细致介绍；

- synchronized会导致争用不到锁的线程进入阻塞状态，所以说它是java语言中一个重量级的同步操纵，被称为重量级锁。**为了缓解上述【synchronized】的性能问题，JVM从1.5开始，引入了轻量锁与偏向锁，默认启用了自旋锁，他们都属于乐观锁。**（注意哦，这里是JVM的Synchronized锁，不是AQS锁；在下一节梳理它们差别）

- 如果要阻塞或唤醒一个线程就需要操作系统介入，需要在用户态与核心态之间切换，这种切换会消耗大量的系统资源。因为用户态与内核态都有各自专用的内存空间，专用的寄存器等，用户态切换至内核态需要传递给许多变量、参数给内核，内核也需要保护好用户态在切换时的一些寄存器值、变量等，以便内核态调用结束后切换回用户态继续工作。

要理解锁升级机制，我们要搞清楚它是怎么判断，这就是利用了[Java对象头](https://blog.csdn.net/zqz_zqz/article/details/70246212)（Java对象结构：对象头 + 数组实际数据 + 对齐填充组成）的一个关键字段——**Mark Word**：（markword数据的长度在32位和64位的虚拟机（未开启压缩指针）中分别为32bit和64bit，**它的最后2bit是锁状态标志位，无锁和偏向锁是需要看倒数第三位的状态**，所以我们这里也只看这三位）：

| 状态             | 标志位 | 存储内容                             |
| ---------------- | ------ | ------------------------------------ |
| 未锁定           | 001    | 对象哈希码、对象分代年龄             |
| 可偏向           | 101    | 偏向线程ID、偏向时间戳、对象分代年龄 |
| 轻量级锁定       | 00     | 指向锁记录的指针                     |
| 膨胀(重量级锁定) | 10     | 执行重量级锁定的指针                 |
| GC标记           | 11     | 空(不需要记录信息)                   |

![image-20220509170604760](https://img-blog.csdn.net/20170419215511634?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvenF6X3pxeg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

跟着讲`7.2 锁升级的过程`也行，但是这里可以补充一点小知识，就是这个标志位在修改时要注意的一些细节，比如stop-the-world情况。

**以偏向锁举例：**

- 如果在运行过程中，同步锁只有一个线程访问，不存在多线程争用的情况，则线程是不需要触发同步的，这种情况下，就会给线程加一个偏向锁。（不去升级）
- 如果在运行过程中，遇到了其他线程抢占锁，**则持有偏向锁的线程会被挂起，JVM会消除它身上的偏向锁，**将锁恢复到标准的轻量级锁。（所以如果有人竞争偏向锁，那这个持有偏向锁的线程就会释放锁）

**偏向锁获取过程：**

1. 访问Mark Word中偏向锁的标识是否设置成1，锁标志位是否为01，确认为可偏向状态。
2. 如果为可偏向状态，则测试线程ID是否指向当前线程，如果是，进入步骤5，否则进入步骤3。
3. 如果线程ID并未指向当前线程，则通过CAS操作竞争锁。如果竞争成功，则将Mark Word中线程ID设置为当前线程ID，然后执行5；如果竞争失败，执行4。
4. 如果CAS获取偏向锁失败，则表示有竞争。**当到达全局安全点（safepoint，会导致stop the world，时间很短）时获得偏向锁的线程被挂起**，偏向锁升级为轻量级锁，然后被阻塞在安全点的线程继续往下执行同步代码。**（撤销偏向锁的时候会导致stop the world）**
5. 执行同步代码。

**高并发的应用会禁用掉偏向锁。**Jvm开启/关闭偏向锁

- 开启偏向锁：`-XX:+UseBiasedLocking` `-XX:BiasedLockingStartupDelay=0`
- 关闭偏向锁：`-XX:-UseBiasedLocking`

**偏向锁释放过程：**

1. 之前在获取锁的时候它拷贝了锁对象头的markword，在释放锁的时候如果它发现在它持有锁的期间有其他线程来尝试获取锁了，并且该线程对markword做了修改，两者比对发现不一致，则切换到轻量锁。
2. 而这个对比切换是需要check的，所以要stop-the-world



### 7.2 [锁升级的过程](https://blog.csdn.net/weixin_52593321/article/details/119918202)

> 无锁 ——> 偏向锁 ——> CAS操作 ——> 轻量级锁 ——> 竞争失败 ——> 自旋 ——> 竞争失败 ——> 重量级锁

1. 当没有被当做锁的时候，这就是个普通对象，锁标志位为01，是否偏向锁为0

2. 当对象被当做同步锁时，一个线程A抢到锁时，锁标志位依然是01，是否偏向锁为1，前23位记录A线程的线程ID，此时锁升级为偏向锁

3. 当线程A再次试图来获得锁时，JVM发现同步锁对象的标志位是01，是否偏向锁是1，也就是偏向状态，Mark Word中记录的线程id就是线程A自己的id，表示线程A已经获得了这个偏向锁，可以执行同步锁的代码，这也是偏向锁的意义

4. 当一个线程B尝试获取锁，JVM发现当前的锁处于偏向状态，并且现场ID不是B线程的ID，那么线程B会先用CAS将线程id改为自己的，这里是有可能成功的，因为A线程一般不会释放偏向锁。如果失败，则执行5

5. 偏向锁抢锁失败，则说明当前锁存在一定的竞争，偏向锁就升级为轻量级锁。JVM会在当前线程的现场栈中开辟一块单独的空间，里面保存指向对象锁Mark Word的指针，同时在对象锁MarkWord中保存指向这片空间的指针。上面的保存都是CAS操作，如果竞争成功，代表线程B抢到了锁，可以执行同步代码。如果抢锁失败，则继续执行6

6. 轻量级锁抢锁失败，则JVM会使用自旋锁，自旋锁并非是一个锁，则是一个循环操作，不断的尝试获取锁。从JDK1.7开始，自旋锁默认开启，自旋次数由JVM决定。如果抢锁成功，则执行同步代码；如果抢锁失败，则执行7
   - 但是线程自旋是需要消耗CPU的，说白了就是让CPU在做无用功，如果一直获取不到锁，那线程也不能一直占用CPU自旋做无用功，所以需要**设定一个自旋等待的最大时间**。如果持有锁的线程执行的时间超过自旋等待的最大时间扔没有释放锁，就会导致其它争用锁的线程在最大等待时间内还是获取不到锁，这时争用线程会停止自旋进入阻塞状态。（关于自旋锁，可以参考本节`4.1 自旋锁`）
   - JVM还针对当前CPU的负荷情况做了较多的优化： 
     - 如果平均负载小于CPUs则一直自旋；
     - 如果有超过(CPUs/2)个线程正在自旋，则后来线程直接阻塞；
     - 如果正在自旋的线程发现Owner发生了变化则延迟自旋时间（自旋计数）或进入阻塞；
     - 如果CPU处于节电模式则停止自旋；
     - 自旋时间的最坏情况是CPU的存储延迟（CPU A存储了一个数据，到CPU B得知这个数据直接的时间差）；
     - 自旋时会适当放弃线程优先级之间的差异；

7. 自旋锁**重试一定次数之后**（10次以上，或者CPU核数的一半；就是上面说的优化）仍然未抢到锁，同步锁会升级至重量级锁，锁标志位改为10，在这个状态下，未抢到锁的线程都会被阻塞，由Monitor来管理，并会有线程的park与unpark，因为这个存在用户态和内核态的转换，比较消耗资源，故名重量级锁
   ————————————————

## 8、AQS锁框架AbstractQueuedSynchronizer——他和Synchronized的关系

> 上面第7节，介绍了锁升级的原理，其实java中的悲观锁有两种：
>
> 1. Synchronized锁
> 2. AQS框架下的锁，则是先尝试CAS乐观锁去获取锁，获取不到才会转换为悲观锁，如：ReentrantLock。
>
> 下面我们看一下他们之间的差别。

- 隐式锁：类似于Synchronized加锁机制，JVM的内置锁，不需要手动加锁和解锁

- 显示锁：ReentrantLock，实现JUC 里面Lock，实现是基于AQS实现，需要手动加锁和解锁。ReentrantLock的lock()和unlock()

- JVM内置锁的灵活度要低于AQS锁。JVM内置锁几乎不可能跨方法加锁，Synchronized加锁的是对象，但是AQS可以
- Synchronized可以理解为操作系统实现的锁，AQS是利用JAVA语言自己实现的一个锁机制

————————————————

### 8.1 AQS 原理

> 并发编程时候，被问到“请说一下你对AQS原理的理解”。
>
> 使用AQS的方式通常不是让锁或同步器直接继承AQS类，而是将AQS的子类作为锁或同步器类的一个辅助内部类，锁或同步器的方法调用AQS子类对象的方法完成同步操作。

- AQS 核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒 时锁分配的机制，这个机制 AQS 是用 CLH 队列锁实现（FIFO双向队列）的，即将暂时获取不到锁的线程加入到队列中。
- **底层：它底层用了CAS技术来保证操作的原子性，同时利用FIFO队列实现线程间的锁竞争**
- CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列(**虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系**）。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点（Node）来实现锁的分配。支持各种各样的锁的实现：
  - 阻塞等待队列
  - 共享、独占
  - 公平、非公平
  - **可重入**（重要特性）——比如：ReentrantLock锁
  - **允许中断**（重要特性）
- AQS框的组成，分为两个组件。**第一同步器，第二队列。**

#### 8.1.1 AQS 定义两种资源共享方式

1） **Exclusive（独占）**

- 只有一个线程能执行，如 ReentrantLock。又可分为公平锁和非公平锁,ReentrantLock 同时支持两种锁

2）**Share（共享）**

- 多个线程可同时执行，如 Semaphore/CountDownLatch。Semaphore、CountDownLatCh、CyclicBarrier、ReadWriteLock我们都会在后面讲到。ReentrantReadWriteLock 可以看成是组合式，因为ReentrantReadWriteLock 也就是读写锁允许多个
  线程同时对某一资源进行读。不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源state的
  获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS在上层已经帮我们实现好了。





### 8.2 AQS的同步器

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220509201346.png)

- 同步器的顶级类是JUC.locks（Author:Doug Lea）的AbstractOwnableSynchronizer，进入里面我们可以看到head，tail，state,一堆offset和unsafe，所以，可以知道AQS的同步器基本包含了用于获取锁的线程对象Thread（exclusiveOwnerThread）、判断是否有线程获取锁的字段status、队列的头部节点head、队列的尾部节点tail。
- 而AbstractOwnableSynchronizer中的每一个节点的数据结构，对应了一个内部类——Node类。他有很多字段，但是基本上还是维护了一个队列，帮助我们去定位到相关联的锁对象。比如几个重要的字段：
  - nextWaiter，下一个节点的状态：共享和独占状态，SHARED（共享），EXCLUSIVE（独占）。
  - waitStatus等待状态，有五个值。CANCELLED，SIGNAL，CONDITION，PROPAGATE，0。
  - 前置节点，后置节点：prev，next。
  - 线程对象的引用，thread。

![AbstractOwnableSynchronizer的内部](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220509201852.png)

> 上面都是框架类的不用在意，如果回答问题，就是下面这个

#### 8.2.1 State字段 - CAS操作保证变量更新的原子性

- AQS中最重要的一个字段就是同步状态字段state，锁和同步器的实现都是围绕着这个字段的修改展开的；
- AQS中也暴露出了一些方法供我们重写以操作这个字段，例如tryAcquire, tryRelease, tryAcquireShared, tryReleaseShared，这四个方法底层又是调用AQS提供的getState, setState, compareAndSwapState来修改同步状态。
  - 例如，ReentrantLock中，同步状态state代表独占锁的重入次数，获取锁时+1，释放锁时-1。
  - 而在ReentrantReadWriteLock中，同步状态state被划分为高16位和低16位，分别表示读锁计数和写锁计数。



### 8.3 [AQS基于队列的线程管理](https://blog.csdn.net/qq_29328443/article/details/108109773)

> 未获取到锁的线程通过unsafe类中park方法去进行阻塞，把阻塞的线程按照先进先出的原则去加入到一个双向链表的结构中（虚拟的）。当获得锁资源的线程在释放锁之后，会从这个链表的头部去唤醒下一个等待的线程，再去竞争锁。
>
> 这时候又分为公平锁（按阻塞链表依次进行）和非公平锁（不管有没有阻塞链表，该线程都会去尝试持有变量的state字段）。

- 对于修改同步状态失败的线程，以及调用await而阻塞的线程，AQS替我们提供了这些线程的管理机制，包括线程的排队、阻塞和唤醒等等。
- 这些线程的管理机制对于锁和同步器的编写者来说透明的，他们也没有必要修改这些线程管理相关的方法，因此这些方法在AQS类中被声明为final的，这里用到了模板方法设计模式。
- AQS内部有两个队列可以用来管理线程，分别是**同步队列和条件队列。**（其实也就是上面8.2中说的node实现类）
  - 同步队列是基于CLH队列改进得到的，它是一个基于双链表的队列，获取同步状态失败而需要阻塞的线程会被放入同步队列，并在合适的时机被唤醒。同步队列中的节点有独占模式和共享模式，对于独占模式，同一时刻只能有一个线程持有锁；对于共享模式，同一时刻可以有多个线程持有锁
  - 条件队列是一个基于单链表的队列，它主要是为了实现await和signal方法，这两个方法类似Object类中的wait和signal。当线程调用await方法时，它会被封装为Node节点放入条件队列末尾，释放同步状态然后阻塞；当await阻塞的线程被唤醒或中断时，会导致转移到同步队列，然后恢复同步状态。
    ————————————————
    版权声明：本文为CSDN博主「ponnylv」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
    原文链接：https://blog.csdn.net/qq_29328443/article/details/108109773







# 三、Java多线程——池化思想

> 池化思想其实很多地方都有用到，这里是java中的应用；在计算机网络中，TCP的拥塞窗口其实也是一个连接池的思路。因为连接的建立和断开都很耗时。一开始的拥塞窗口比较小，然后随着数据不断传输，这个窗口才会不断变大，虽然有些参数可以用来优化这个窗口，但是**他始终不是一个复用的思想**。
>
> 讲完这两个之间的差别之后，我们再去分析线程池的内容，就比较好了。

## 1、Java线程池

池化思想：线程池、字符串常量池、数据库连接池。**——>**提高利用率。

**无线程池的步骤：**

1. 手动创建线程对象
2. 执行任务
3. 执行完毕，释放线程对象

**有线程池的步骤：**

1. 在创建了线程池后，等待提交过来的任务请求
2. 当调用execute()方法添加一个请求任务时，线程池会做如下判断
   1. 如果正在运行的线程数量小于corePoolSize，那么马上创建线程运行这个任务
   2. 如果正在运行的线程数量大于或等于corePoolSize，那么将这个任务放入队列
   3. 如果这时候队列满了且正在运行的线程数量还小于maximumPoolSize，那么还是要创建非核心线程立刻运行这个任务
   4. 如果队列满了且正在运行的线程数量大于或等于maximumPoolSize，那么线程池会启动饱和拒绝策略来执行
3. 当一个线程完成任务时，它会从队列中取下一个任务来执行
4. 当一个线程无事可做超过一定的时间（keepAliveTime）时，线程池会判断
   1. 如果当前运行的线程数大于corePoolSize，那么这个线程就被停掉
   2. 所以线程池的所有任务完成后它最终会收缩到corePoolSize的大小

### 1.1 使用线程池的一些注意点？

- 在连接池里面，[**一般单个的connection不是线程安全的**。](https://www.jianshu.com/p/9a2e24f27ed5)但是整个connection pool是线程安全的。它在多线程环境中使用时,会导致数据操作的错乱,特别是有事务的情况.connection.commit()方法就是提交事务,你可以想象,在多线程环境中,线程A开启了事务,然后线程B却意外的commit,这该是个多么纠结的情况.（对于单独查询的情况，似乎不会出现数据错乱的情况。是因为在JDBC中，使用了锁进行同步。）——**所以，在使用事务的场景中，无论操作多少次DB，事实上都是操作的同一个Connection对象。**
- connection对象的底层是一个socket连接，以及相关联的输入输出流；
- 在整个使用过程中，connection对象会被业务线程独占，用完之后，再调用close()方法
- connection会实现closable接口，在close()方法中，不是关闭的socket的链接，connection对象放回到连接池里面，然后其它的线程再去从线程池中拿到这个connection来使用**（复用）**
- 一般情况下，连接池是单例的。比如链接redis集群， 一个服务只需要创建一个线程池就可以，然后复用其中的连接。在创建连接池的时候，要指定它的最大连接数（maximumPoolSize）和最小连接数（corePoolSize）、过期时间等。（具体的看下面的说明）
- 连接池使用心跳来确定一个connection对象是否仍然在使用。





## 2、Java线程池的七大参数

java中创建线程池底层是通过java.util.concurrent.ThreadPoolExecutor这个类

有7大参数：

- **corePoolSize：**线程池中的常驻核心线程数，在创建了线程池后，当有请求任务来之后，就会安排池中的线程去执行请求任务。当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中。
- **maximumPoolSize：**线程池能够容纳同时执行的最大线程数，此值必须大于等于1。
- **keepAliveTime：**多余的空闲线程的存活时间，当前线程池数量超过corePoolSize时，当空闲时间达到keepAliveTime值时，多余空闲线程会被销毁直到只剩下corePoolSize个线程为止。
- **unit：**keepAliveTime的单位。
- **workQueue：**任务队列，被提交但尚未被执行的任务。
- **threadFactory：**表示生成线程池中工作线程的线程工厂，用于创建线程一般用默认的即可。
- **handler：**拒绝策略，表示当队列满了，再也塞不下新任务了，同时，工作线程大于等于线程池的最大线程数，无法继续为新任务服务，这时候我们就需要拒绝策略机制合理的处理这个问题，默认会抛异常
  - AbortPolicy（默认）：直接抛出java.util.concurrent.RejectedExecutionException异常阻止系统正常运行
  - CallerRunsPolicy："调用者运行"一种调节机制，该策略既不会抛弃任务，也不会抛出异常，而是将某些任务回退到调用者，从而降低新任务的流量。
  - DiscardOldestPolicy：抛弃队列中等待最久的任务，然后把当前任务加入队列中尝试再次提交当前任务。
  - DiscardPolicy：直接丢弃任务，不予任何处理也不抛出异常。

## 3、Java是怎么创建多线程的？

java中创建多线程的四种方式，分别是继承Thread类，实现Runnable接口，jdk5.0以后又增加了两种方式：实现Callable接口和使用线程池。

### 3.1 继承Thread类

1. 定义一个类继承Tread类`class Demo extends Thread`
2. 重写run方法：里面写线程内要运行的任务代码
3. 主函数中新建（实例化）一个Thread子类（Demo类）的对象`Demo d1 = new Demo("旺财");`
4. 调用start方法（开启线程，并调用run方法）`d1.start();`

### 3.2 实现Runnable接口

1. 定义子类实现Runnable接口`class Demo2 implements Runnable`
2. 子类中重写run方法：将线程的任务代码封装到run方法中；
3. 创建实现子类的对象`Demo2 d = new Demo2();`
4. 通过Thread类创建线程对象，并将该子类对象作为构造器的参数进行传递`Thread t1 = new Thread(d);`
5. 调用Thread类的start方法

### 3.3 实现Callable接口

1. 创建Callable的实现类`class NumThread implements Callable`
2. 重写call方法，将线程的任务代码封装到call方法中
3. 创建Callable接口实现子类的对象；`NumThread numThread = new NumThread();`
4. 创建FutureTask的对象，将此Callable接口实现类的对象作为构造器的参数进行传递： `FutureTask futureTask = new FutureTask(numThread);`
5. 创建Thread对象，并调用start()。将FutureTask的对象作为参数传递到Thread类的构造器中：new Thread(futureTask).start();
6. 调用Callable中 get()方法，获取返回值（多的一步，可以返回消息）

### 3.4 使用线程池

1. 提供指定线程数量的线程池；借助于Executors中的方法；`ExecutorService service = Executors.newFixedThreadPool(10);`
2. 执行指定的线程的操作，需要提供实现Runnable接口或Callable接口实现类的对象
   1. Runnable：service.execute（子类对象）
   2. Callable：service.submit（子类对象）
3. 关闭连接池`service.shutdown();`

#### [3.4.1线程池好处](https://www.cnblogs.com/benjieqiang/p/11376076.html#3833114111)

> 提高响应速度（减少了创建新线程的时间）
> 降低资源消耗（重复利用线程池中的线程，不需要每次都创建）；
> 便于线程管理

线程池示例：

```java
package ExecutorDemo;


import java.util.concurrent.ExecutorService;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

import static java.util.concurrent.Executors.*;


public class ExecutorController {
    public static void main(String[] args) {
        //cachedThreadPool();
        //fixedThreadPool();
        scheduledThreadPool();
        //singleThreadExecutor();
    }


    //    缓存线程池
    public static void cachedThreadPool() {
        ExecutorService cachedThreadPool = newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            final int index = i;
            try {
                // sleep可明显看到使用的是线程池里面以前的线程，没有创建新的线程
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            cachedThreadPool.execute(new Runnable() {
                @Override
                public void run() {
                    // 打印正在执行的缓存线程信息
                    System.out.println(index + "=" + Thread.currentThread().getName() + "正在被执行");
                }
            });
        }
        cachedThreadPool.shutdown();
    }

    //创建一个可重用固定个数的线程池，以共享的无界队列方式来运行这些线程
    public static void fixedThreadPool() {
        ExecutorService fixedThreadPool = newFixedThreadPool(3);
        for (int i = 0; i < 5; i++) {
            fixedThreadPool.submit(() -> System.out.println(Thread.currentThread().getName()
                    + "正在被执行"));
        }
        fixedThreadPool.shutdown();
    }

    //创建一个定长线程池，支持定时及周期性任务执行
    public static void scheduledThreadPool() {
        //创建一个定长线程池，支持定时及周期性任务执行——延迟执行
        ScheduledExecutorService scheduledThreadPool = newScheduledThreadPool(5);
        scheduledThreadPool.scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                System.out.println("延迟1秒后每3秒执行一次");
            }
        }, 1, 3, TimeUnit.SECONDS);

    }

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
}
```

## 4、死锁

> 这一节内容参考了`javaReview01 - 二、操作系统 - 4、死锁`

死锁是指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。

死锁的四个必要条件：

- 互斥条件：一个资源每次只能被一个进程使用；
- 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放；
- 不剥夺条件:进程已获得的资源，在末使用完之前，不能强行剥夺；
- 循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系；

**（预防）解决方法：（其实就是针对上面的四种条件去做）**

- 打破互斥条件。即允许进程同时访问某些资源。（不实际）
- 打破不可抢占条件。即允许进程强行从占有者那里夺取某些资源。（会降低系统性能）
- 打破占有且申请条件。可以实行资源预先分配策略。即进程在运行前一次性地向系统申请它所需要的全部资源。如果某个进程所需的全部资源得不到满足，则不分配任何资源，此进程暂不运行。（资源利用率低）
- 打破循环等待条件，实行资源有序分配策略。采用这种策略，即把资源事先分类编号，按号分配，使进程在申请，占用资源时不会形成环路。

### 4.1 怎么查看？怎么避免？

**死锁的查看：**

**在操作系统中，**

1. 先用`jps`命令获取正在运行的java进程对应的进程号
2. 然后，查看**带锁信息的堆栈：**`jstack -l 8216`

![image-20220504151844722](https://img2018.cnblogs.com/blog/1209816/201908/1209816-20190812233159608-871428362.png)

这里显示：

1）Thread-1和Thread-0相互等待，造成死锁

2）Thread-1在DeadLockDemo的38行等待

3）Thread-0在DeadLockDemo的23行等待

**在Mysql中，**它内部也是一些线程在运行（死锁的具体表现有两种：  Mysql 增改语句无法正常生效 使用Mysql GUI 工具编辑字段的值时，会出现异常。）。有两种方法

1. 直接查询哪些表正在被多人同时使用（这是不被允许的）：
   - 查询是否锁表`show OPEN TABLES where In_use > 0;`
   - 查询进程（如果您有SUPER权限，您可以看到所有线程。否则，您只能看到您自己的线程）`show processlist`
   - 杀死进程id（就是上面命令的id列）
2. 利用锁的事务：
   - 查看当前的事务`SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX;`
   - 杀死进程id（就是上面命令的trx_mysql_thread_id列）—— kill 线程ID
   - 其他有关查看死锁的命令：
     - 查看当前锁定的事务 `SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;`
     - 查看当前等锁的事务 `SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;`

**死锁的避免**：（Mysql的死锁避免，可以参考`javaReview03 - 一、MySQL数据库 - 9.3数据库死锁场景？怎么避免死锁`）

它不限制进程有关申请资源的命令，而是对进程所发出的每一个申请资源命令加以动态地检查，并根据检查结果决定是否进行资源分配。

1. 安全序列
   - 一个进程序列<P1，…，Pn>是安全的，如果对于每一个进程Pi(1≤i≤n），它以后尚需要的资源量不超过系统当前剩余资源量与所有进程Pj (j < i )当前占有资源量之和，系统处于安全状态 (安全状态一定是没有死锁发生的)。
   - 否则，就可能进入死锁状态，因为进程可能在中间过程释放资源，也会使得系统正常运转下去，从而不会进入死锁。
2. 银行家算法（也是要去找安全序列）

**死锁的检测与恢复：**

- 撤消进程，剥夺资源。终止参与死锁的进程，收回它们占有的资源，从而解除死锁。
- 进程回退策略，即让参与死锁的进程回退到没有发生死锁前某一点处，并由此点处继续执行，以求再次执行时不再发生死锁。









# 四、JVM

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxNy85LzQvZGE3N2Q5MDE0Njc4NmMwY2IzZTE3MGI5YzkzNzZhZTQ_aW1hZ2VWaWV3Mi8wL3cvMTI4MC9oLzk2MC9mb3JtYXQvd2VicC9pZ25vcmUtZXJyb3IvMQ)

JVM的内存结构包括五大区域：**程序计数器、虚拟机栈、本地方法栈、堆区、方法区。**

堆：存的全是对象本身，被所有线程共享。

栈：基本数据类型的对象，和对象的**引用**；操作指令；方法的出口

方法区：（又叫静态区），所有的类信息，静态变量

举个例子：

- AppMain.java

```java

public   class  AppMain                //运行时, jvm 把appmain的信息都放入方法区
{

    public   static   void  main(String[] args)  //main 方法本身放入方法区。

    {undefined

    Sample test1 = new  Sample( " 测试1 " );   //test1是引用，所以放到栈区里， Sample是自定义对象应该放到堆里面

    Sample test2 = new  Sample( " 测试2 " );

    test1.printName();

    test2.printName();

    }

}
```

- Sample.class

```
public   class  Sample        //运行时, jvm 把appmain的信息都放入方法区

{

    /** 范例名称 */

    private  name;      //new Sample实例后， name 引用放入栈区里，  name 对象放入堆里

    /** 构造方法 */

    public  Sample(String name)

    {undefined

    this .name = name;

    }

    /** 输出 */

    public   void  printName()   //print方法本身放入 方法区里。

    {undefined

    System.out.println(name);

    }

}
```

这是个简单例子，AppMain.class中调用了Simple.class。我们来看一下，<font color='red' size='4.5'>JVM是怎么操作的？</font>

1. 系统收到了我们发出的指令，启动了一个Java虚拟机进程，这个进程首先从classpath中找到AppMain.class文件，读取这个文件中的二进制数据，然后把Appmain类的类信息存放到运行时数据区的**方法区**中。（这叫类的加载）
2. Java虚拟机定位到方法区中AppMain类的Main()方法的字节码（使用Java本地接口调用 Main方法，也就是找字节码），开始执行它的指令。
3. 发现，main()方法中，第一条就是`Sample test1=new Sample("测试1");`很明显，这就是让JVM实例化一个对象，并使引用变量test1引用这个实例。
   - JVM直接去方法区找，有没有Sample类信息，发现没有，那就把他加载上来，并且把类信息存到方法区中
4. 找到Simple类之后，马上就要在**堆区**中为一个新的Sample实例分配内存，这个Sample实例持有着指向方法区的Sample类的类型信息的引用（还会往回指向，因为方法区里面有数据）。
5. 在JVM的**虚拟机栈**中，维护了一个方法调用栈，用来跟踪线程运行中一系列的方法调用过程，栈中的每一个元素就被称为栈帧，每当线程调用一个方法的时候就会向方法栈压入一个新帧（现在我们还没压入方法）。这里的帧用来存储方法的参数、局部变量和运算过程中的临时数据。
6. 位于“=”前的test1是一个在main()方法中定义的变量，可见，它是一个局部变量（注意，是局部变量，不是常量！！），因此，它被会添加到了执行main()方法的主线程的JAVA**方法调用栈（即虚拟机栈）**中。而“=”将把这个test1变量指向堆区中的Sample实例，也就是说，**它持有指向Sample实例的引用**。（通过它可以找到堆中的Simple实例对象）
7. JAVA虚拟机将继续执行后续指令，在堆区里继续创建另一个Sample实例，在虚拟机栈中存入test2，指向它。
8. 然后依次执行它们的printName()方法。当JAVA虚拟机执行test1.printName()方法时，JAVA虚拟机根据局部变量test1持有的引用，定位到堆区中的Sample实例，再根据Sample实例持有的引用，定位到方法去中Sample类的类型信息，从而获得printName()方法的字节码，接着执行printName()方法包含的指令。（所以，操作顺序是，虚拟机栈——>堆——>方法区）



### PS:为什么要使用Native Method(为什么要有Native本地方法)：

- 本地方法栈和虚拟机栈所发挥的作用是非常相似的，它们之间的区别不过是虚拟机栈是非虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则是为虚拟机执行Native方法服务的。
- 与java环境外交互（比如操作系统）
- 最常见的：wait()、notify()、notifyAll()最终调用的都是jvm级的native方法。
- JVM怎样使Native Method跑起来：当一个带有本地方法的类被加载时，其相关的DLL并未被加载，因此指向方法实现的指针并不会被设置。当本地方法被调用之前，这些DLL（方法描述符：方法代码存于何处，它有哪些参数，方法的描述符（public之类）等）才会被加载，这是通过调用java.system.loadLibrary()实现的。



程序计数器、虚拟机栈、本地方法栈都会随着线程的开始和终止被创建和销毁；

JAVA的**堆区和方法区**，这部分内存的分配和回收是**动态**的，正是垃圾收集器所需关注的部分。

## 1、JVM的4种垃圾回收算法：

1. **标记-清除算法：**标记阶段的任务是标记出所有需要被回收的对象，清除阶段就是回收被标记的对象所占用的空间。标记-清除算法不需要进行对象的移动，只需对不存活的对象进行处理，在存活对象比较多的情况下极为高效，但由于标记-清除算法直接回收不存活的对象，因此会造成内存碎片。
2. **复制算法(Copying)：**它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用的内存空间一次清理掉，这样一来就不容易出现内存碎片的问题。（但是却对内存空间的使用做出了高昂的代价，**因为能够使用的内存缩减到原来的一半**。）
3. **标记-整理算法(Mark-compact)：**该算法标记阶段和Mark-Sweep一样，但是在完成标记之后，它不是直接清理可回收对象，而是将存活对象都向一端移动(美团面试题目，记住是完成标记之后，先不清理，先移动再清理回收对象)，然后清理掉端边界以外的内存(美团问过)。（但是速度就慢，因为多了对象移动的操作）
4. **分代收集算法 Generational Collection：**它的核心思想是根据对象存活的生命周期将内存划分为若干个不同的区域。
   1. 一般情况下将堆区划分为老年代（Tenured Generation）和新生代（Young Generation）。在堆区之外还有一个代就是永久代（Permanet Generation）。老年代的特点是每次垃圾收集时只有少量对象需要被回收，而新生代的特点是每次垃圾回收时都有大量的对象需要被回收，那么就可以根据不同代的特点采取最适合的收集算法。
   2. 新生代都采取Copying算法，因为新生代中每次垃圾回收都要回收大部分对象，也就是说需要复制的操作次数较少，但是实际中并不是按照1：1的比例来划分新生代的空间的，一般来说是将新生代划分为一块较大的Eden空间和两块较小的Survivor空间（一般为8:1:1），每次使用Eden空间和其中的一块Survivor空间，当进行回收时，将Eden和Survivor中还存活的对象复制到另一块Survivor空间中，然后清理掉Eden和刚才使用过的Survivor空间。（注意两个survivor之间是相互传递的，这次使用A，下次就是B）；Survivor之间转移对象的时候，每次都会把对象的年龄加1，如果到了15（默认是15，可以设置），就把它转移到老年代中
   3. 老年代的特点是每次回收都只回收少量对象，一般使用的是Mark-Compact算法(标记整理算法)。
      - 当Eden没有足够空间的时候就会 触发jvm发起一次Minor GC。（新生代发生的GC也叫做Minor GC）
      - 当survivor1区不足以存放 eden和survivor0的存活对象时，就将存活对象直接存放到老年代。若是老年代也满了就会触发一次Full GC(Major GC)，也就是新生代、老年代都进行回收。
5. **持久代（Permanent Generation）(也就是方法区)的回收算法**：用于存放静态文件，如Java类、方法等。持久代对垃圾回收没有显著影响，但是有些应用可能动态生成或者调用一些class，例如Hibernate 等，在这种时候需要设置一个比较大的持久代空间来存放这些运行过程中新增的类。
   1. 方法区主要回收的内容有：废弃常量和无用的类。
      - 对于废弃常量也可通过引用的可达性来判断（强/弱引用）
      - 对于无用的类则需要同时满足：1）该类所有的实例都已经被回收，也就是Java堆中不存在该类的任何实例；2）加载该类的ClassLoader已经被回收；3）该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。



## 2、老年代空间分配担保机制

### 2.1 为什么要进行空间担保？

- 是因为新生代采用**复制收集算法**，假如大量对象在Minor GC后仍然存活（最极端情况为内存回收后新生代中所有对象均存活），而Survivor空间是比较小的，这时就需要老年代进行分配担保，把Survivor无法容纳的对象放到老年代。**老年代要进行空间分配担保，前提是老年代得有足够空间来容纳这些对象**，但一共有多少对象在内存回收后存活下来是不可预知的，**因此只好取之前每次垃圾回收后晋升到老年代的对象大小的平均值作为参考**。使用这个平均值与老年代剩余空间进行比较，来决定是否进行Full GC来让老年代腾出更多空间。

### 2.2 怎么操作分配担保？

- 年轻代每次minor gc之前JVM都会计算下老年代剩余可用空间，如果这个可用空间小于年轻代里现有的所有对象大小之和（包括垃圾对象）就会去看一个”-XX:HandlePromotionFailure”(jdk默认就设置了)的参数是否设置了，如果有这个参数，就会看老年代的可用内存大小，是否大于**之前（用历史数据作参考）**每一次minor gc后进入老年代的对象的平均大小。如果上一步的结果是小于或者之前说的参数没有设置，那么会触发一次Full gc,对老年代和年轻代一起回收一次垃圾，如果回收完还是没有足够空间存放新对象就会发生”OOM”；

- 当然，如果minor gc之后剩余存活的需要挪动到老年代的对象大小还是大于老年代可用空间，那么也会触发full gc,full gc完之后如果还是没有空间放minor gc之后存活的对象，则也会发生”OOM”。

## 3、G1垃圾回收器

### 3.1 回收原理？

G1是一个分代的，增量的，并行与并发的标记-整理垃圾回收器。

1. 初始标记(stop the world事件 CPU停顿只处理垃圾)；

2. 并发标记(与用户线程并发执行)；

3. 最终标记(stop the world事件 ,CPU停顿处理垃圾)；

4. 筛选回收(stop the world事件 根据用户期望的GC停顿时间回收)——**优先选择垃圾较多的区域进行回收**（启发式算法）。（和CMS的区别所在）



### 3.2 **G1用户自己设置停顿时间的参数是什么？**

G1还有一个及其重要的特性：软实时（soft real-time）。所谓的实时垃圾回收，是指在要求的时间内完成垃圾回收。“软实时”则是指，用户可以指定垃圾回收时间的限时，G1会努力在这个时限内完成垃圾回收，但是G1并不担保每次都能在这个时限内完成垃圾回收。通过设定一个合理的目标，可以让达到90%以上的垃圾回收时间都在这个时限内。

### 3.3 **GC Roots有哪些？**

- 虚拟机栈中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中JNI（即一般说的native方法）中引用的对象

#### 3.3.1 [强引用和弱引用之间的差别？其实中间还有个软引用、虚引用](https://blog.csdn.net/CSDN_DK317/article/details/119742574)

> 强引用 > 软引用 > 弱引用 > 虚引用

**强引用和弱引用之间区别：**

1. 弱引用,一般都是带有明显的创建提示
2. 最大的区别,就是垃圾回收器的回收机制不同:强引用,只要有东西指着这个就是一定不会回收;弱引用,如果只有一个弱引用指着这个对象,那么垃圾回收器仍然会回收这个对象.

**例证分析：**

#### 3.3.2 **强引用：**（一般我们都是用这个）

```java
 String str = "abc";
 List<String> list = new Arraylist<String>();
 list.add(str);
Student student = new Student();
```

此处的list中的数据就不会释放，即使内存不足他也会抛出错误，而不是清除数据。同样的，只要此引用存在，没有被释放（没有使student = null），垃圾回收器就永远不会回收。

2. **软引用（SoftReference）**

如果内存空间足够，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。

```java
import java.lang.ref.SoftReference;
 
public class SoftRef {  
 
    public static void main(String[] args){  
        System.out.println("start");            
        Obj obj = new Obj();            
        SoftReference<Obj> sr = new SoftReference<Obj>(obj);  
        obj = null;  
        System.out.println(sr.get());  
        System.out.println("end");     
    }       
}  
 
class Obj{  
    int[] obj ;  
    public Obj(){  
        obj = new int[1000];  
    }  
}
```

所以，软引用可用来实现内存敏感的高速缓存。例如浏览器的后退按钮，这个后退时显示的网页内容可以重新进行请求或者从缓存中取出。

#### 3.3.3 **弱引用：**（WeakReference）

在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间**是否足够**，**都会回收它的内存**。注意：由于垃圾回收器是一个优先级很低的线程，因此不一定会马上发现那些只具有弱引用的对象。

**弱引用与软引用的区别在于：**只具有弱引用的对象拥有更短暂的生命周期。

```java
Student student = new Student();     //只要student还指向Student就不会被回收
WeakReference<Student> weakStudent = new WeakRefence<Student>(student);
```

当要获得WeakRefence引用的student时，可以使用如下方法：

```java
weakStudent.get();
//如果此方法返回的值为空，那么说明weakStudent指向的对象student已经被回收了。
```

#### 3.3.4 虚引用（directBuffer就是最直观的例子）

虚引用和前面的软引用、弱引用不同，它并不影响对象的生命周期。在java中java.lang.ref.PhantomReference类表示。如果一个对象与虚引用关联，则跟没有引用与之关联一样，在任何时候都可能被垃圾回收器回收。

虚引用主要用来跟踪对象被垃圾回收的活动。比如，当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会把这个虚引用加入到与之 关联的引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。

<font size='4'>PS：最后，**在实际程序设计中一般很少使用弱引用与虚引用，**使用软用的情况较多，这是因为软引用可以加速JVM对垃圾内存的回收速度，可以维护系统的运行安全，防止内存溢出（OutOfMemory）等问题的产生。</font>

### 3.4 **CMS和G1区别在哪？**

- G1垃圾回收器是compacting（整理）的，因此其回收得到的空间是连续的（注意是逻辑意义上的连续，而是eden和old代之间有关联性）。这避免了CMS回收器因为不连续空间所造成的问题。如需要更大的堆空间，更多的floating garbage。连续空间意味着G1垃圾回收器可以不必采用空闲链表的内存分配方式，而可以直接采用bump-the-pointer的方式；
- G1回收器的内存与CMS回收器要求的内存模型有极大的不同。**G1将内存划分一个个固定大小的region，每个region可以是年轻代、老年代的一个。**内存的回收是以region作为基本单位的（不是新生代、老年代了）；old区域是存储了来着eden区的引用，eden区域也会记录，**导致G1回收的时候更快，**因为发生GC的时候，只会扫描对应老年代的脏卡区域。
- CMS收集器以最小的停顿时间为目标的收集器。 G1收集器可预测垃圾回收的停顿时间（建立可预测的停顿时间模型）
- CMS收集器是使用“标记-清除”算法进行的垃圾回收，容易产生内存碎片（最后将不得不通过担保机制对堆内存进行压缩）；G1收集器使用的是“标记-整理”算法，进行了空间整合，降低了内存空间碎片。（这个就是第一点，只不过更直白）
- **CMS 的问题：**并发回收导致CPU资源紧张、无法清理浮动垃圾（新产生的垃圾只能在下一次CMS中处理）、并发失败（和第一个有关系，因为其他程序还在并发执行，要有预留的内存）、内存碎片；

### 3.5 关于CMS、G1垃圾回收器的重新标记、最终标记疑惑?

并发标记是用户应用一起进行的过程，不能避免有新的reference指向老生代的object，这个object我们很难直接判断它到底是死是活，唯一准确的方法就是停机再扫描一遍，这就使CMS与G1 算法本身失去意义了。

- 对于”产生变动“的对象：一个很“贼”的点子就是，我们权当这类object都是活的，且从它们出发再（小范围）的trace一下遇到的object也是活的，这个过程就叫重新标记。因此，在并发标记的时候，记录下每个有新reference指向老生代的object（“这里的产生变动”指的就是这个），可以作为重新标记（stw）的出发点集合。**可以理解到：**其实我们最终得到的live set 是比真正的live set要大一些，因为会参杂一些实际上已死的对象（叫floating object）。下一次marking的“清洗”，因为已经没有reference再指向这类floating object。——CMS的重新标记原理（不能错杀）
- 对于“错过的对象”：对象可能会在 G1 收集期间死亡并且不会被收集。G1 使用一种称为开始快照 (SATB) 的技术来保证垃圾收集器找到所有活动对象。——G1最终标记原理（不能漏杀）：对并发标记前的活动对象快照，在最终标记期间重新标记。
  - 但是这还是不能解释G1为啥需要最终标记？原来是，G1的GC 中 SATB的Write Barrier 产生的标记并不是实时更新的，而会记录在本线程的 update buffer 中，到最终标记阶段，需要做的事情就是把这些 buffer 都给 flush 出来，完成所有标记，这点**与 CMS 的 Remark 有很大不同。**



## 4、Full GC效果不好 每次只能从90%-》85%之后又90%了，这种情况下应该怎么办比较好？

- 如果是一次fullgc后，剩余对象不多。那么说明你eden区设置太小，导致短生命周期的对象进入了old区。
- 如果一次fullgc后，old区回收率不大，那么说明old区太小。

（本质就是调整old区和eden区的大小）

## 5、垃圾收集器

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220310142704.png)

三大类垃圾收集器：

- 串行收集器（SerialXX）：效率高，单线程收集，STW；
  - 当内存不足时，串行GC设置停顿标识，待所有线程都进入安全点(Safepoint)时，应用线程暂停，串行GC开始工作，采用单线程方式回收空间并整理内存。
  - 串行收集器特别适合堆内存不高、单核甚至双核CPU的场合。
- 吞吐量优先（ParXX）：吞吐量高，并行收集，STW；
  - 暂停时并行地进行垃圾收集
  - 年轻代采用复制算法，老年代采用标记-整理，在回收的同时还会对内存进行压缩。
  - 调整新生代空间大小，来降低GC触发的频率。
  - 并且在满足最差延时的情况下，并行收集器将提供最佳的吞吐量。
- 并发标记清除（CMS）：并发收集，单线程收集垃圾，STW，不阻碍用户线程。（响应时间优先）——年轻代使用STW式的并行收集，老年代回收采用CMS进行垃圾回收。
  - GC ROOT进行初始标记（STW！！以STW的方式标记所有的根对象）
  - 并发标记（并发标记则同应用线程一起并行，标记出根对象的可达路径）
  - 重新标记（STW！！标记那些由mutator线程(指引起数据变化的线程，即应用线程)修改而可能错过的可达对象）
  - 并发清理（最后得到的不可达对象将在并发清除阶段进行回收）不需要stw

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220310144140.png)

- G1回收器（参考上面的`G1垃圾回收器`）

## 6、JVM调优

1. 根据业务场景（低延迟还是高吞吐）：选择合适的gc收集器
2. 查看CPU、内存（jstack、jhat）、I/O延时（jstat、jps）的状态，判断是否需要增大堆内存，或者调整新生代、老年代的比例。
3. 代码优化：对于对象的强、软、弱、虚引用调整
4. 使用G1收集器
5. 增大空间。



## 7、如何判断一个对象是否存活？

分为两种算法

1. **引用计数法：**给每一个对象设置一个引用计数器，当有一个地方引用该对象的时候，引用计数器就+1，引用失效时，引用计数器就-1；当引用计数器为0的时候，就说明这个对象没有被引用，也就是垃圾对象，等待回收；
   **缺点：**无法解决循环引用的问题，当A引用B，B也引用A的时候，此时AB对象的引用都不为0，此时也就无法垃圾回收，所以一般主流虚拟机都不采用这个方法；
2. **可达性分析算法**：从一个被称为**GC Roots**的对象向下搜索，如果一个对象到GC Roots没有任何引用链相连接时，说明此对象不可用。（主流！！！）

### 7.1 怎么解决检查引用消耗的问题？hotspot算法

问题：现在很多应用仅仅在方法区就有数百兆。如果要逐个检查里面的引用(我的理解就是检查栈内存里面所有的数据类型，但是里面只有一部分是引用类型)，势必消耗很多时间。

**解决：**

1. HotSpot使用一个叫Oopmap的数据结构来达到这个目的，在类加载完成时候，HotSpot就把对象内什么偏移量上是什么类型的数据计算出来，在JIT编译过程中，也会在特定位置记录下栈和寄存器中在哪些位置是引用，这样GC扫描的时候直接就知道了那些地方有引用信息.有了OopMap的协助下，HotSpot可以快速的完成GC Roots枚举.
2. HopSpot并没有为每条指令都生成OopMap，只有在特定的位置，即**安全点（safe Point）**才会记录引用信息，即程序执行时并非在所有地方都能停顿下来执行GC，而只有在到达安全点时才能暂停。
   - 安全点的选定基本上是以程序“是否具有让程序长时间执行的特征”为标准来选定的（这句话的意思是说:如果一个方法调用要花费很长时间，你不可能让GC等待方法调用完成后，再去进入安全点，这样就会导致GC要等好长时间，所以安全点的选定，就应该判断程序是否将要执行很长时间，如果是，就把安全点放到他们之后，如循环尾部，方法调用后，方法临返回前，抛异常的位置)），“长时间执行”最明显的特征就是指令序列复用，例如方法调用、循环跳转、异常跳转等，所以具有这些功能的指令才会产生Safepoint。
3. 但是如果有的线程不运行，如果他的状态是sleep或者block，线程是无法响应中断请求的，也就无法进入安全点,也就没有办法进行GC,因此就需要**安全域(Safe Region)**解决.Safe Region 是指在一段代码片段中，引用关系不会发生变化。在这个区域内的任意地方开始 GC 都是安全的。
   - 线程在进入 Safe Region 的时候先标记自己已进入了 Safe Region，等到被唤醒时准备离开 Safe Region 时，先检查能否离开，如果 GC 完成了，那么线程可以离开，否则它必须等待直到收到安全离开的信号为止



## 8、内存泄漏（memory leak）

> 说一下理解？它的应用？项目中的复现？
>
> 常见的：ThreadLocal的key弱引用导致的泄露；堆中元素（对象实例）没有及时释放；Netty的ByteBuf虚引用带来的泄露；Redis的字符串sdshdrXX在缩容的时候一直不调用sdsRemoveFreeSpace导致内存一直被占用
>
> 说实话，这种问题反而是非常实际的！他基本反映了在编程时的一些问题，也就是编程规范的问题，比如及时回收啊，及时声明啊这些，都是很有必要的。它的原理很简单，但是在实际代码中的反映却比较复杂，我们可以展开说一下！

- **内存泄漏**是指程序中已动态分配的堆内存由于某种原因程序未释放或无法释放，造成系统内存的浪费，导致程序运行速度减慢甚至系统崩溃等严重后果。（让JVM误以为此对象还在引用中，无法回收，造成内存泄漏；）
  - **严格来说，**只有对象不会再被程序用到了，但是GC又不能回收它们的情况，才叫内存泄漏。但实际情况很多时候一些不太好的实践（或疏忽）会导致**<font color='red'>对象的生命周期变得很长甚至导致OOM，</font>**也可以叫做宽泛意义上的“内存泄漏”。(我觉得这个就是核心！说白了就是生命周期不一致，短的没及时回收)
- 此外，内存泄漏通常不会直接产生可观察的错误症状，而是逐渐积累，降低系统整体性能，极端的情况下可能使系统崩溃。（特别是电脑/服务器经常不关机的情况）
  - 内存泄漏和内存溢出的关系：内存泄漏的增多，最终会导致内存溢出。但是内存溢出并不是一定要有内存泄漏；
  - **例如，最直观的一个例子，**当Y生命周期结束的时候，X依然引用着Y，这时候，垃圾回收期是不会回收对象Y的；如果对象X还引用着生命周期比较短的A、B、C，对象A又引用着对象 a、b、c，这样就可能造成大量无用的对象不能被回收，进而占据了内存资源，造成内存泄漏，直到内存溢出。

### 9.1Java中内存泄露的8种情况（这个也可以说是项目中遇到的问题）

1. **静态集合（内部）类。**如HashMap、LinkedList等等。如果这些容器为静态的，那么它们的生命周期与JVM程序一致，则容器中的对象在程序结束之前将不能被释放，从而造成内存泄漏。简单而言，长生命周期的对象持有短生命周期对象的引用，尽管短生命周期的对象不再使用，但是因为长生命周期对象持有它的引用而导致不能被回收。

```java
public class MemoryLeak {
    static List list = new ArrayList();
    public void oomTests(){
        Object obj ＝ new Object();  //局部变量,内存泄露，造成obj对象不能被回收
        list.add(obj);
        //可以主动的让 obj = null即可；但是这可能还不够，因为list还有obj的对象引用，我们还要把list.clear()或者list = null才行。
    }
}
```

上面展示了静态集合导致的问题，下面是静态内部类导致：

```java
public class MainActivity{
    private Info sInfo;		//指向了一个静态内部类，并实例化一个对象；
    private void onCreate(){
        if(sInfo!=null){
            sInfo = new Info(this);
        }
    }
    private void onDestory(){
        if(sInfo!=null){
            sInfo = null;
        }
    }
}
class Info{		//这是一个静态内部类；
    Activity ac;
    public Info(Activity activity){
        a = activity;
    }
}
```

虽然主程序的onCreate已经结束，但是sInfo并没有回收，因为它指向了一个静态内部类。**（这个问题经常出现）**；解决方法也简单，上面的onDestroy中把sInfo赋值为null就行了

2. **单例模式，**和静态集合导致内存泄露的原因类似，因为单例的静态特性，它的生命周期和 JVM 的生命周期一样长，所以如果单例对象如果持有外部对象的引用（外部类用了这个单例，`AppBean.getInstance(this)`），那么这个外部对象也不会被回收（即使这个外部对象的方法已经执行完成，他还是在栈里面），那么就会造成内存泄漏。
   - 如何避免：我们之前是直接用构造函数生成基于单例的上下文`this.Context=context`，它的生命周期太长了。我们可以改用应用程序的上下文`this.Context = context.getApplicationContext()`。

3. **内部类持有外部类。**由于内部类（匿名内部类、比如常见的Handler、Thread，AsyncTask等）持有外部类的实例对象，即使外部类实例对象不再被使用，这个外部类对象将不会被垃圾回收，这也会造成内存泄漏。

```java
public class MainActivity{
	@Override
	protected void onCreate(){
		new Thread(new Runnable(){
			@Override
			public void run(){
				try{
					Thrad.sleep(2000);
					MatinActivity.loadData();		//持有外部类的引用；可以把MainActivity变成软引用，并创建一个静态变量；并且加上一个判空的判断
				}catch(InterruptedException e){
					e.printStackTrace();
				}
			}
		}).start();
	}
}
```

<font color='red'>项目中遇到的情况，</font>比如，在MainActivity中可以开启一个线程，去请求一张图片。假如MainActivity返回了，但是线程还没有执行完（可能睡眠了），这时候线程还持有MainActivity的引用，线程再去调MainActivity的其它方法时就会返回空指针异常。(可以把MainActivity变成软引用)

4. **各种连接，如数据库连接、网络连接和IO连接等。**在对数据库进行操作的过程中，首先需要建立与数据库的连接，当不再使用时，需要调用close方法来释放与数据库的连接。只有连接被关闭后，垃圾回收器才会回收对应的对象。（如果在访问数据库的过程中，对Connection、Statement或ResultSet不显性地关闭，将会造成大量的对象无法被回收，从而引起内存泄漏）

```java
public static void main(String[] args) {
    try{
        Connection conn =null;
        Class.forName("com.mysql.jdbc.Driver");
        conn =DriverManager.getConnection("url","","");
        Statement stmt =conn.createStatement();
        ResultSet rs =stmt.executeQuery("....");
    } catch（Exception e）{//异常日志
    } finally {
        // 1．关闭结果集 Statement
        // 2．关闭声明的对象 ResultSet
        // 3．关闭连接 Connection
    }
}
```

5. **变量不合理的作用域（这个经常出现）。**
   - 一个变量的定义的作用范围大于其使用范围，很有可能会造成内存泄漏。
   - 另一方面，如果没有及时地把对象设置为null，很有可能导致内存泄漏的发生。

```java
public class UsingRandom {
    private String msg;
    public void receiveMsg(){
        msg = readFromNet();    //从网络中接受数据保存到msg中
        saveDB(msg);         //把msg保存到数据库中
        // private String msg;
        // msg = null
    }
}
```

这段代码就有两个错误，一个是msg在receiveMag中用完之后不能及时回收（这时，可以把把msg设置为null，这样垃圾回收器也会回收msg的内存空间）；

6. **改变哈希值。**（这个很好理解，也不应该去出现）当一个对象被存储进HashSet集合中以后，就不能修改这个对象中的那些参与计算哈希值的字段了。否则，对象修改后的哈希值与最初存储进HashSet集合中时的哈希值就不同了，在这种情况下，即使在contains方法使用该对象的当前引用作为的参数去HashSet集合中检索对象，也将返回找不到对象的结果，这也会导致无法从HashSet集合中单独删除当前对象（删不掉），造成内存泄漏。

```java
import java.util.HashSet;
 
public class ChangeHashCode {
    public static void main(String[] args) {
        HashSet set = new HashSet();
        Person p1 = new Person(1001, "AA");
        Person p2 = new Person(1002, "BB");
 
        set.add(p1);
        set.add(p2);
 
        p1.name = "CC"; //导致了内存的泄漏
        set.remove(p1); //删除失败
 
        //[Person{id=1002, name='BB'}, Person{id=1001, name='CC'}]
        System.out.println(set);
 
        set.add(new Person(1001, "CC"));
        //[Person{id=1002, name='BB'}, Person{id=1001, name='CC'}, Person{id=1001, name='CC'}]
        System.out.println(set);
 
        set.add(new Person(1001, "AA"));
        //[Person{id=1002, name='BB'}, Person{id=1001, name='CC'}, Person{id=1001, name='CC'}, Person{id=1001, name='AA'}]
        System.out.println(set);
 
    }
}
 
class Person {
    int id;
    String name;
 
    public Person(int id, String name) {
        this.id = id;
        this.name = name;
    }
 
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Person)) return false;
 
        Person person = (Person) o;
 
        if (id != person.id) return false;
        return name != null ? name.equals(person.name) : person.name == null;
    }
 
    @Override
    public int hashCode() {
        int result = id;
        result = 31 * result + (name != null ? name.hashCode() : 0);
        return result;
    }
 
    @Override
    public String toString() {
        return "Person{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}
```

7. **缓存泄露（生产环境中经常遇到）。**比如：之前项目在一次上线的时候，应用启动奇慢直到夯死，就是因为代码中会加载一个表中的数据到缓存（内存）中，测试环境只有几百条数据，但是生产环境有几百万的数据。

   可以使用WeakHashMap（软引用）代表缓存，此种Map的特点是，当除了自身有对key的引用外，此key没有其他引用那么此map会自动丢弃此值。

```java
public class MapTest {
    static Map wMap = new WeakHashMap();
    static Map map = new HashMap();
 
    public static void main(String[] args) {
        init();
        testWeakHashMap();
        testHashMap();
    }
 
    public static void init() {
        String ref1 = new String("obejct1");
        String ref2 = new String("obejct2");
        String ref3 = new String("obejct3");
        String ref4 = new String("obejct4");
        wMap.put(ref1, "cacheObject1");
        wMap.put(ref2, "cacheObject2");
        map.put(ref3, "cacheObject3");
        map.put(ref4, "cacheObject4");
        System.out.println("String引用ref1，ref2，ref3，ref4 消失");
 
    }
 
    public static void testWeakHashMap() {
        System.out.println("WeakHashMap GC之前");
        for (Object o : wMap.entrySet()) {
            System.out.println(o);
        }
        try {
            System.gc();
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("WeakHashMap GC之后");
        for (Object o : wMap.entrySet()) {
            System.out.println(o);
        }
    }
 
    public static void testHashMap() {
        System.out.println("HashMap GC之前");
        for (Object o : map.entrySet()) {
            System.out.println(o);
        }
        try {
            System.gc();
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("HashMap GC之后");
        for (Object o : map.entrySet()) {
            System.out.println(o);
        }
    }
 
}
/**
String引用ref1，ref2，ref3，ref4 消失
WeakHashMap GC之前
obejct2=cacheObject2
obejct1=cacheObject1
WeakHashMap GC之后
HashMap GC之前
obejct4=cacheObject4
obejct3=cacheObject3
HashMap GC之后
obejct4=cacheObject4
obejct3=cacheObject3
*/
```

8. **监听器和其他回调(这个也是项目中经常遇到的)**。如果客户端在你实现的API中注册回调，却没有显示的取消，那么就会积聚。需要确保回调立即被当作垃圾回收的最佳方法是只保存它的弱引用，例如将他们保存成为WeakHashMap中的键。

### 9.2 内存泄漏的案例

看一段手写HashMap的代码：

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
 
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
 
    public void push(Object e) { //入栈
        ensureCapacity();
        elements[size++] = e;
    }
 
    public Object pop() { //出栈
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }
 
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

代码的主要问题在pop函数，每一次pop一个数之后，elemets[size] 还是指向了Object，由于引用未进行置空，gc是不会释放的。

修改一下：

```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null;
    return result;
}
```

一旦引用过期，清空这些引用，将引用置空，这样垃圾回收器就会回收没有被引用的Object对象，防止内存泄漏。

### 9.3 怎么解决内存泄漏？或者说怎么排查？

> 上面已经给出了很多种内存泄漏的场景，和对应的解决方案了。这个问题如果再继续追问下去的话，其实就是**JVM调优相关的东西**~~
>
> 可以参考[10.3 OOM的排查流程。](https://blog.csdn.net/GUDUzhongliang/article/details/122671502)

1. 借助内存分析工具。为了找出到底是哪些对象没能被回收，我们加上运行参数-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=heap.bin，意思是发生OOM时把堆内存信息dump出来。运行程序直至异常，于是得到heap.dump文件，然后我们借助eclipse的MAT插件来分析

   - 在线上的应用，内存往往会设置得很大，这样发生OOM再把内存快照dump出来的文件就会很大，可能大到内存不足以打开这个dump文件

2. 利用操作系统定位排查。

   - 用jps定位到进程号：`jps -l`
   - 用jstat分析gc活动情况：`jstat -gcutil -t -h8 24836 1000`.

   ```
   C:\Users\spareyaya\IdeaProjects\maven-project\target\classes\org\example\net>jstat -gcutil -t -h8 24836 1000
   Timestamp         S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
              29.1  32.81   0.00  23.48  85.92  92.84  84.13     14    0.339     0    0.000    0.339
              30.1  32.81   0.00  78.12  85.92  92.84  84.13     14    0.339     0    0.000    0.339
              31.1   0.00   0.00  22.70  91.74  92.72  83.71     15    0.389     1    0.233    0.622
              //S0、S1、E是新生代的两个Survivor和Eden，O是老年代区，M是Metaspace，CCS使用压缩比例，YGC和YGCT分别是新生代gc的次数和时间，FGC和FGCT分别是老年代gc的次数和时间，GCT是gc的总时间。虽然发生了gc，但是老年代内存占用率根本没下降，说明有的对象没法被回收（当然也不排除这些对象真的是有用）。
   ```

   - 上面是命令意思是输出gc的情况，输出时间，每8行输出一个行头信息，统计的进程号是24836，每1000毫秒输出一次信息。
   - 用jmap工具dump出内存快照(效果和第一种处理办法一样，不同的是它不用等OOM就可以做到，而且dump出来的快照也会小很多。)`jmap -dump:live,format=b,file=heap.bin 24836`——这时会得到heap.bin的内存快照文件，然后就可以用eclipse来分析了。

我们进行jvm的主要目的是尽量减少停顿时间，提高系统的吞吐量。但是如果我们没有对系统进行分析就盲目去设置其中的参数，可能会得到更坏的结果，jvm发展到今天，各种默认的参数可能是实验室的人经过多次的测试来做平衡的，适用大多数的应用场景。



## 9、什么是内存溢出？

> 上面都讲了内存泄漏了，干脆讲一下内存溢出！一般说的溢出，其实就是虚拟机栈溢出（执行方法所需的数据）、堆溢出（对象实例）；方法区由于存的是常量，类信息，静态常量等信息，一般不溢出；



### 9.2 怎么实现栈溢出？怎么实现堆溢出？以及怎么解决？

> 对于栈溢出，还可以增加一下底层的攻击实验！关于修改寄存器。

栈是线程私有的，生命周期与线程相同，每个方法在执行的时候都会创建一个栈帧，用来存储局部变量表，操作数栈，动态链接，对象的引用，方法出口等信息。

**栈溢出**：方法执行时创建的栈帧个数超过了栈的深度。

**举例**：递归创建新方法。

```java
public class StackError {
    private int i = 0;

    public void fn() {
        System.out.println(i++);
        fn();
    }

    public static void main(String[] args) {
        StackError stackError = new StackError();
        stackError.fn();
    }
}
```

**解决方法**：调整JVM栈的大小：`-Xss 1024k`或者`-Xss 1m`(但是这个还是没有解决根本问题)——在IDEA中点击Run菜单的Edit Configuration

堆中主要存放的是对象。

**堆溢出：**不断的new对象会导致堆中空间溢出。如果虚拟机的栈内存允许动态扩展，当扩展栈容量无法申请到足够的内存时。

> 这还会出现一个新的问题——**内存抖动**——主要是频繁(很重要)在循环里创建对象(导致大量对象在短时间内被创建，频繁内存抖动会导致垃圾回收频繁运行，造成系统卡顿。(短时间内产生大量对象，需要大量内存，而且还是频繁抖动，就可能会需要回收内存以用于产生对象，垃圾回收机制就自然会频繁运行了)
>
> 怎么解决？ **复用**（线程池啊，direct buffer等）

```java
public class HeapError {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();

        try {
            while (true) {
                list.add("Floweryu");
            }
        } catch (Throwable e) {
            System.out.println(list.size());
            e.printStackTrace();
        }
    }
}
```

**解决方法**：调整堆的大小：`-Xmx83886080 -Xmx81920k -Xmx80m`

### 10.3 一般情况下OOM的排查方案

最直观的**内存溢出**的解决方案（实操类型的）：

1. 修改 JVM 启动参数，直接增加内存。 (-Xms ， -Xmx 参数一定不要忘记加。 )
2. 第二步，检查错误日志，查看 “ OutOfMemory ”错误前是否有其它异常或错误。
3. 对代码进行走查和分析，找出可能发生内存溢出的位置。重点排查如下：
   - 检查对数据库查询中，是否有一次获得全部数据的查询。一般来说，如果一次取十万条记录到内存，就可能引起内存溢出。**对于数据库查询尽量采用分页的方式查询**。
   - 代码中是否有死循环或递归调用。（栈溢出）
   - 检查是否有大循环重复产生新对象实体。（堆溢出）
   - 检查是否有大循环重复产生新对象实体。
4. 最后，使用内存查看工具动态查看电脑内存使用情况。
   - 用top命令查看cpu占用情况，可以定位cpu过高的程序进程。
   - 用`top -Hp pid`命令查看对应线程的情况
   - 用jstack工具查看线程栈情况`jstack 7268 | grep 1c77 -A 10`查看7268号（10进制）进程的1c77号（16进制）——在输出中可以看到正在执行的方法（`在执行com.spareyaya.jvm.service.EndlessLoopService.service这个方法`）;代码行号是19行，这样就可以去到代码的19行，找到其所在的代码块，看看是不是处于循环中，这样就定位到了问题。





# PS：小tips

## 1、 如何不用到第三个变量就能进行两个变量的值交换

巧妙运用两个变量的和

```java
		int a=2;
		int b=89;
		
		a=a+b;   //a=91;
		b=a-b;	 //91-89=2;
		a=a-b;	 //91-2=89;
        
```

## 2、java的乐观锁和悲观锁

JDK8中ConcurrentHashMap参考了JDK8 HashMap的实现，采用了数组+链表+红黑树的实现方式来设计，内部大量采用CAS（compare and swap）操作。

cas是一种基于锁的操作，而且是乐观锁。在java中锁分为乐观锁和悲观锁。

- 悲观锁是将资源锁住，等一个之前获得锁的线程释放锁之后，下一个线程才可以访问。
- 乐观锁采取了一种宽泛的态度，通过某种方式不加锁来处理资源，比如通过给记录加version来获取数据，性能较悲观锁有很大的提高。

CAS 操作包含三个操作数 —— 内存位置(V)、预期原值(A)和新值(B)。如果内存地址里面的值和A的值是一样的，那么就将内存里面的值更新成B。CAS是通过无限循环来获取数据的，若果在第一轮循环中，a线程获取地址里面的值被b线程修改了，那么a线程需要**自旋**（也是一种操作，并不是挂起），到下次循环才有可能机会执行。

- 自旋锁不会使线程状态发生切换，一直处于用户态，即线程一直都是active的；不会使线程进入阻塞状态，减少了不必要的上下文切换，执行速度快。

## 3、 阻塞队列了解过吗？

> 阻塞队列也是Queue队列的一种，叫BlockingQueue。
>
> AQS的实现也是基于双向阻塞队列的思想完成的。

BlockingQueue是线程安全的，所以很多时候我们可以利用这个特性，去解决业务中的问题，比如在使用生产者/消费者模式的时候，生产者只需要往队列里添加元素，消费者只需要从队列里取出它们就可以了。

阻塞队列关键的两个方法在于put 和 take，put的作用是插入元素，但是当队列满了的时候，不会抛出异常，也不会返回false，**而是让插入的线程处于等待状态，**直到队列有空闲空间，当其他线程调用take的时候，此时队列就会释放之前处于阻塞的线程，并把刚才那个元素添加进去。

**常见的阻塞队列：**

- ArrayBlockingQueue：最典型的有界队列，内部是用数组存储元素的，利用 ReentrantLock实现线程安全。
- LinkedBlockingQueue：LinkedBlockingQueue内部用链表实现的。LinkedBlockingQueue 也被称作无界队列，代表它几乎没有界限。
- SynchronousQueue：SynchronousQueue 最大的不同之处在于，它的容量为 0，所以没有一个地方来暂存元素，导致每次取数据都要先阻塞，直到有数据被放入，相反，每次放数据的时候也会阻塞，直到有消费者来取。
- PriorityBlockingQueue：支持优先级的无界阻塞队列
- **DelayQueue**：DelayQueue用于放置实现了Delayed接口的对象，Delayed 接口继承自 Comparable，里面的getDelay需要我们实现，getDelay 方法返回的是“还剩下多长的延迟时间才会被执行”，如果返回 0 或者负数则代表任务已过期，才能从队列中取走。




## 6、Java基本数据类型与封装类的区别？

1. 基本数据类型是值传递，封装类是引用传递

2. 基本数据类型是存放在栈中的，而封装类是存放于堆中的

3. 基本数据类型初始值如:int=0,而封装类Integer=null

4. **集合中添加的元素一定是封装类引用数据类型**

5. **声明基本数据类型不需要实例化可直接赋值，而封装类必须申请一个存储空间实例化才可赋值**。

### 6.1 map< , >前面那个能用int吗？

不能,泛型里面不能用基本数据类型,用对应的引用数据类型替代就好了,int用integer

## 7、equals与==的区别

对于**基本数据类型**：

- byte,short,char,int（注意这里不是integer泛型；integer泛型会在于int比较的时候，会自动拆箱为int型，其他时候不会拆箱）,long,float,double,boolean 等**基本数据类型**，他们之间的比较，应该双等号（==）,比较的是他们的值。——基本数据类型，equals和==都是一样的。（没有差别）

对于**引用类型的数据：**

- ==是判断两个变量或实例是不是指向同一个内存空间，equals是判断两个变量或实例所指向的内存空间的值是不是相同 
- 即：==是指对内存地址进行比较 ， equals()是对字符串的内容进行比较

```java
        String strA = "abc";
        String strB = "abc";	//strA和strB都在程序的栈空间，指向了静态元素区（字符串常量）
        String strC = new String("abc");	
        String strD = new String("abc");//strC和D则是在堆内存中。
        System.out.println(strA==strB);     //true
        System.out.println(strA.equals(strB));  //true
        System.out.println(strC==strD);     //false
        System.out.println(strC.equals(strD));      //true
```

## 8、hashCode和equals方法的区别与联系（只针对Object而言），如果是具体的对象（String），equals还是比较的值

问到这个问题的时候，回答的思路是：1. 先把equals与==的区别讲一下；2. 然后讲为什么有hashcode；3. 他们的联系；4.他们的区别；5. 如果不这样写，会出现什么问题？

什么情况下出现了hashCode？为什么要HashCode？

- 以场景切入，这个主要是为了应对查找嘛——>如果一个链表或者说集合，如果是有序的，还能一个个查找，但是无序的呢？就很难查。而且，当元素较多时，逐一的比较效率势必下降很快。（注意：**hashCode是为了提高在散列结构存储中查找的效率，在线性表中没有作用。**）
- 于是有人发明了一种哈希算法来提高从该集合中查找元素的效率，这种方式将集合分成若干个存储区域（可以看成一个个桶），每个对象可以计算出一个哈希码（这里又引出哈希偏移（指分布不离散，偏向某一区域），甚至是哈希冲突的情况——特别是分布式事务中的一致性Hash问题，在选择主机的时候，为了应对哈希偏移，可以把主机在不同的哈希段上做复制！！）
- 根据哈希码分组，每组分别对应某个存储区域，这样一个对象根据它的哈希码就可以分到不同的存储区域（不同的桶中）。**实际的使用中，**一个对象一般有key和value，可以根据key来计算它的hashCode。

那到底为什么要重写两个呢？（注意，如果不做说明，equals针对引用对象Object而言，**没有重写，**Object中的equals方法，比较的就是两个**对象的地址**(就是使用==来比较的)）

**他们的区别：**

JDK对equals(Object obj)和hashCode()这两个方法的定义和规范：

- 在Java中任何一个**对象**都具备equals(Object obj)和hashCode()这两个方法，因为他们是在**Object类**中定义的。 （可以看到还是有很大差别的！！）
  - equals(Object obj)方法用来判断**两个对象的内存地址**是否“相同”，如果“相同”则返回true，否则返回false。（Boolean类型返回）
  -  hashCode()方法返回一个int数，在Object类中的默认实现是“将该对象的内部地址转换成一个整数返回”。（int型返回）

所有散列函数都有如下一个基本特性：
1：如果a=b，则h(a) = h(b)。
2：如果a!=b，则h(a)与h(b)可能得到相同的散列值。

**所以：**

​    3.**若两个对象equals返回true，则hashCode有必要也返回相同的int数。**

​    4.若两个对象equals返回false，则hashCode不一定返回不同的int数,但为不相等的对象生成不同hashCode值可以提高哈希表的性能。

​    5.若两个对象hashCode返回相同int数，则equals不一定返回true。

​    6.**若两个对象hashCode返回不同int数，则equals一定返回false**。（很像是必要条件和充分条件的考点！！）

**他们的联系：**

- 其实上面的定义已经看出来了，HashCode和equals其实没啥联系，但是我们在**重写**hashcode和equals的时候应该满足一个规范：
  - hashCode和equals返回值应该是稳定的，不应有随机性。（比如hashcode中加了random方法，就不是稳定的）
  - 俩对象==返回true则这两个对象的equals也应该返回true。
  - 俩对象equals则这两个对象的hashCode应该相等。
- 但是他们**在集合中是需要联合使用的**。比如，HashMap中，先用hashcode(key)找到桶，然后对于hash冲突的点，他们会放到一个链表（红黑树）中，这个时候我们就要对比key值是否equals。
  - 所以，如果我们用了 hashmap 之类的库（**里面的逻辑导致 hashcode eqals 强关联**）。所以一个结论就是：重写equal()方法时也要重新hashCode,防止出现安全性问题（没有去重）——注意前后顺序，先equal方法，后hashCode方法（因为根据性质，“若两个对象equals返回true，则hashCode有必要也返回相同的int数”）。
- 如果只是单纯地想在某个属性值相等时对象相等，重写 equals（[具体的重写方法可以参考](https://blog.csdn.net/weixin_44203158/article/details/109961146)） 即可。（否则equals就是比较的引用地址，绝大多数情况下是不相等，导致hashcode中的equals返回为false，就判断两个数不一样，就都输出）

### 8.1 为什么重写equals方法，还必须要重写hashcode方法

1. **为了提高效率**： 采取重写hashcode方法，先进行hashcode比较，如果不同，那么就没必要在进行equals的比较了，这样就大大减少了equals比较的次数，这对比需要比较的数量很大的效率提高是很明显的，一个很好的例子就是在集合（HashMap）中的使用。

2. **为了保证同一个对象**：保证在equals相同的情况下hashcode值必定相同，如果重写了equals而未重写hashcode方法，可能就会出现两个没有关系的对象equals相同的（因为equal都是根据对象的特征进行重写的，可以是值判断），但hashcode却是不相同的（因为会使用从Object继承来的本地hashCode()方法，两个对象虽然在值上是相等的，但是他们的hash值不相等）。
3. [实际操作参考](https://blog.csdn.net/bingxuesiyang/article/details/90041772)

## 9、String、StringBuffer与StringBuilder的区别

**String：**

- Java 提供了 String 类来创建和操作字符串。
- Java 提供了 String 类来创建和操作字符串。

和 String 类不同的是，StringBuffer 和 StringBuilder 类的对象能够被多次的修改，并且不产生新的未使用对象。

StringBuilder 类在 Java 5 中被提出，它和 StringBuffer 之间的最大不同在于 StringBuilder 的方法不是线程安全的（不能同步访问）。

**StringBuffer 和 StringBuilder 的区别：**

- 区别1：线程安全：StringBuffer 的所有公开方法都是 synchronized 修饰的，而 StringBuilder 并没有 synchronized 修饰。
- 区别2：缓冲区：StringBuffer 每次获取 toString 都会直接使用缓存区的 toStringCache 值来构造一个字符串。 StringBuilder 则每次都需要复制一次字符数组，再构造一个字符串。 所以， StringBuffer 对缓存区优化，不过 StringBuffer 的这个toString 方法仍然是同步的。

- 区别3：性能：StringBuilder 是没有对方法加锁同步的，所以毫无疑问，StringBuilder 的性能要远大于 StringBuffer。



## 10、为什么Java中Integer泛型定义的100相等，但是1000不等

```
Integer i1=1000;
Integer i2=1000;
System.out.println(i1==i2);//结果是false
```

在包装类  Integer 中存数字时 ：

- 如果数字在一个字符的范围之内(－128~127) == 比较的是数字大小;因为线程会在方法区（元空间）开辟一块常量池，这些数字就是都在这个常量池里面
- 如果超过这个范围，== 比较的就是这两个对象的地址，包装类创建两个不同的对象地址自然不同。

但是，事实上，我们一般不用包装类来定义基础类型，都是int。
