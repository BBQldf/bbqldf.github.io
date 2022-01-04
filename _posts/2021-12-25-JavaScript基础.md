---
layout:     post
title:     JavaScript基础
subtitle:   JavaScript-基本框架、BoM、DoM、JQuery等
date:       2021-12-25
author:     ldf
header-img: img/post-bg-JavaScript01.png
catalog: true
tags:
    - java基础
    - JavaScript
    - code
---

> 前端是学习java绕不开的一部分。前端的基础是html、css，但是真正要学习前端开发的话，其实是“从前往后”的。因为与后端开发是不一致的，所有这里的笔记，并不会特别详尽，但是基本架构还是可以保证的！

# 一、JS基本框架



- HTML（结构层）：超文本标记语言（Hyper Text Markup Language），决定网页的结构和内容
- CSS（表现层）：层叠样式表（Cascading Style Sheets），设定网页的表现样式。
- JavaScript（行为层）：是一种弱类型脚本语言，其源码不需经过编译，而是由浏览器解释运行，用于控制网页的行为

## 1、JavaScript框架

- **JQuery：**大家熟知的JavaScript库，优点就是简化了DOM操作，缺点就是DOM操作太频繁，影响前端性能；在前端眼里使用它仅仅是为了兼容IE6，7，8；
- **Angular：**Google收购的前端框架，由一群Java程序员开发，其特点是将后台的MVC模式搬到了前端并增加了模块化开发的理念，与微软合作，采用了TypeScript语法开发；对后台程序员友好，对前端程序员不太友好；最大的缺点是版本迭代不合理（如1代–>2 代，除了名字，基本就是两个东西；截止发表博客时已推出了Angular6）
- **React：**Facebook 出品，一款高性能的JS前端框架；特点是提出了新概念 【虚拟DOM】用于减少真实 DOM 操作，在内存中模拟 DOM操作，有效的提升了前端渲染效率；缺点是使用复杂，因为需要额外学习一门【JSX】语言；
- **Vue：**一款渐进式 JavaScript 框架，所谓渐进式就是逐步实现新特性的意思，如实现模块化开发、路由、状态管理等新特性。其特点是综合了 Angular（模块化）和React(虚拟 DOM) 的优点；
- **Axios：**前端通信框架；因为 Vue 的边界很明确，就是为了处理 DOM，所以并不具备通信能力，此时就需要额外使用一个通信框架与服务器交互；当然也可以直接选择使用jQuery 提供的AJAX 通信功能；

## 2、UI框架

- Ant-Design：阿里巴巴出品，基于React的UI框架
- ElementUI、iview、ice：饿了么出品，基于Vue的UI框架
- BootStrap：Teitter推出的一个用于前端开发的开源工具包
- AmazeUI：又叫“妹子UI”，一款HTML5跨屏前端框架

## 3、JavaScript构建工具（略）

- Babel：JS编译工具，主要用于浏览器不支持的ES新特性，比如用于编译TypeScript
- WebPack：模块打包器，主要作用就是打包、压缩、合并及按序加载

# 二、快速入门

## 1、基本语法入门

- 内部标签\<script> ...\</script>
- 外部引入src="js/hj.js"

**基本语法测试：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <!--JavaScript严格区分大小写-->
    <script>
        // 1. 定义变量   变量类型 变量名 = 变量值
        var score = 1 ;
        //alert(num)
        // 2. 条件控制

        if (score > 60 && score < 70){
            alert("60~70");
        }else if(score > 70 && score < 80){
            alert("70~80");
        }else{
            alert("other")
        }
    </script>
</head>
<body>

</body>
</html>

