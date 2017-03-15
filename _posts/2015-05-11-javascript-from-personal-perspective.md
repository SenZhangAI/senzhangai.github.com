---
layout: post
title: "JavaScript笔记"
description: "关于JavaScript的语言特点的分析，以及关键知识点整理"
keywords: "JavaScript, 笔记"
category: Programming
tags: [JavaScript]
---

看了一点JavaScript的书，《JavaScript基础与案例开发详解》，做一下笔记。
很多地方可能理解有误，毕竟只看了三天，所以如果需要，往后还有待进一步查证。

## 补充资料

[虚拟机随谈（一）：解释器，树遍历解释器，基于栈与基于寄存器，大杂烩](http://rednaxelafx.iteye.com/blog/492667)

## JavaScipt整体思路

### 应用场合

首先就我所知的那么一点点知识，主要应用于网络，至少是现在浏览器的主流标准。
> 是**前端**[<0;100;1m浏览器都支持的唯一编程语言，CoffeeScript、TypeScript、Dart都要编译成JavaScript才能使用。
在**JavaScript**以前，很多应用比如对密码的格式的正确性判断都要回调给服务器端，
这样一来一回耽误了不少时间，**JavaScript**的出现正好能让有些事情拜托浏览器实现。
通过浏览器翻译JS语言，就能更好地实现页面的动态响应之类的事情。

作为应用于网络的语言，那么语言规则主要满足几点：

1. 尽可能简洁，因为网络对文件大小要求严格。

2. 解释型动态语言，这样能尽快地刷新屏幕，避免等待。

### 解释性语言

作为应用解释性语言，语法尽可能简单，不会很严格，走到哪里是哪里，
总之就是浏览器收到JS指令之后赶紧翻译，翻译一出是一出。那么几个问题来了，

**后文无关性**

遇到了`</script>`语句。那么不管你在哪里出现，即使是注释里面，说不定你就是要结束，那么赶紧结束吧。
所以直接后面的代码就废掉了。

**不严格的语法要求**

遇到参数列表不匹配的情况，通常的编译型语言看成是**重载**而去查看最匹配的，
JS可没有那个闲工夫，找到就算，考虑重载多花时间啊。

### 基于对象

把所有的东西都看成是对象，就是**标识符和属性（值）的集合**，这样有两点好处：

1. 形式上统一,不仅自身语言形式上统一，而且很符合HTML的风格，感觉上应该跟MVC的设计构架很贴合。

2. 便于搜索。猜测其翻译的思路可能是把已有的对象做成查表，这样遇到下一句调用对象时就开始往上翻表查询。查到就算。
这么一来对象的作用域也就很重要的。

此外，最外围的global/window对象含有所有的其他标识符和属性，猜测用来提供资源的根节点。

既然是基于对象就要考虑继承的问题了，JS是依靠**原型（prototype）**实现。

## 关于原型（prototype）

原型是JS实现继承的机制，这样如果需要获得一个通用的算法，就不需要每个对象都得得实现了。

每个对象都具有prototype属性（包括prototype对象自身），但不是每个对象都能**直接**访问自身的prototype属性，
只有拥有construct内部属性的对象才能**直接**访问prototype属性。

### 原型继承体系

约定用Op/Sp/Fp 分别表示Object/String/Function的**构造函数的原型**。

Op指向null

Object显式指向Op，Sp/Fp隐式指向Op

String显式指向Sp，Function显式指向Fp

## 关于构造函数的语义

这个稍微难一些，没有细看，所以不加分析。如以后遇到需要补充。

## 关于作用域

层层嵌套的关系，内层找不到标识符就去外层找，外层找不到就往更外层找，都找不到就提示出错了。
这被称为**链式作用域(chain scope)**。
只有两个环境，全局环境和函数环境，要么隶属于全局环境，要么隶属于某个函数环境。

有个有趣的关键词是**闭包**，作用有点像创建一个**public**函数来访问**private**成员。
这是因为函数内的变量仅内部可见，要想修改就要在内部再定义个外部可访问的函数，
只是有点不明白，为什么要叫**闭包**呢？

实现要点是：

1. 创建一个内部函数`f_in()`

2. `return f_in()`

例如：

```javascript
var foo = ( function() {
  var secret = 'secret';
  // “闭包”内的函数可以访问 secret 变量，而 secret 变量对于外部却是隐藏的
  return {
    get_secret: function () {
    // 通过定义的接口来访问 secret
    return secret;
    },
    new_secret: function ( new_secret ) {
    // 通过定义的接口来修改 secret
    secret = new_secret;
    }
  };
} () );

foo.get_secret (); // 得到 'secret'
foo.secret; // Type error，访问不能
foo.new_secret ('a new secret'); // 通过函数接口，我们访问并修改了 secret 变量
foo.get_secret (); // 得到 'a new secret'
```

## DOM

首先应该是BOM（Browser Object Model, 浏览器对象模型）定义了浏览器的整体结构，非标准，各浏览器实现不完全一致。
alert函数就是BOM中window对象的常用方法。

而DOM（Document Object Model, 文档对象模型）仅关注载入的文档，即HTML标签对象，
作为HTML和XML的标准，动态访问文档的结构、内容、样式。

DOM是一个树状结构，根节点为document。

### 访问节点

#### 利用节点信息访问

属性名      |描述
:----------:|:------------:
nodeType  | 访问节点类型
nodeValue | 节点值
parentNode| 父节点引用
childNodes| 所有子节点
firstChild| 第一个子节点引用
lastChild | 最后一个子节点引用
previousSibling|前一个兄弟节点
nextSibling|后一个兄弟节点

#### 利用Id、TagName等访问

`getElementById()`、`getElementByTagName`等

#### 利用CSS选择器访问

```javascript
document.querySelector("select rull"); //返回第一个匹配结果
document.querySelectorAll("select rull"); //返回所有匹配结果
```

### 节点信息修改

```javascript
node.getAttribute("name"); //访问
node.setAttribute("name",value); //修改属性值
document.body.bgcolor = "red"; //方法2 直接对属性赋值
```

### 移动/创建节点

```javascript
node.insertBefore(newChild,refChild); //在refChild前插入新的子节点
node.replaceChild(newChild,oldChild); //替换
node.appendChild(newChild); //插入末尾的子节点
node.removeChild(oldChild); //删除，并返回删除节点的引用

document.createElement(tagName); //创建指定标签名的元素节点
document.createTextNode(text); //创建文本节点
document.createComment(comment); //创建注释
document.createDocumentFragment(); //创建文档片段
note.cloneNode(deep=true|false); //克隆节点，如deep==true，则包括所有子节点
```

所有create的节点都需要append等方法插入到DOM中，
这样的操作很耗资源，如果使用`Fragment`辅助一次解决能获得性能提升。

#### 动态生成元素

需要补充的是IE和非IE的标准不一样，

IE标准：

```javascript
document.createElement("<input type=radio name='group1' checked />");
```

非IE

```javascript
document.createElement("input");
```

这样为了保证跨平台兼容性不得不采用如下办法：

```javascript
try{
  // IE 专用
  node1 = document.createElement("<input type=radio name='group1' checked />");
  node2 = document.createElement("<input type=radio name='group1' />");
} catch(e) {
  // 非IE专用
  node1 = document.createElement("input");
  node2 = document.createElement("input");
}

node1.type = 'radio';
node1.name = 'group1';
node1.checked = true;

node2.type = 'radio';
node2.name = 'group1';

document.body.appendChild(node1);
document.body.appendChild(node2);
```

## 关于CSS

### CSS的格式

```
选择器 {
  属性1:值;
  属性2:值;
}
```

### CSS选择器

**类规则** .list

**ID规则** #header 理论上ID意思是唯一，如果getElementById出现重复则错误，
通常ID规则用来制定页面整体框架。

**嵌套规则** 例如 li a{...}

## 事件

JS中IE与非IE浏览器DOM标准对于事件的诸多定义都不相同，有几个点需要考虑：

### 事件注册和删除

这个是已经定义好了的，例如onclick事件。这被称为**监听器注册**。
方式包括直接嵌入DOM（**属性赋值方式**、**属性引用方式**、**纯脚本属性方式**）

以及实现JS和HTML分离的**事件接口方式**。
事件接口方式的关键字是``this.addEventListener(...)||this.attachEvent(...) //非IE||IE``

相对应的监听器删除关键字是``this.removeEventListener(...)||this.detachEvent(...) //非IE||IE``

###事件的目标

或者说事件源来自DOM哪个节点。关键字为``event.target||event.srcElement //非IE||IE``

## 常见用例

### 表单

表单是网页与服务器沟通的桥梁。在BOM中form作为所有表单元素的父节点存在。
如果一个表单元素（例如input元素）在form之外则无法与服务器沟通。

#### 表单的动态响应

常用的文本框事件是获取焦点，例如``this.userName.onfocus=function(){...}``
以及失去焦点 ``this.userName.onblur=function(){...}``

#### 表单的验证

常用的submit事件是正确性验证，方法是: (也常用onblur失焦事件来激活验证)

```javascript
this.elements[this.element.length-1].onclick = function(e){
    // 检测输入是否合规则
    }
```

#### 复选框对象

常用的checkbox复选框事件是全选/不选，通过checked属性判断是否选中

### 表格

主要关注表格的排序以及通过边框拖动大小，排序中注意本地语言（例如简体中文）的排序需要用到`localeCompare()`函数

### 网页word

也包括回复框中的输入，比如斜体、粗体、表情、对齐等。
需要用到`<iframe />`标签实现富文本编辑。`<frameset />`标签已过时，不适合，例如会引用很多子html文件，造成文档过于混乱。

### JavaScript动画

重点是计时，有两种方法：

1. 通过**循环**直到达到总帧，循环的实现用**递归**比较好。然而有一个缺点是，如果系统此时繁忙卡顿，计时将延迟。

2. 通过系统时间解决计时延迟问题。

```javascript
var duration = +new Date() -startTime
```

### 多媒体

重点关注图层，用`zIndex`实现。以及CSS的`opacity`实现透明度。

### Web拖动

重点是获取鼠标坐标，用`clientX`、`clientY`、`pageX`、`pageY`实现。

### Cookie

是客户端的临时数据存储，意为曲奇饼干，表示很符合客户喜好。

通过`var CanCookie = navigator.cookieEnabled`确定浏览器是否开启Cookie功能。增加健壮性。

## 资源加载策略

为了更快加载网页，

### DOM回调事件
首先加载内容，然后才加载图片。方式是回调事件。

```javascript
window.onload = function(){ // 在所有页面数据加载完成后触发，包括图片
  //do something...
}
```

`onload`回调函数可以**在DOM完成后自动执行一些事情，但对加速加载页面没有帮助**。

为了加速，需要另一种回调函数:

```javascript
//非IE
document.addEventListener('DOMContentLoaded', function(){
  //do something...
}, false);

//IE
document.onreadystatechange = function(){
  if(document.readState == 'complete'){
    //do something...
  }
}
```

### 图片预加载

即首先用`new Image();`或者`document.createElement("img");`来新建图片，并用`document.body.appendChild(img)`加入DOM，并行执行。或者加入进度条。

### CSS文件动态加载

简单说就是快速改变样式，只需要改一下css的src即可，速度很快。方式是：

```javascript
var head = document.getElementByTagName('HEAD').item(0);
var style = document.createElement('link');

style.rel = 'stylesheet';
style.type = 'text/css';
head.appendChild(style);
style.href = "someStyleVar" + ".css";
```

### Ajax

Ajax全称是**Asynchronous JavaScript and XML**，本质是**异步**。

异步的核心思想是只进行部分数据交互。一种异步的实现思想方式是**XMLHTTP**。

最简单的异步实现是`iframe`并将其隐藏`display:none`，但较麻烦，并非专门用于异步。

#### XMLHttpRequest

Ajax的核心控制器是XMLHttpRequest对象，创建方法为：

```javascript
// IE所有支持版本
var msXMLAllversion = ["Msxml2.xmlhttp.5.0", "Msxml2.xmlhttp.4.0",
  "Msxml2.xmlhttp3.0", "Msxml2.xmlhttp", "Microsoft.xmlhttp"]
var XHR = null;
// 判断是否原生支持XMLHttpRequest
if(window.XMLHttpRequest){
  XHR = new XMLHttpRequest(); //核心仅此一句
} else {
  for (var i = 0; i < msXMLAllversion.length; i++){
    try{
      XHR = new ActiveXObject(msXMLAllversion[i]);
    } catch(e) {
      continue; //这一个try-catch很精彩
    }
    break;
  }
}
```

##### 1. 请求设置和发送

`open(请求方式, 请求地址 [, 是否异步] [, 用户名] [, 密码])`

请求方式包括get、post、put等

例如：`XHR.open('get', 'server.html', true)`

##### 2. 请求交互判断

```javascript
//监听
XHR.onreadystatechange = function(){ // do something

  //readyState==4 表示请求数据下载完成
  //statue==200   HTTP中表示页面加载正常
  if(XHR.readyState==4 && XHR.statue==200){

  //处理回调事件
  }
};

```

##### 3. 返回值

通用的返回属性包括`responseText`和`responseXML`

### JSON

JSON是一种数据存储格式，全称为**JavaScript Object Notation**。比XML更简单、流行，因为格式相对简单，例如Sublime的设置就是JSON。

JSON只存储数字和字符串，用`eval(jsonString)`来解析JSON。

## JS的缺点

1. 没有标准库

2. 容易出错，出错后即停止，后续程序无法运行，可以用`try-catch`方式解决。
