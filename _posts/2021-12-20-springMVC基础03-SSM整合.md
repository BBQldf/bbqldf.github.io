---
layout:     post
title:     springMVC基础03
subtitle:   SSM整合
date:       2021-12-20
author:     ldf
header-img: img/post-bg-springMVC01.jpg
catalog: true
tags:
    - java基础
    - springMVC
    - code
---



# 一、整合SSM框架

## 1、环境要求

环境：

- IDEA
- MySQL 8.0.27
- Tomcat 9.0.55
- Maven 3.8.4

**要求：**

- 需要熟练掌握MySQL数据库，Spring，JavaWeb及MyBatis知识，简单的前端知识；

## 2、数据库环境

创建一个存放书籍数据的数据库表

```mysql
CREATE DATABASE `ssmbuild`;
 
USE `ssmbuild`;
 
DROP TABLE IF EXISTS `books`;
 
CREATE TABLE `books` (
`bookID` INT(10) NOT NULL AUTO_INCREMENT COMMENT '书id',
`bookName` VARCHAR(100) NOT NULL COMMENT '书名',
`bookCounts` INT(11) NOT NULL COMMENT '数量',
`detail` VARCHAR(200) NOT NULL COMMENT '描述',
KEY `bookID` (`bookID`)
) ENGINE=INNODB DEFAULT CHARSET=utf8;
 
INSERT  INTO `books`(`bookID`,`bookName`,`bookCounts`,`detail`)VALUES
(1,'Java',1,'从入门到放弃'),
(2,'MySQL',10,'从删库到跑路'),
(3,'Linux',5,'从进门到进牢');
```

## 3、基本环境搭建

### 1). 新建一Maven项目-ssmbuild ， 添加web的支持

### 2). 导入相关的pom依赖！

```xml
<dependencies>
   <!--Junit-->
   <dependency>
       <groupId>junit</groupId>
       <artifactId>junit</artifactId>
       <version>4.12</version>
   </dependency>
   <!--数据库驱动-->
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <version>5.1.47</version>
   </dependency>
   <!-- 数据库连接池 -->
   <dependency>
       <groupId>com.mchange</groupId>
       <artifactId>c3p0</artifactId>
       <version>0.9.5.2</version>
   </dependency>
 
   <!--Servlet - JSP -->
   <dependency>
       <groupId>javax.servlet</groupId>
       <artifactId>servlet-api</artifactId>
       <version>2.5</version>
   </dependency>
   <dependency>
       <groupId>javax.servlet.jsp</groupId>
       <artifactId>jsp-api</artifactId>
       <version>2.2</version>
   </dependency>
   <dependency>
       <groupId>javax.servlet</groupId>
       <artifactId>jstl</artifactId>
       <version>1.2</version>
   </dependency>
 
   <!--Mybatis-->
   <dependency>
       <groupId>org.mybatis</groupId>
       <artifactId>mybatis</artifactId>
       <version>3.5.2</version>
   </dependency>
   <dependency>
       <groupId>org.mybatis</groupId>
       <artifactId>mybatis-spring</artifactId>
       <version>2.0.2</version>
   </dependency>
 
   <!--Spring-->
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-webmvc</artifactId>
       <version>5.1.9.RELEASE</version>
   </dependency>
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-jdbc</artifactId>
       <version>5.1.9.RELEASE</version>
   </dependency>
</dependencies>
```

### 3). Maven资源过滤设置

```xml
<build>
   <resources>
       <resource>
           <directory>src/main/java</directory>
           <includes>
               <include>**/*.properties</include>
               <include>**/*.xml</include>
           </includes>
           <filtering>false</filtering>
       </resource>
       <resource>
           <directory>src/main/resources</directory>
           <includes>
               <include>**/*.properties</include>
               <include>**/*.xml</include>
           </includes>
           <filtering>false</filtering>
       </resource>
   </resources>
</build>
```

到此就是基本配置（几乎所有的操作都是这样的）

### 4). 建立基本结构和配置框架

