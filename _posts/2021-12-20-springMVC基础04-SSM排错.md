---
layout:     post
title:      SSM
subtitle:   SSM整合
date:       2021-12-20
author:     ldf
header-img: img/post-bg-springMVC01.jpg
catalog: true
tags:
    - java基础
    - springMVC
    - debug
    - code
---

# 一、前言：问题展示

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20211216215024.png)

搭完框架，看一下结果，报错！第一件事，先用Junit测试一下，排除掉是框架不通的问题，bean是否注入成功。

# 二、Junit调试

针对单个Java方法的测试。这里我们需要测试

![测试结果](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20211217203834.png)

提示我，没有找到这个数据库！？

- 哦，原来是database.properties配置数据库id写错了（不要盲目复制网上的。。。）

<font size="5" color='red'>**再测一次：**</font>

![网页视图](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20211217211333.png)

![终端报错](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20211217211408.png)

提示我，没有找到这个Beans？？那就是装配问题！用Junit编写测试类，

```java
import com.kuang.pojo.Books;
import com.kuang.service.BookService;
import org.junit.Test;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Mytest {
    @Test
    public void test(){
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        BookService bookServiceImpl = (BookService) context.getBean("BookServiceImpl");
        for (Books books : bookServiceImpl.queryAllBook()) {
            System.out.println(books);
        }
    }
}
```

直接从spring配置文件中引入Beans，然后输出相关信息！

<font size="5" color='red'>**再测一次：**</font>

![测试成功](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20211217205134.png)

正常显示，但是这里有一条警告，[查一下情况？](https://blog.csdn.net/jacksonary/article/details/79009884)

- 最少连接数为10，最大连接数为30，查了一下c3p0的默认初始化时候给的连接数为3，也是，你给人家定个最小为10，初始化却为3，好了在上述配置文件中添加一下初始化连接数的配置即可，数量介于最小和最大连接数

在spring-dao.xml文件中修改，

```xml
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <!-- 配置连接池属性 -->
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>

        <!-- c3p0连接池的私有属性 -->
        <property name="maxPoolSize" value="30"/>
        <!-- 这个必须写在minPoolSize和maxPoolSize之间，表示初始化时获取的连接数，这个值一般设置为minPoolSize，缺省时默认为3 -->
        <property name="initialPoolSize" value="10"/>
        
        <property name="minPoolSize" value="10"/>
        <!-- 关闭连接后不自动commit -->
        <property name="autoCommitOnClose" value="false"/>
        <!-- 获取连接超时时间 -->
        <property name="checkoutTimeout" value="10000"/>
        <!-- 当获取连接失败重试次数 -->
        <property name="acquireRetryAttempts" value="2"/>
    </bean>
```

# 三、具体视图报错

测试通过以后，初步判定是spring装填没问题，数据库也没问题，那只可能是springMVC有问题！

所以去看SpringMVC的配置文件：spring-mvc.xml、web.xml

- spring-mvc.xml中只有视图解析器，不会出错！
- web.xml中存在问题。检查发现：
  - contextConfiguration参数导致SpringMVC绑定的是spring-mvc.xml文件了（这是以前在单独学SpringMVC的时候，绑定的，因为所有的配置信息都放在spring-mvc.xml中了），但是现在，spring-mvc.xml中只有视图解析器
  - 我们要绑定的是spring-service.xml，这里实现了Beans的装配
  - 而Junit能测试成功，原因就是，我们从ApplicationContext.xml文件引入的Beans，ApplicationContext.xml已经import了所有的配置文件！
- 所以，改一下配置，这里改成classpath:applicationContext.xml都可以
  - 改成classpath:classpath:spring-service.xml还是不行，会报错（org.springframework.beans.factory.BeanCreationException）；因为spring-dao中还有配置SqlSessionFactory对象操作~

```xml
    <servlet>
        <servlet-name>DispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
          <!--这里之前在SpringMVC中是<param-value>classpath:aspring-mvc.xml</param-value>
             但是这样，会导致spring拿不到bean（现在从spring入口）
             所以要放到最外层的applicationContext.xml文件中-->
            <param-value>classpath:applicationContext.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>DispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```

<font size="5" color='red'>**再测一次：**</font>

![测试正常](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20211217211832.png)

all right！

遇到问题不要慌~！去看报错，思考原因，一条条解决即可！
