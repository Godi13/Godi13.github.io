---
title: Benchmark.js 与 jsPerf 试玩
date: 2017-04-13 20:16:45
categories:
- benchmark
---

今天群里有同学提问`“你们知道那种循环效率比较高吗？”`，然后我想起之前在[benchmark 怎么写](https://github.com/alsotang/node-lessons/tree/master/lesson10)看过、但还没试过的`Benchmark`，遂决定使用一下

使用`Benchmark`主要有两种方式，一种是本地跑，一种是使用[jsPerf](https://jsperf.com/)线上分享，具体使用方法可以看[使用Benchmark.js和jsPerf分析代码性能](https://segmentfault.com/a/1190000003486676)

我对比了下`for` `map` `reduce` `forEach`4种循环方式，其中`num`是`1~1000`的数组，`empty`是1000个`undefined`数组，结果如下图：

![](http://images.godi13.com/2017-04-14-loop.png)

发现`for`跑`undefined`数组比有数字的数组慢好多，而`map`跑有数字的比`undefined`的快

我还在本地测试了一下`var`与`let`在`for`循环中是否有差异

```js
const Benchmark = require('benchmark')
const suite = new Benchmark.Suite;

const arr = [0,1,2,3,4,5,6,7,8,9]

// add tests
suite.add('let', function() {
  for(let i = 0, len = arr.length; i < len; i++) {
    arr[i]
  }
})
.add('var', function() {
  for(var i = 0, len = arr.length; i < len; i++) {
    arr[i]
  }
})
// add listeners
.on('cycle', function(event) {
  console.log(String(event.target));
})
.on('complete', function() {
  console.log('Fastest is ' + this.filter('fastest').map('name'));
})
.run()
```

结果如下，发现`let`比`var`慢很多

```bash
let x 24,180,382 ops/sec ±0.85% (88 runs sampled)
var x 49,310,969 ops/sec ±0.96% (90 runs sampled)
Fastest is var
```

这些不测一测是不会知道有这么明显的差异的，接下来就是分析为什么了，这我就先不写了

有兴趣的朋友可以到这[http://jsperf.com/browse](http://jsperf.com/browse)看看，里面有250个最新的网友们的测试，昨天我随意点进去两个看了下，其中之一在测试有`console.log`跟没有之间的差别，还有一个测试了各种`DOM`选择方式的差异

还有很多更好的用法，可以参考这些github项目[fast-js](https://github.com/alsotang/fast-js)、[fast.js](https://github.com/codemix/fast.js)

我也是刚接触这个，有啥更好的认知了回头再补充

# 其它参考

- [你不懂JS: 异步与性能 第六章: 基准分析与调优](https://github.com/getify/You-Dont-Know-JS/blob/1ed-zh-CN/async%20&%20performance/ch6.md) 这篇写的非常详细，有兴趣深入研究的可以好好看一下
- [Bulletproof JavaScript benchmarks](https://calendar.perfplanet.com/2010/bulletproof-javascript-benchmarks/)