```

### 浏览器控制台使用：

用浏览器调试代码：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20211224202626.png)

## 2、数据类型

**基本数据类型：**

- 数值：js不区分小树和整数——统一用num表示

- 文本
- 图形
- 音频
- 视频

### **基本数据类型表示**

1. 变量：var

2. number

```text
123//整数123
123.1//浮点数123.1
1.123e3//科学计数法
-99//负数
NaN	//not a number
Infinity // 表示无限大
```

3. 字符串：‘abc’ “abc”（单双引号都行）

4. 布尔值：true，false

5. 逻辑运算：与（&&）、或（||）、非（！）

6. **比较运算符：**=（赋值）；==（等于：类型不一样，值一样，也会判断为true）；===（绝对等于：类型一样，值一样，结果为true）

7. null 和 undefined：
   - null 空
   - undefined 未定义

数组：Java的数组必须是相同类型的对象！JS中不需要这样，数组里面可以是上面的任意一种（数值、字符串、布尔型）

```js
//保证代码的可读性，尽量使用[]
var arr = [1,2,3,4,5,'hello',null,true];
//第二种定义方法
new Array(1,2,3,4,5,'hello');
```

- 取数字下标：如果越界了，就会 报undefined

9. 对象：
   - 对象是大括号，数组是中括号；
   - 每个属性之间使用逗号隔开，最后一个属性不需要逗号

```js
var person = {
	name:'Tom',
	age:3,
	tags:['js','java','web','...']
}
```

   - 取对象值：
```js
person.name
> "Tom"
person.age
> 3
```



<font size="5" color="red">**几个注意点：**</font>

这是一个JS的缺陷，坚持不要使用 == 去做比较

- NaN === NaN（false），这个与所有的数值都不相等，包括自己
- 只能通过isNaN（NaN）来判断这个数是否是NaN
- **浮点数问题：**console.log((1/3) === (1-2/3))（false），因为它会分步计算。尽量避免使用浮点数进行运算，存在精度问题！



## 3、几个特殊特性

### 3.1 流程控制

- if判断
- while循环，避免程序死循环
- for循环
- forEach循环：ES5.1特性

```js
var age = [12,3,12,3,12,3,12,31,23,123];
//函数
age. forEach (function (value) {
console.1og(value)
})
```

- for …in-------下标

```js
//for(var index in object){}
for(var num in age){
if (age.hasOwnProperty(num)){
	console.1og("存在")
	console.1og(age [num])
	}
}
```

### 3.2 Map和Set

ES6的新特性~

- Map

```js
//ES6  Map
//学生的成绩，学生的名字
// var names = ["tom"，"jack", "haha"];
// var scores = [100,90,80];
var map = new Map( [[' tom ',100],['jack',90],['haha' ,80]]);
var name = map.get('tom'); //通过key 获得value
map.set('admin' ,123456); //新增或修改
map.delete("tom"); //删除
```

- Set：无序不重复的集合

```js
set.add(2);//添加!
set.delete(1); //删除!
console.log(set.has(3));//是否包含某个元素 !
```

## 4、函数

两种方式（以绝对值函数为例）：

```js
//方式一：直接定义；一旦执行到return代表函数结束，返回结果！如果没有执行return，函数执行完也会返回结果，结果就是undefined
function abs(x){
    if(x>=0){
        return x;
        }ese{
        return -x;
    }
}
//方式二：function(x){…}这是一个匿名函数。但是可以吧结果赋值给abs，通过abs就可以调用函数！
//方式一和方式二等价！
var abs = function(x){
    if(x>=0){
        return x;
        }ese{
        return -x;
    }
}

//调用函数：
abs(10)//10
abs(-10) //10

```

- 参数问题：
  - javaScript可以传任意个参数，也可以不传递参数~
  - 需要增加判断

```js
//手动抛出异常来判断
if(typeof x!=='number'){
	throw 'not a number!'
}
```

- 关键字：arguments：代表，传递进来的所有参数，是一个数组！

```js
var abs = function(x){
    console.1og("x=>"+x);
    for (var i = 0; i <arguments.length;i++){
    	console. 1og (arguments [i]);
    }
    if(x>=0){
    return x;
    }e1se{
    return -x;
    }
}
```

- 关键字：...rest：获取除了已经定义的参数之外的所有参数。

```js
function aaa(a,b,...rest) {
console.1og("a=>"+a);
console.1og("b=>"+b);
console.1og(rest);
}
```

## 5、方法

方法就是把函数放在对象的里面，**对象**只有两个东西：**属性和方法**

```
var kuangshen = {
    name: ' 秦疆'，
    bitrh: 2000,
    //方法
    age: function () {
    //今年-出生的年
    var now = new Date().getFul1Year();
    return now-this.bitrh;
    }
}