- com.kuang.pojo——存放实体类
- com.kuang.dao——存放数据库
- com.kuang.service——存放具体服务servlet
- com.kuang.controller——存放整体调度操作
- mybatis-config.xml——额外的，配置mybatis

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
       PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
 
</configuration>
```

- applicationContext.xml——配置spring

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
 
</beans>
```

上面这两个还只是个框架，后面还要继续添加配置！

- database.properties——数据库配置文件

```properties
#mysql5.7的配置；有时也可能需要加时区
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/ssmbuild?useSSL=true&useUnicode=true&characterEncoding=utf8
jdbc.username=root
jdbc.password=123456

#mysql8.0需要加一个时区的配置；需要增加显式禁用SSL的设置;数据库名字要写ssmbuild（别写错了，写自己的名字）
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/ssmbuild?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=UTC
jdbc.username=root
jdbc.password=root
```

<font size="4">ok~，现在是框架也有了！</font>

## 4、Mybatis层编写

### 1). IDEA关联数据库

### 2). 编写Mybatis核心配置文件——mybatis-config.xml

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
       PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
   
   <typeAliases>
       <package name="com.kuang.pojo"/>
   </typeAliases>
   <mappers>
       <mapper resource="com/kuang/dao/BookMapper.xml"/>
   </mappers>
 
</configuration>
```

### 3). 编写数据库对应的实体类 com.kuang.pojo.Books

```java
package com.kuang.pojo;
 
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
 
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Books {
   
   private int bookID;
   private String bookName;
   private int bookCounts;
   private String detail;
   
}
```

记得在pom.xml中导入lombok插件

### 4). 编写Dao层的 Mapper接口——BookMapper.java（注意选择interface）

```java
package com.kuang.dao;
 
import com.kuang.pojo.Books;
import java.util.List;
 
public interface BookMapper {
 
   //增加一个Book
   int addBook(Books book);
 
   //根据id删除一个Book
   int deleteBookById(int id);
 
   //更新Book
   int updateBook(Books books);
 
   //根据id查询,返回一个Book
   Books queryBookById(int id);
 
   //查询全部Book,返回list集合
   List<Books> queryAllBook();
 
}
```

### 5). 编写接口对应的配置文件——BookMapper.xml 

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
       PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 
<mapper namespace="com.kuang.dao.BookMapper">
 
   <!--增加一个Book-->
   <insert id="addBook" parameterType="Books">
      insert into ssmbuild.books(bookName,bookCounts,detail)
      values (#{bookName}, #{bookCounts}, #{detail})
   </insert>
 
   <!--根据id删除一个Book-->
   <delete id="deleteBookById" parameterType="int">
      delete from ssmbuild.books where bookID=#{bookID}
   </delete>
 
   <!--更新Book-->
   <update id="updateBook" parameterType="Books">
      update ssmbuild.books
      set bookName = #{bookName},bookCounts = #{bookCounts},detail = #{detail}
      where bookID = #{bookID}
   </update>
 
   <!--根据id查询,返回一个Book-->
   <select id="queryBookById" resultType="Books">
      select * from ssmbuild.books
      where bookID = #{bookID}
   </select>
 
   <!--查询全部Book-->
   <select id="queryAllBook" resultType="Books">
      SELECT * from ssmbuild.books
   </select>
 
</mapper>
```

### 6). Service层接口和实现类

#### 6.1 接口——BookService

```java
package com.kuang.service;
 
import com.kuang.pojo.Books;
 
import java.util.List;
 
//BookService:底下需要去实现,调用dao层
public interface BookService {
   //增加一个Book
   int addBook(Books book);
   //根据id删除一个Book
   int deleteBookById(int id);
   //更新Book
   int updateBook(Books books);
   //根据id查询,返回一个Book
   Books queryBookById(int id);
   //查询全部Book,返回list集合
   List<Books> queryAllBook();
}
```

#### 6.2 实现类——BookServiceImpl

