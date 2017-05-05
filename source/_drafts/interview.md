---
title: 2017年初面试遇到的一些题
date: 2017-04-27 21:46:20
categories:
- 面试
---

# 基础

### 1、 清除浮动的方式

> 参考资料：
- [CSS中的浮动和清除浮动，梳理一下！](http://www.jianshu.com/p/09bd5873bed4)
- [What methods of ‘clearfix’ can I use?](http://stackoverflow.com/questions/211383/what-methods-of-clearfix-can-i-use/1633170#1633170)
- [The very latest clearfix reloaded](http://cssmojo.com/the-very-latest-clearfix-reloaded/)

**浮动元素会脱离文档流并向左/向右浮动，直到碰到父元素或者另一个浮动元素**

清除浮动主要有两种方式，分别是clear清除浮动和BFC清除浮动

- clear属性不允许被清除浮动的元素的左边/右边挨着浮动元素，底层原理是在被清除浮动的元素上边或者下边添加足够的清除空间
- BFC全称是块状格式化上下文，它是按照块级盒子布局的。

```css
/* 现代浏览器clearfix方案，不支持IE6/7 */
.clearfix:after {
  display: table;
  content: " ";
  clear: both;
}

/* 全浏览器通用的clearfix方案
  引入了zoom以支持IE6/7 */
.clearfix:after {
  display: table;
  content: " ";
  clear: both;
}
.clearfix{
  *zoom: 1;
}

/* 全浏览器通用的clearfix方案【推荐】
  引入了zoom以支持IE6/7
  同时加入:before以解决现代浏览器上边距折叠的问题 */
.clearfix:before,
.clearfix:after {
  display: table;
  content: " ";
}
.clearfix:after {
  clear: both;
}
.clearfix {
  *zoom: 1;
}
```

### 2、 `Session` 和 `Cookie`



### 3、 DOM 0级、2级，事件绑定的方法

> 参考资料：
- [DOM 事件深入浅出（一）](http://www.jianshu.com/p/8c41a302bb17)
- [DOM 事件深入浅出（二）](http://www.jianshu.com/p/3c2e465480be)

### 4、 居中的方法

> 参考资料：
- [CSS秘密花园： 垂直居中](http://www.w3cplus.com/css3/css-secrets/vertical-centering.html)
- [CSS的垂直居中和水平居中总结](https://juejin.im/post/582c04032f301e00594327d4)

```css
main {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

main {
  width: 18em;
  margin: 50vh auto 0;
  transform: translateY(-50%);
}

main {
  position: absolute;  /* 或fixed */
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  margin: auto;
}
```

### 5、 Map Set



### 6、 什么是http、https如何加密



# 简单算法

### 1、 数组去重

### 2、 35个人生日重复的概率

### 3、 累加1、3、5、7、9……

# 设计模式等

### 1、 实现LazyMan

### 2、 发布订阅模式

# React相关

### 1、 React大概原理流程

### 2、 Redux流程

# JS题

```js
for( i = 0, j = 0; i < 10, j < 6; i++, j++) { k = i + j }
for( i = 0, j = 0; i < 6, j < 10; i++, j++) { k = i + j }
```

```js
var result = []
function foo() {
  for ( i = 0; i < 3; i++ ) {
    result[i] = function () {
      alert(i)
    }
  }
}
foo()
result[0]()
result[1]()
result[2]()
```