//属性
kuangshen. name
//方法，一定要带()
kuangshen. age()
```

## 6、内部对象

### 6.1、Date

```js
var now = new Date(); //sat Jan 04 2020 10:47:06 GMT+0800 (中国标准时间)
now.getFul1year(); //年
now.getMonth(); //月0~11 代表月
now.getDate(); //日
now.getDay(); // 星期几
now.getHours(); //时
now.getMinutes(); //分
now.getseconds(); //秒
now.getTime(); //时间戳全世界统一1970 1.1 0:00:00毫秒数
console. 1og(new Date (1578106175991)) //时间戳转为时间
```

### 6.2、JSON&AJAX

这一块在SpringMVC中[已经讲过了](https://bbqldf.github.io/2021/12/15/springMVC%E5%9F%BA%E7%A1%8002-%E5%89%8D%E5%90%8E%E7%AB%AF%E6%95%B0%E6%8D%AE%E4%BA%A4%E6%8D%A2/)！

1. **在javascript中，一切皆为对象，任何js支持的类型都可以用JSON表示格式**

- 对象都用{}
- 数组都用[]
- 所有的键值对 都是用key:value

2. **AJAX**

- 原生的js写法 xhr异步请求
- jQuery封装好的方法$(#name).ajax("")
- axios请求

## 7、面向对象编程

> 原型对象
> javascript、java、c#------面向对象；但是javascript有些区别！

原本的：

- 类：模板
- 对象：具体实例

### 7.1 原型——Object.\_\_proto__

```js
var Student = {
    name :"qinjiang"，
    age: 3,
    run:function() {
        console.log(this.name +”run....");
        }
};
var xiaoming = {
    name: "xiaoming"
};

//原型对象
xiaoming.__proto__ = Student;	//这时候
//console.log(xiaoming.run())输出为 xiaoming run...；
//console.log(xiaoming.age())输出为 3 ；


var Bird = {
    fly: function () {
    console.log(this.name +”fly....");
    }
};
//小明的原型变成了Bird，console.log(xiaoming.run())输出错误！
xiaoming.__proto__ = Bird;
```

注意：

- 继承，每个对象都有一个`__proto__`属性，这个属性是用来标识自己所继承的原型。

### 7.2 class继承

> class关键字，是在ES6引入的

1、定义一个类、属性、方法

```js
//定义一个学生的类
class Student{
    constructor(name){
    	this.name = name; 
    }
    hello(){
        alert('he11o')
        }
}
var xiaoming = new student("xiaoming");	//新实例化一个对象
var xiaohong = new student("xiaohong");
xiaoming.hello()
```

2、class之间的继承

```js
//集成上面的Student父类
class XiaoStudent extends Student {
    constructor(name,grade){
    	super(name);	//这里要显示地写上，因为继承了父类的name属性
   		this.grade = grade;
    }
    myGrade(){
        alert('我是一名小学生');
    }
}
var xiaoming = new Student("xiaoming");
var xiaohong = new XiaoStudent("xiaohong",1);

/*
xiaohong.myGrade()	//输出“我是一名小学生”
xiaohong.grade()	//输出“1”
*/
```

本质：

- 也还是用到了上面的`__proto__`属性

<font size="5" color='red'>继承链：</font>

- JavaScript 对象有一个指向一个原型对象的链。当试图访问一个对象的属性时，它不仅仅在该对象上搜寻，还会搜寻该对象的原型，以及该对象的原型的原型，依次层层向上搜索，直到找到一个名字匹配的属性或到达原型链的末尾。

# 三、操作BOM对象（重点）

## 1、window对象

- window代表浏览器窗口

```js
window. alert(1)	//弹窗
window.innerHeight	//获取窗口的内部高度，会一直随着窗口大小变化
window.innerwidth

window.outerHeight	//获取的是浏览器整个高度
window.outerwidth
```

## 2、Navigator（不建议使用）

- Navigator封装了浏览器的信息

```js
Navigator {vendorSub: '', productSub: '20030107', vendor: 'Google Inc.', maxTouchPoints: 0, userActivation: UserActivation, …}
appCodeName: "Mozilla"
appName: "Netscape"
appVersion: "5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36"
```

大多数时候，我们不会使用navigator对象，因为会被认为修改!

## 3、location（重要）

可以被用作重定向，location代表了当前页面的URL信息

```js
hash: ""
host: "new-tab-page"
hostname: "new-tab-page"
href: "chrome://new-tab-page/"
origin: "chrome://new-tab-page"
pathname: "/"
port: ""
protocol: "chrome:"
reload: ƒ reload()	//刷新网页
location.assign("https://www.baidu.com")	//设置新的地址
```

## 4、document（内容DOM）

- document代表当前的页面，HTML DOM文档树

```js
document.title
'百度一下，你就知道'
```

- 获取具体的文档树节点

```js
<dl id="app">
<dt>Java</dt>
<dd> JavasE</dd>
<dd> JavaEE</dd>
</dl>
<script>
var dl = document . getElementById('app'); //这个就可以把上面的<dl></dl>的所有内容获取到
</script>
```

## 5、获取cookie——document.cookie

服务器端可以设置cookie为httpOnly，只读，别人就无法取到cookie

## 6、history——浏览器历史记录（不建议使用）

```js
history.back();		//后退
history.forward();		//前进
```

# 四、操作DOM对象（重点）

浏览器网页就是一个Dom树形结构！对这个结构可以做的事情：

- **更新：**更新Dom节点
- **遍历Dom节点：**得到Dom节点
- **删除：**删除一个Dom节点
- **添加：**添加一个新的节点

## 1、要操作一个Dom节点，就必须要先获得这个Dom节点！

<font size="5" color='red'>（JS大部分时间不是在做基础语法，而是在处理网络通信、操作DOM）</font>

```js
<div id="father">
	<h1>标题一</h1>
	<p id="p1">p1</p>
	<p class="p2">p2</p>	
</div>


//对应css选择器
var h1 = document.getElementsByTagName('h1');
var p1 = document.getElementById('p1');

var p2 = document.getElementsByClassName('p2');
var father = document.getElementById('father');

var childrens = father.children; //获取父节点下的所有子节点
// father.firstchild	//获取第一个节点
// father.lastchild		//获取最后一个节点
```

这是原生代码，之后我们尽量都使用jQuery();

## 2、更新节点

```js
<div id="id1">
</div>
<script>
var id1 = document.getElementById('id1');
</script>
```

- 获得这个id1之后，操作其**文本属性：**

`id1. innerText= '456'`修改文本的值
`id1. innerHTML='<strong> 123</strong>'`可以解析HTML文本标签

- **操作css属性：**

```js
id1.sty1e.co1or = 'yellow'; //属性使用字符串包裹
id1.style.fontsize= '20px'; // -转驼峰命名问题
id1.style.padding = '2em'
```

## 3、删除节点

删除节点的步骤：先获取父节点，再**通过父节点删除自己**

```js
var p1 = document.getElementById('p1');
var father = p1.parentElement;

father.removeChild(p1);	//方法1：直接删除
father.removeChild(father.children[0])	//方法2：通过下标删除
```

**注意：**删除多个节点的时候，children是在时刻变化的，删除节点的时候一定要注意。

## 4、插入节点

我们获得了某个Dom节点，假设这个dom节点是空的，我们通过**Object.innerHTML**方法就可以增加一个元素了，但是这个Dom节点已经存在元素了，我们就不能这么干了！**会产生覆盖**

- 追加（插入）

```js
<p id="js">Javascript</p>
<div id="list">
    <p id="se">JavaSE</p>
    <P id="ee">JavaEE</p>
    <p id="me">JavaME</p>
</div>
<script>
    var js = document . getElementById('js');
    var list = document. getE1 ementById('list');
    1ist. appendchild(js);//追加到后面
</script>
```

5、创建一个新的标签

```html
<script>
	var js = document.getElementById('js');//已经存在的节点
    var list = document.getElementById('list');

//通过JS创建一个新的节点
    var newP = document.creatElement('p');//创建一个p标签
    newP.id = 'newP';
    newP.innerText = 'Hello,Kuangshen';
	list.appendChild('newP');	//追加到原始网页


//创建一个标签节点（通过setAttribute这个属性可以设置任意的值）；setAttribute参数是键值对格式！
    var myScript = document.creatElement('script');
    myScript.setAttribute('type','text/javascript');
    list.appendChild('myScript');

    //修改背景颜色
    var body = document.getElementsByName('body');
    body[0].style.backgroundColor="red";
    
    //可以创建一个style标签,追加进去，来改变背景颜色
    var myStyle = document.creatElement('style');//创建了一个空style标签
    myStyle.setAttribute('type','text/css');
    myStyle.innerHTML = 'body{background-color:chartreuse;}';//设置标签内容
    
    document.getElementByTagName('head')[0].appendChild(myStyle);
</script>

```

- insertBefore——在前面插入（不重要。。。）

```html
<p id="js">Javascript</p>
<div id="list">
    <p id="se">JavaSE</p>
    <P id="ee">JavaEE</p>
    <p id="me">JavaME</p>
</div>


var ee = document.getElementById('ee');
var js = document.getElementById('js');	//要加入的新节点
var list = document.getElementById('list');
//要包含的节点.insertBefore(newNode,targetNode)
list.insertBefore(js,ee);

```

## 5、操作表单

> 表单是什么？form-----也是DOM树中的一个节点。在前端操作一些验证类型的表单操作，可以避免在服务器操作的开销！

常见的属性：

- 文本框----text
- 下拉框----select
- 单选框----radio
- 多选框----checkbox
- 隐藏域----hidden
- 密码框----password
- …

### 1). 获得要提交的信息

```js
<body>
    <form action = "post">
        <p>
            <span>用户名：</span><input type="text" id = "username" />
        </p>
        <!--多选框的值就是定义好的value-->
        <p>
            <span>性别：</span>
            <input type = "radio" name = "sex" value = "man" id = "boy"/>男
           	<input type = "radio" name = "sex" value = "woman" id = "girl"/>女
        </p>
    </form>
    <script>
    	var input_text = document.getElementById("username");
        //得到输入框的值
        input_text.value 
        //修改输入框的值
        input_text.value  = "value";
        


        var boy_radio = document.getElementById("boy");
        var girl_radio = document.getElementById("girl");

        //对于单选框，多选框等等固定的值，boy_radio.value只能取到当前的值
		boy_radio.value;	//直接取到“man”
        boy_radio.checked;//查看返回的结果，是否为true，如果为true，则被选中。
        girl_radio.checked = true;//赋值
        
    </script>
</body>
```

### 2). 提交表单

- md5加密密码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.bootcss.com/blueimp-md5/2.10.0/js/md5.min.js"></script>
</head>
<body>

<form action="#" method="post">
    <p>
        <span>用户名：</span><input type="text" id="username" name="user-name">
    </p>
    <p>
        <span>密码：</span><input type="password" id="password" name="user-pwd">
    </p>
    <!--绑定事件onclick——被点击-->
    <button type="submit" onclick="a()">提交</button>
</form>

<script>
    function a(){
        var name = document.getElementById("username");
        var pwd = document.getElementById("password");
        console.log(name.value);
        //md5算法
        pwd.value = md5(pwd);
        console.log(pwd.value);
    }
</script>

</body>
</html>
```

- 测试效果（在登录的一瞬间，密码长度会变长）：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20211227205548.png)

- 改进版——md5加密密码，表单优化：

```html
<!DOCTYPE html>
<html lang = "en">
    <head>
        <meta charset = "UTF-8">
        <title>Title</title>
        <!--MD5加密工具类-->
        <script src = "https://cdn.bootcss.com/blueimp-md5/2.10.0/js/md5.min.js">
        	
        </script>
    </head>
    <body>
        <!--表达绑定提交事件
			οnsubmit= 绑定一个提交检测的函数，true，false
			将这个结果返回给表单，使用onsubmit接收
		-->
        <form action = "https://www.baidu.com" method = "post" onsubmit = "return aaa()">
            <p>
            	<span>用户名：</span><input type="text" id = "username" name = "user-name"/>
        	</p>
            <p>
            	<span>密码：</span><input type="password" id = "user-pwd" />
        	</p>
            <input type = "hidden" id = "md5-password" name = "password"> 
            
            <!--绑定事件 onclick 被点击-->
            <button type = "submit">提交</button>
            
        </form>
        
        <script>
            //用函数验证一些md5加密，用console显示
            
        	function aaa(){
                alert(1);
                var username = document.getElementById("username");
                var pwd = document.getElementById("password");
                var md5pwd = document.getElementById("md5-password");
                //pwd.value = md5(pwd,value);
                md5pwd.value = mad5(pwd.value);
                //可以校验判断表单内容，true就是通过提交，false就是阻止提交
                return false;
                
            }
        </script>
        
    </body>
</html>
```

测试效果（提交的一瞬间，密码不会变长；md5效果，name属性在这里被提取）：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20211227205849.png)

