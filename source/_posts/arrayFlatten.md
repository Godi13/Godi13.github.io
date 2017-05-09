---
title: 一道关于 Array 深度展开的面试题
date: 2017-05-08 23:34:02
categories:
- 面试
---

今天面试的时候，考官出了这么一个题，写一个函数，输入`a = [1,[2,3,[4,5,6]]]`，输出`a = [1,2,3,4,5,6]`。当时我脑子有点短路，做了好久，给出了个不太符合要求的答案如下

```js
var a = [1,[2,3,[4,5,6]]]
var arr = []
function merge(a) {
  for(var i = 0; i < a.length; i++){
    if(Array.isArray(a[i])){
      merge(a[i])
    } else {
      arr.push(a[i])
    }
  }
  return arr
}
```

回家后稍微完善了下，符合了考官输入输出的要求了（不过也晚了）

```js
function flatten(a) {
  if (!Array.isArray(a)) return

  var arr = []

  return (function merge(a) {
    for(var i = 0, l = a.length; i < l; i++){
      if(Array.isArray(a[i])){
        merge(a[i])
      } else {
        arr.push(a[i])
      }
    }
    return arr
  })(a)
}
```

然后跟朋友聊天问问有啥简单的方法，朋友给了一个挺符合这个题的简单方法类似如下

```js
const flatten = (arr) => arr.toString().split(',').map(v => Number(v))
```

不过这个答案只适用于这道题，如果不是全数字的就不行了。于是在`GitHub`上搜索`array flatten`，看了看排名前三的源码，发现有好多不同的写法，比如

```js
function flatten() {
  var args = Array.from(arguments)
  do {
    args = [].concat.apply([], args)
  } while (args.some(Array.isArray))
  return args
}
```

也有这么写的

```js
function flatten (array) {
  var result = Array.prototype.concat.apply([], array)
  if (result.length === array.length) {
    return result
  }
  return flatten(result)
}
```

写法各不相同，还有好多看似类似，但细节上不一样的，所以关键就看性能了，好在[arr-flatten](https://github.com/jonschlinkert/arr-flatten)和[array-flatten](https://github.com/blakeembrey/array-flatten)中就都有`benchmark`，进行了各种比较

[arr-flatten](https://github.com/jonschlinkert/arr-flatten)这个项目中，在各种不同的、可以在 [npm](https://www.npmjs.com/) 上搜到的`array flatten`实现方法之间进行了比较，最后的结论是[array-flatten](https://github.com/blakeembrey/array-flatten)执行的效率最高

![](http://images.godi13.com/2017-05-09-flattenPerf.png)

[array-flatten](https://github.com/blakeembrey/array-flatten)项目里的`benchmark`主要是自己源码中细节不同的写法之间的差异，最后得出的最佳实践如下

```js
function flatten (array) {
  if (!Array.isArray(array)) {
    throw new TypeError('Expected value to be an array')
  }
  return flattenFrom(array)
}

function flattenFrom (array) {
  return flattenDown(array, [])
}

function flattenDown (array, result) {
  for (var i = 0; i < array.length; i++) {
    var value = array[i]

    if (Array.isArray(value)) {
      flattenDown(value, result)
    } else {
      result.push(value)
    }
  }

  return result
}
```

这套代码思路与我给出的答案是一样的，但由于细节上的不同，导致用`benchmark`测试，执行效率比我写的那个快了约20%
