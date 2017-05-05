---
title: 记录集成 coveralls 时遇到的一坑
date: 2017-05-04 14:46:20
tags:
- coveralls
---

这两天想试一下[coveralls](https://coveralls.io/)，遂找了一个测试比较全的，使用`mocha`的`React + Redux`的[Todo项目](https://github.com/lewis617/react-redux-book/tree/master/14-15-todomvc)试验了一下，想着在这个基础上做一做练习，用用`karma`、`Jest`、`AVA`，结果刚开始就遇到了一个坑，折腾了好久才解决。

这个坑的大概是由于测试文件中使用了`ES6`语法，使用`istanbul cover`命令的时候就会报错，在网上找了很多方法试验，最终是成功的方法记录如下

1、 安装必要的包

```bash
yarn add cross-env babel-plugin-istanbul nyc --dev
```

2、 在`package.json`中添加如下内容

```json
{
  …
  "nyc": {
    "require": [
      "babel-register"
    ],
    "reporter": [
      "lcov",
      "text"
    ],
    "include": [
      # 测试文件路径，如 "test/**/*.spec.js"
    ],
    "sourceMap": false,
    "instrument": false
  }
  …
}
```

或者新建一个`.nycrc`文件放入

```json
{
  "require": [
    "babel-register"
  ],
  "reporter": [
    "lcov",
    "text"
  ],
  "include": [
    # 测试文件路径，如 "test/**/*.spec.js"
  ],
  "sourceMap": false,
  "instrument": false
}
```

3、 在`.babelrc`的`env`中添加

```json
{
  …
  "env": {
    …
    "test": {
      "plugins": [
        [
          "istanbul", {
            "exclude": [
              # 测试文件路径，如 "**/*.spec.js"
            ]
          }
        ]
      ]
    }
  }
}
```

4、 在`package.json`的`scripts`中加入

```
"test:cov": "cross-env NODE_ENV=test nyc --reporter=text --reporter=lcov mocha test/**/*.spec.js"
```

5、 在终端执行`yarn test:cov`，最终成功效果如下

![](http://images.godi13.com/2017-05-05-test.png)

6、 最后再添加上`.travis.yml`和`.coveralls.yml`文件，再通过一些配置，就可以有 [![Build Status](https://travis-ci.org/Godi13/Todo-React-Redux.svg?branch=master)](https://travis-ci.org/Godi13/Todo-React-Redux) [![Coverage Status](https://coveralls.io/repos/github/Godi13/Todo-React-Redux/badge.svg?branch=master)](https://coveralls.io/github/Godi13/Todo-React-Redux?branch=master) 这种徽章了。这里就不再赘述了，可以看下面的参考资料

# 参考资料

- [前端开源项目持续集成三剑客](http://efe.baidu.com/blog/front-end-continuous-integration-tools/)
- [如何让你的github项目更加高大上](http://www.tuicool.com/articles/rIzIZbe)
- [.travis.yml配置](https://docs.travis-ci.com/user/languages/javascript-with-nodejs/)
- [nyc官网](https://istanbul.js.org/)