# 五、jQuery

jQuery库，里面存在大量的JavaScript函数，用美元符号封装！

**三个网站：**

- 在线的网站（百度搜“cdn jquery 加速”）：https://code.jquery.com/；https://www.jq22.com/cdn/；https://www.bootcdn.cn/jquery/
- 在官网下载：https://jquery.com/
- 文档工具站：http://jquery.cuishifeng.cn/

## 1、公式：$(selector).action()

```html
<!DOCTYPE html>
<html lang = "en">
    <head>
        <meta charset = "UTF-8">
        <title>Title</title>
       // <script src="lib/jquery-3.4.1.js"></script>
        <script src="https://code.jquery.com/jquery-3.6.0.js"></script>
    </head>
    <body>
        <a href="" id = "test-jquery">点我</a>
        <script>
        	//选择器就是css选择器
            $('#test-jquery').click(function(){
                alert('hello,jQuery!');
            });
        </script>
    </body>
</html>
```

- 选择器

```html
//原生js，选择器少，麻烦不好记
//标签(tag)选择器
document.getElementByTagName();
//id选择器
document.getElementById();
//class选择器
document.getElementByClassName();

//jQuery css中的选择器它全部都能用！
$('p').click();//标签选择器
$('#id1').click();//id选择器，用“#”
$('.class1').click;//class选择器，用“.”
```