```java
package com.kuang.service;
 
import com.kuang.dao.BookMapper;
import com.kuang.pojo.Books;
import java.util.List;
 
public class BookServiceImpl implements BookService {
 
   //调用dao层的操作，设置一个set接口，方便Spring管理
   private BookMapper bookMapper;
 
   public void setBookMapper(BookMapper bookMapper) {
       this.bookMapper = bookMapper;
  }
   
   public int addBook(Books book) {
       return bookMapper.addBook(book);
  }
   
   public int deleteBookById(int id) {
       return bookMapper.deleteBookById(id);
  }
   
   public int updateBook(Books books) {
       return bookMapper.updateBook(books);
  }
   
   public Books queryBookById(int id) {
       return bookMapper.queryBookById(id);
  }
   
   public List<Books> queryAllBook() {
       return bookMapper.queryAllBook();
  }
}
```

OK，到此，底层需求操作编写完毕！（Mybatis层写完！）

## 5、Spring层编写

### 1). Spring整合dao层

编写Spring整合Mybatis的配置文件

spring-dao.xml (他应该和applicationContext.xml是同属于一个配置中)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:context="http://www.springframework.org/schema/context"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">
 
   <!-- 配置整合mybatis -->
   <!-- 1.关联数据库文件 -->
   <context:property-placeholder location="classpath:database.properties"/>
 
   <!-- 2.数据库连接池 -->
   <!--数据库连接池
       dbcp 半自动化操作 不能自动连接
       c3p0 自动化操作（自动的加载配置文件 并且设置到对象里面）
	   druid:
	   hikari:SpringBoot2.0默认使用
   -->
   <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
       <!-- 配置连接池属性 -->
       <property name="driverClass" value="${jdbc.driver}"/>
       <property name="jdbcUrl" value="${jdbc.url}"/>
       <property name="user" value="${jdbc.username}"/>
       <property name="password" value="${jdbc.password}"/>
 
       <!-- c3p0连接池的私有属性 -->
       <property name="maxPoolSize" value="30"/>
       <property name="minPoolSize" value="10"/>
       <!-- 关闭连接后不自动commit -->
       <property name="autoCommitOnClose" value="false"/>
       <!-- 获取连接超时时间 -->
       <property name="checkoutTimeout" value="10000"/>
       <!-- 当获取连接失败重试次数 -->
       <property name="acquireRetryAttempts" value="2"/>
   </bean>
 
   <!-- 3.配置SqlSessionFactory对象 -->
   <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
       <!-- 注入数据库连接池 -->
       <property name="dataSource" ref="dataSource"/>
       <!-- 配置MyBaties全局配置文件:mybatis-config.xml -->
       <property name="configLocation" value="classpath:mybatis-config.xml"/>
   </bean>
 
   <!-- 4.配置扫描Dao接口包，动态实现Dao接口注入到spring容器中 -->
   <!--解释 ：https://www.cnblogs.com/jpfss/p/7799806.html-->
   <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
       <!-- 注入sqlSessionFactory -->
       <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
       <!-- 给出需要扫描Dao接口包 -->
       <property name="basePackage" value="com.kuang.dao"/>
   </bean>
 
</beans>
```

### 2). Spring整合service层

spring-service.xml(他应该和applicationContext.xml是同属于一个配置中)：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans.xsd
   http://www.springframework.org/schema/context
   http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 1. 扫描service相关的bean -->
    <context:component-scan base-package="com.kuang.service" />

    <!--2. BookServiceImpl注入到IOC容器中
        也可以中注解：@Service；@Autowired-->
    <bean id="BookServiceImpl" class="com.kuang.service.BookServiceImpl">
        <property name="bookMapper" ref="bookMapper"/>
    </bean>

    <!-- 3.配置事务管理器（声明式） 不用横切事务（weaver）-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 注入数据库连接池 -->
        <property name="dataSource" ref="dataSource" />
    </bean>

    <!--4. aop横切事务支持！-->
</beans>
```

Spring层搞定！再次理解一下，Spring就是一个大杂烩，一个容器！

## 6、SpringMVC层编写

