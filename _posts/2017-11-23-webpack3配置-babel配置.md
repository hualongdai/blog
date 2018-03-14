---
title: Webapck3配置 - Babel配置
date: 2017-11-23
categories:
- 工具
tag: 
- Webpack
- Babel
---

在webpack配置中，babel配置必不可少，那么如何配置，怎么配置，配置什么。

安装babel

如果是想安装最新版本

```bash
npm install babel-loader@8.0.0-beta.0 @babel/core @babel/preset-env --save-dev
```

如果是想安装普通版本

```bash
npm install babel-loader babel-core babel-preset-env --save-dev
```
<!-- more -->
安装好后，我们在`webpack.config.js`配置如下

```js
{
  ...
  module: {
    rules: [
      {
        test: '/\.js$/',
        use: 'babel-loader',
        exclude: /node_modules/
      }
    ]
  }
  ...
}
```

这是最基本的配置，但是这个时候`babel`不知道用什么规范来编译，那么常见的`babel presets`都有那些?

- babel-preset-es2015
- babel-preset-es2016
- babel-preset-es2017
- babel-preset-env (包含了es2015 - es2017 和 latest)
- babel-preset-react
- babel-preset-stage 0 - 3 (包含0 - 3 4个阶段的预案规范)

上面我们已经安装了`@babel/preset-env`

那么配置文件修改如下
```js
{
  ...
  module: {
    rules: [
      {
        test: '/\.js$/',
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env']
          }
        },
        exclude: /node_modules/
      }
    ]
  }
  ...
}
```

上面的预置(preset)配置了，我们实现了es6，es7等 **语法** 的编译，但是我们在es6，es7等新的 **函数和方法** 任然不能被处理，那么在一些低版本的浏览器就会报错，例如：

- Generator
- Set
- Map
- Array.from
- Array.prototype.includes
...

那么这个时候就得引入`babel-polyfill` 或者 `babel-plugin-transform-runtime`

安装

```
npm install babel-polyfill --save
```

引用

在入口文件直接引入就OK了 `import 'babel-polyfill'`

安装`babel-plugin-transform-runtime`

```
npm install npm install babel-plugin-transform-runtime --save-dev
npm insatll babel-runtime --save
```

引用

```js
{
  ...
  module: {
    rules: [
      {
        test: '/\.js$/',
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env',{
              targets: {
                browsers: ['last 2 versions']            
              }
            }],
            plugins: ['@babel/transform-runtime']
          }
        },
        exclude: '/node_modules/'
      }
    ]
  }
  ...
}
```

当然由于实际开发中，babel的配置很多，所以我们一般都都把babel的配置写在`.babelrc`文件中

```js
// .babelrc
{
  "presets": [
    ["@babel/preset-env", {
      "targets": {
        "browsers": ["last 2 versions"]
      }
    }]
  ],
  "plugins": ["@babel/transform-runtime"]
}
```

那么 `babel-polyfill` 或者 `babel-plugin-transform-runtime` 有啥区别

> Babel uses very small helpers for common functions such as _extend. By default this will be added to every file that requires it. This duplication is sometimes unnecessary, especially when your application is spread out over multiple files.

> This is where the transform-runtime plugin comes in: all of the helpers will reference the module babel-runtime to avoid duplication across your compiled output. The runtime will be compiled into your build.

> Another purpose of this transformer is to create a sandboxed environment for your code. If you use babel-polyfill and the built-ins it provides such as Promise, Set and Map, those will pollute the global scope. While this might be ok for an app or a command line tool, it becomes a problem if your code is a library which you intend to publish for others to use or if you can’t exactly control the environment in which your code will run.

> The transformer will alias these built-ins to core-js so you can use them seamlessly without having to require the polyfill.

总结：

babel-polyfill：

会造成全局污染，适合实际开发应用

babel-plugin-transform-runtime：

不会造成全局污染，更适合开发第三方框架、库

[这里](https://segmentfault.com/q/1010000005596587?from=singlemessage&isappinstalled=1) 有个不错的讲解

后续更新：

可能你配置完，你会发现，preset 和 plugin 感觉功能重合了，他们有区别吗？

答案是有的，具体区别

- plugins优先于presets进行编译
- plugins按照数组的index增序(从数组第一个到最后一个)进行编译
- presets按照数组的index倒序(从数组最后一个到第一个)进行编译