## 2、事件

三大类事件：

- **鼠标事件**
- **键盘事件**
- 其他事件

```
//鼠标事件
.mousedown()(jQuery)	//按下
.mouseenter()(jQuery)	//
.mouseleave()(jQuery)	//离开
.mousemove()(jQuery)	//移动
.mouseout()(jQuery)		//
.mouseover()(jQuery)	//点击结束
.mouseup()(jQuery)		//抬起
```

- 测试——获取鼠标当前坐标：

```html
<!DOCTYPE html>
<html lang = "en">
<head>
    <meta charset = "UTF-8">
    <title>Title</title>
    <script src="https://code.jquery.com/jquery-3.6.0.js"></script>
    <style>
        #divMove{
            width:500px;
            height:500px;
            border:1px solid red;
        }
    </style>
</head>
<body>
<!--要求：获取鼠标当前的一个坐标-->
mouse：<span id = "mouseMove"></span>
<div id = "divMove">
    在这里移动鼠标试试
</div>
<script>
    //当网页元素加载完毕之后，响应事件
    //$(document).ready(function(){})的简写就是下面的：
    $(function(){
        $('#divMove').mousemove(function(e){
            $('#mouseMove').text('x:'+e.pageX+"y:"+e.pageY)
        })
    });
</script>
</body>
</html>
```