### 1). 首先把项目变成web支持

File->AddFrameworksSupport->webApplication（web 4.0）

### 2). SpringMVC核心配置

web.xml中添加调度（DispatcherServlet+乱码过滤） + spring-mvc.xml中添加注解和视图解析器

- **web.xml：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">


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


    <!--encodingFilter-->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>
            org.springframework.web.filter.CharacterEncodingFilter
        </filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--Session过期时间-->
    <session-config>
        <session-timeout>15</session-timeout>
    </session-config>

</web-app>
```

- **spring-mvc.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:context="http://www.springframework.org/schema/context"
      xmlns:mvc="http://www.springframework.org/schema/mvc"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans.xsd
   http://www.springframework.org/schema/context
   http://www.springframework.org/schema/context/spring-context.xsd
   http://www.springframework.org/schema/mvc
   https://www.springframework.org/schema/mvc/spring-mvc.xsd">
 
   <!-- 配置SpringMVC -->
   <!-- 1.开启SpringMVC注解驱动 -->
   <mvc:annotation-driven />
   <!-- 2.静态资源默认servlet配置-->
   <mvc:default-servlet-handler/>
 
   <!-- 3.配置jsp 显示ViewResolver视图解析器 -->
   <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
       <property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
       <property name="prefix" value="/WEB-INF/jsp/" />
       <property name="suffix" value=".jsp" />
   </bean>
 
   <!-- 4.扫描web相关的bean -->
   <context:component-scan base-package="com.kuang.controller" />
 
</beans>
```

### 3). 额外的一步—配置applicationContext.xml

- 如果用idea，可以直接点击上面的配置信息：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20211216184628.png)

- 或者直接在applicationContext.xml写入（两种方法不冲突，可以同时进行）：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
 
   <import resource="spring-dao.xml"/>
   <import resource="spring-service.xml"/>
   <import resource="spring-mvc.xml"/>
   
</beans>
```

至此，配置就完成了！！后面就是写视图层的操作了！

# 二、整合SSM功能

写功能：本质上，就是让Controller层和jsp层实现交互！！



## 1、查询书籍功能

### 1). 创建一个Controller实现类——BookController.java

```JAVA
package com.kuang.controller;

import com.kuang.pojo.Books;
import com.kuang.service.BookService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.List;

@Controller
@RequestMapping("/book")
public class BookController {

    //1.首先让Controller层调用 service层
    @Autowired
    @Qualifier("BookServiceImpl")
    private BookService bookService;

    //2. 实现功能：查询全部书籍，并且返回一个书籍展示页面
    @RequestMapping("/searchBook")
    public String list(Model model){
        //1. 调用业务层的方法，
        List<Books> list = bookService.queryAllBook();
        //2. 把结果返回给前端
        model.addAttribute("listmsg",list);
        return "searchBook";
    }

}

```

- 创建相应的jsp视图——searchBook.jsp

```jsp
<%--
  Created by IntelliJ IDEA.
  User: 705lab
  Date: 2021/12/16
  Time: 18:56
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>书籍展示</title>
</head>
<body>

<h1>
    书籍展示！！！
</h1>

<%--${listmsg}--%>
</body>
</html>
```

<font size='4' color='red'>直接测试，出现异常！！</font>

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20211216215024.png)

<font size="5">Error!!!解决方案参考[这里~](https://bbqldf.github.io/2021/12/18/springMVC%E5%9F%BA%E7%A1%8005-SSM%E6%8E%92%E9%94%99/)</font>

### 2). 美化jsp页面

- 编写首页 **index.jsp**

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>首页</title>
    <style>
      a{
        text-decoration: teal;
        color: brown;
        font-size: 18px;
        text-align: center;
        line-height: 38px; /*和h3一样的高度*/
        background: blanchedalmond;
        border-radius: 5px; /*添加圆角边框*/
      }
      h3{
        width: 180px;
        height: 38px;
      }
    </style>
  </head>
  <body>
  <h3>
    <a href="${pageContext.request.contextPath}/book/searchBook">进入书籍页面</a>
  </h3>
  </body>
</html>

```

- 修改书籍列表页面 searchBook.jsp（区分大小写！！！）

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>书籍列表</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- 引入 Bootstrap -->
    <link href="https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>

<div class="container">

    <div class="row clearfix">
        <div class="col-md-12 column">
            <div class="page-header">
                <h1>
                    <small>书籍列表 —— 显示所有书籍</small>
                </h1>
            </div>
        </div>
    </div>

    <div class="row">
        <div class="col-md-4 column">
            <a class="btn btn-primary" href="${pageContext.request.contextPath}/book/toAddBook">新增</a>
        </div>
    </div>

    <div class="row clearfix">
        <div class="col-md-12 column">
            <table class="table table-hover table-striped">
                <thead>
                <tr>
                    <th>书籍编号</th>
                    <th>书籍名字</th>
                    <th>书籍数量</th>
                    <th>书籍详情</th>
                    <th>操作</th>
                </tr>
                </thead>

                <tbody>
                <c:forEach var="book" items="${requestScope.get('listmsg')}">  <%--取书籍的信息，这里需要controller里面model定义的信息--%>
                    <tr>
                        <td>${book.getBookID()}</td>
                        <td>${book.getBookName()}</td>
                        <td>${book.getBookCounts()}</td>
                        <td>${book.getDetail()}</td>
                    </tr>
                </c:forEach>
                </tbody>
            </table>
        </div>
    </div>
</div>
```

查看效果：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20211217220302.png)



## 2、添加书籍功能

### 1). BookController 类编写,增加一个addBook的方法（添加完后返回到展示界面）

```java
@RequestMapping("/toAddBook")
public String toAddPaper() {
   return "addBook";
}
 
@RequestMapping("/addBook")
public String addPaper(Books books) {
   System.out.println(books);
   bookService.addBook(books);
   return "redirect:/book/searchBook";	/*返回到展示界面*/
}
```

### 2). 添加书籍视图addBook.jsp

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<html>
<head>
    <title>新增书籍</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- 引入 Bootstrap -->
    <link href="https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
<div class="container">

    <div class="row clearfix">
        <div class="col-md-12 column">
            <div class="page-header">
                <h1>
                    <small>新增书籍</small>
                </h1>
            </div>
        </div>
    </div>
    <form action="${pageContext.request.contextPath}/book/addBook" method="post">
        书籍名称：<input type="text" name="bookName" required><br><br><br>        <%--name属性和实体类Books.java中的变量名一样--%>
        书籍数量：<input type="text" name="bookCounts" required><br><br><br>
        书籍详情：<input type="text" name="detail" required><br><br><br>
        <input type="submit" value="添加">
    </form>

</div>
```

- **测试效果：**

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20211218094847.png)

## 3、修改(更新)书籍功能

### 1). BookController 类编写，增加一个updateBook方法（更新完后返回展示界面）

```java
    //4. 实现功能：  4.1 从主界面获得要修改书籍的信息，并跳转到增加书籍界面；
    @RequestMapping("/toUpdateBook")
    public String toUpdateBook(Model model, int id) {
        Books book = bookService.queryBookById(id);
        System.out.println("toUpdateBook:"+ book);
        model.addAttribute("Qbooks",book);
        return "updateBook";
    }
    //4. 实现功能：  4.2 从更新界面获得修改书籍的信息，并跳转回主界面；
    @RequestMapping("/updatedBook")
    public String updateBook(Model model, Books book) {
        System.out.println("updated Book:"+ book);
        bookService.updateBook(book);       /*在数据库中更新这个书籍的数据*/

        System.out.println(bookService.queryAllBook());

//        Books books = bookService.queryBookById(book.getBookID()); /*用更新的书的id来查询这本书作为新的books*/
//        System.out.println("更新的booID="+book.getBookID());
//        System.out.println("查询到的书籍数据："+books.getBookID()); /*这个还是那个更新的书籍信息*/

//        model.addAttribute("Qbooks", books);  /*再把这本书的信息放回到前端*/
        return "redirect:/book/searchBook";  /*这里可以直接重定向      */

    }
```