- 测试效果（移动鼠标，x和y发生变化）：

<img src="https://raw.githubusercontent.com/BBQldf/PicGotest/master/20211228095022.png" alt="12" style="zoom:50%;" />

## 3、操作DOM

- 节点文本操作：

```html
<ul 1d="test-ul">
    <1i class="js" >JavaScript</1i>
    <li name="python">Python</1i>
</u1>


//原生代码
    //document.getElementsById('xxx');
    
$('#test-ul li[name=python]').text();//属性选择器，以纯文本获得值
$('#test-ul li[name=python]').text('设置值');//设置值
$('#test-ul').html();//一样的作用，获得值
$('#test-ul').html('<strong>123</strong>');//设置值
```

- CSS的操作：

```html
$('#test-ul li[name=python]').css({"color","red"});
```

- 元素的显示和隐藏：本质是`display:none`属性

```html
$('#test-ul li[name=python]').show();
$('#test-ul li[name=python]').hide();
```

- ajax()函数

```html
$('#form').ajax()

$.ajax({
	url:"test.html",
	context:document.body,
	success:function(){
	$(this).addClass("done");
}})

```

# 六、小技巧

[点击观看](https://www.bilibili.com/video/BV1JJ41177di?p=30)

1. 巩固JS代码——直接去看JQuery源码，看简单的游戏源码
2. 巩固HTML、CSS——爬取网站，在控制台中删除一些没必要的代码，然后down下来，修改对应位置看效果！





一些有意思的样式工具：

- layUI
- Element-UI