### 2). 添加书籍更新视图updateBook.jsp

- 直接在主视图的form表单上添加一个“修改”按钮：

```jsp
 <td>
                            <a href="${pageContext.request.contextPath}/book/toUpdateBook?id=${book.getBookID()}">更改</a>
```



## 4、删除功能

### 1). BookController 类编写，增加一个deleteBook方法（更新完后返回展示界面）

- **这里用RestFul风格的传参：**

```java
    //5. 实现功能：  5.1 从主界面获得删除书籍的id，并跳转回主界面；
    @RequestMapping("/del/{bookgogo}")
    public String deleteBook(@PathVariable("bookgogo") int id) {
        bookService.deleteBookById(id);
        return "redirect:/book/searchBook";
    }
```

### 2). 添加书籍更新视图deleteBook.jsp

直接在主视图的form表单上添加一个“删除”按钮：

```jsp
<td>
                            <a href="${pageContext.request.contextPath}/book/del/}">删除</a>
</td>
```

## 5、新增搜索功能

### 5.1 模拟增加新功能

这个新功能在上面的框架中没有实现！现在模拟新需求——搜索功能！具体实现方法（自底向上）：

1. **dao层——调用数据库**

   1.1 添加一个接口BookMapper.java

```java
    //搜索指定的名字的books；
	//@Param(该注解属于MyBatis)作为Dao层的注解,作用是用于传递参数,从而可以与SQL中的的字段名相对应
    Books queryBookByName(@Param("bookName") String bookName);
```

​		1.2 添加一个数据库配置文件BookMapper.xml

```xml
    <!--搜索相应的书籍-->
    <select id="queryBookByName" resultType="Books">
        select * from ssmbuild.books
        where bookName = #{bookName}
    </select>
```

ok!数据库这里就改完了，后面就是把上面的接口接入service层

1. **service层——调用dao层**

    2.1 把queryBookByName方法放到BookService中

```java
Books queryBookByName(String bookName);
```

​		2.2 在BookServiceImpl中覆写方法——相当于去调用dao层

```java
    @Override
    public Books queryBookByName(String bookName) {
        return bookMapper.queryBookByName(bookName);
    }
```

3. **Controller——调用service层**

   3.1编写servlet

```java
//    6. 实现功能：  6.1 从主界面获得要书籍的id，并跳转回主界面；
    @RequestMapping("/query")
    public String queryBook(String querryBookName, Model model) {
        System.out.println("querryBookName:"+querryBookName);
        Books book_query = bookService.queryBookByName(querryBookName);/*先把这本书的信息取出来*/
        System.out.println("books=>"+book_query);

        //1. 调用业务层的方法，
        List<Books> lst_query = new ArrayList<Books>();
        lst_query.add(book_query);
       // 2. 把结果返回给前端
        if (book_query==null){  //如果没查到，就返回全部书籍信息，并显示错误信息！
            lst_query = bookService.queryAllBook();
            System.out.println("没找到！");
            model.addAttribute("error","未查到名为“"+querryBookName+"“的书籍");
        }
        model.addAttribute("listmsg",lst_query);        //这个要和前面的查询所有书籍的名字一致；因为表单里面是直接引用的这个

       return "searchBook";
    }
```

3. **前端——调用Controller**

```jsp
        <%--查询书籍--%>
        <div class="col-md-4 column">
            <form action="${pageContext.request.contextPath}/book/query/" method="post" style="float: right" class="form-inline">
                <input type="text" name="querryBookName" class="form-control" placeholder="请输入要查询的书籍名称！">
                <input type="submit" value="查询" class="btn btn-primary">
            </form>
        </div>
```



### 5.2 增加业务实现

#### 1). BookController 类编写，增加一个queryBook方法（更新完后返回展示界面）

```
//    6. 实现功能：  6.1 从主界面获得要书籍的id，并跳转回主界面；
    @RequestMapping("/query")
    public String queryBook(String querryBookName, Model model) {
        System.out.println("querryBookName:"+querryBookName);
        Books book_query = bookService.queryBookByName(querryBookName);/*先把这本书的信息取出来*/
        System.out.println("books=>"+book_query);

        //1. 调用业务层的方法，
        List<Books> lst_query = new ArrayList<Books>();
        lst_query.add(book_query);
       // 2. 把结果返回给前端
        if (book_query==null){  //如果没查到，就返回全部书籍信息，并显示错误信息！
            lst_query = bookService.queryAllBook();
            System.out.println("没找到！");
            model.addAttribute("error","未查到名为“"+querryBookName+"“的书籍");
        }
        model.addAttribute("listmsg",lst_query);        //这个要和前面的查询所有书籍的名字一致；因为表单里面是直接引用的这个

       return "searchBook";
    }
```



#### 2). 在主界面添加一个查询框：

```jsp
<div class="row">
    <div class="col-md-4 column">
        <a class="btn btn-primary" href="${pageContext.request.contextPath}/book/toAddBook">新增</a>
    </div>
    <%--查询书籍--%>
    <div class="col-md-4 column">
        <form action="" method="">
            <input type="text" name="querryBookName" class="form-control" placeholder="请输入要查询的书籍名称！">
        </form>
    </div>
</div>
```





## 6、总结

### 6.1 BookController.java

```java
package com.kuang.controller;

import com.kuang.pojo.Books;
import com.kuang.service.BookService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.AbstractList;
import java.util.ArrayList;
import java.util.List;

@Controller
@RequestMapping("/book")
public class BookController {

    //1.首先让Controller层调用 service层
    @Autowired
    @Qualifier("BookServiceImpl")
    private BookService bookService;

    //2. 实现功能：查询全部书籍，并且返回一个书籍展示页面
    @RequestMapping("/searchBook")
    public String list(Model model){
        //1. 调用业务层的方法，
        List<Books> list = bookService.queryAllBook();
        //2. 把结果返回给前端
        model.addAttribute("listmsg",list);
        return "searchBook";
    }

    //3. 实现功能：  3.1 跳转到增加书籍界面；
    @RequestMapping("/toAddBook")
    public String toAddPaper() {

        return "addBook";       /*只用显示，所以只需要返回一个视图*/
    }


    //3. 实现功能：  3.2 把书籍加到数据库，跳转回书籍展示界面；
    @RequestMapping("/addBook")
    public String addBook(Books books) {
        System.out.println(books);      //bookid = 0;
        bookService.addBook(books);  /*这里面从addBook.jsp传回来的books没有bookid，虽然bookid是主键不能为空，但是bookid是自增的。
                            当主键定义为自增长后，这个主键的值就不再需要用户输入数据了，而由数据库系统根据定义自动赋值。*/
        return "redirect:/book/searchBook";        /*返回到展示界面*/
    }

    //4. 实现功能：  4.1 从主界面获得要修改书籍的信息，并跳转到增加书籍界面；
    @RequestMapping("/toUpdateBook")
    public String toUpdateBook(Model model, int id) {
        Books book = bookService.queryBookById(id);
        System.out.println("toUpdateBook:"+ book);
        model.addAttribute("Qbooks",book);
        return "updateBook";
    }
    //4. 实现功能：  4.2 从更新界面获得修改书籍的信息，并跳转回主界面；
    @RequestMapping("/updatedBook")
    public String updateBook(Model model, Books book) {
        System.out.println("updated Book:"+ book);
        bookService.updateBook(book);       /*在数据库中更新这个书籍的数据*/

        System.out.println(bookService.queryAllBook());

//        Books books = bookService.queryBookById(book.getBookID()); /*用更新的书的id来查询这本书作为新的books*/
//        System.out.println("更新的booID="+book.getBookID());
//        System.out.println("查询到的书籍数据："+books.getBookID()); /*这个还是那个更新的书籍信息*/

//        model.addAttribute("Qbooks", books);  /*再把这本书的信息放回到前端*/
        return "redirect:/book/searchBook";  /*这里可以直接重定向      */

    }

    //5. 实现功能：  5.1 从主界面获得删除书籍的id，并跳转回主界面；
    @RequestMapping("/del/{bookGgogo}")
    public String deleteBook(@PathVariable("bookGgogo") int id) {
        bookService.deleteBookById(id);
        System.out.println("删除："+"bookid="+id);
        return "redirect:/book/searchBook";
    }

//    6. 实现功能：  6.1 从主界面获得要书籍的id，并跳转回主界面；
    @RequestMapping("/query")
    public String queryBook(String querryBookName, Model model) {
        System.out.println("querryBookName:"+querryBookName);
        Books book_query = bookService.queryBookByName(querryBookName);/*先把这本书的信息取出来*/
        System.out.println("books=>"+book_query);

        //1. 调用业务层的方法，
        List<Books> lst_query = new ArrayList<Books>();
        lst_query.add(book_query);
       // 2. 把结果返回给前端
        if (book_query==null){  //如果没查到，就返回全部书籍信息，并显示错误信息！
            lst_query = bookService.queryAllBook();
            System.out.println("没找到！");
            model.addAttribute("error","未查到名为“"+querryBookName+"“的书籍");
        }
        model.addAttribute("listmsg",lst_query);        //这个要和前面的查询所有书籍的名字一致；因为表单里面是直接引用的这个

       return "searchBook";
    }



}

```

### 6.2 searchBook.jsp

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>书籍列表</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- 引入 Bootstrap -->
    <link href="https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>

<div class="container">

    <div class="row clearfix">
        <div class="col-md-12 column">
            <div class="page-header">
                <h1>
                    <small>书籍列表 —— 显示所有书籍</small>
                </h1>
            </div>
        </div>
    </div>

    <div class="row">
        <div class="col-md-4 column">
            <a class="btn btn-primary" href="${pageContext.request.contextPath}/book/toAddBook">新增</a>
            <a class="btn btn-primary" href="${pageContext.request.contextPath}/book/searchBook">显示全部书籍</a>
        </div>
        <div class="col-md-4 column"></div>

        <%--查询书籍--%>
        <div class="col-md-4 column">
            <span style="color: crimson;font-size: 20px;font-weight: bolder">${error}</span>
            <form action="${pageContext.request.contextPath}/book/query" method="post" style="float: right" class="form-inline">
                <input type="text" name="querryBookName" class="form-control" placeholder="请输入要查询的书籍名称！">
                <input type="submit" value="搜索" class="btn btn-primary">
            </form>
        </div>
    </div>

    <div class="row clearfix">
        <div class="col-md-12 column">
            <table class="table table-hover table-striped">
                <thead>
                <tr>
                    <th>书籍编号</th>
                    <th>书籍名字</th>
                    <th>书籍数量</th>
                    <th>书籍详情</th>
                    <th>操作</th>
                </tr>
                </thead>

                <tbody>
                <c:forEach var="book" items="${listmsg}">  <%--取书籍的信息，这里需要controller里面model定义的信息--%>
                    <tr>
                        <td>${book.getBookID()}</td>
                        <td>${book.getBookName()}</td>
                        <td>${book.getBookCounts()}</td>
                        <td>${book.getDetail()}</td>
                        <td>
                            <a href="${pageContext.request.contextPath}/book/toUpdateBook?id=${book.getBookID()}">更新</a> |
                            <a href="${pageContext.request.contextPath}/book/del/${book.bookID}">删除</a>
                        </td>
                    </tr>
                </c:forEach>
                </tbody>
            </table>
        </div>

    </div>
</div>
```

### 6.3 效果展示：

- 主页面（查询-index.jsp）：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20211219104935.png)

- 子界面（增、删、改、查）：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20211219105342.png)



<font size="5">学到这里，大家已经可以进行基本网站的单独开发。但是这只是增删改查的<font color="red">基本操作~~~</font></font>







