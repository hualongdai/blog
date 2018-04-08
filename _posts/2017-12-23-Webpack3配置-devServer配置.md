---
title: Webapck3配置 - devServer配置
date: 2017-12-23
categories:
- 工具
tag: 
- Webpack
---

webpack dev server 配置
主要功能

- live reloading
- 路径重定向
- https
- 浏览器显示编译错误
- 接口代理
- 模块热更新

```js
{
  ...
  devSever: {
    proxy: { ... },
    historyApiFallback: { ... },
    hot: true
    ...
  }
}
```
<!-- more -->

### devServer 参数详解

historyApiFallback:  当我们使用 H5 的history api的时候，往往不会刷新页面，我们就通过往浏览器history 添加记录来实现页面跳转
```
// 简单使用
historyApiFallback: true  这样页面任何地址跳转都会跳转到 index.html  而不会出现404

// 高级使用
devSever: {
  historyApiFallback: {
    rewrites: [
      { form: '/pages/a', to: '/page/a.html' } // 访问 localhost/pages/a 会指向 localhost/page/a.html
      { from: /^\/([a-zA-Z0-9]+\/?)([a-zA-Z0-9]+)/, to: function(context) {
        return '/' + context.match[1] + context.match[2] + '.html'
      }} // 匹配 pages/a -> pages/a.html , pages/ -> pages.html
    ]
  }
}
```

proxy: 代理 (这个跟http-proxy-middleware 是一样的，可以看这个插件的 配置)

```
devServer: {
  proxy: {
    "/api": {
      target: "http://www.baidu.com",  // 如果 请求 '/api/list/v1' -> 'http://www.baidu.com/api/list/v1'
      changeOrigin: true, // 如果目标是虚拟主机必须设置这个  否则不能请求到，这个主要是改变 本地主机的host 为目标的url
      logLevel: 'debug', // 开启请求日志
      pathRewrites: {
        // 地址重定向
        '^/api/old-path' : '/api/new-path',
      },
      headers: {
        // 在这里 加Cookie 或者 UA 等等
      }
    }
}
}
```

热更新

```js
{
  devServer: {
    hot: true // 只单单这是这个不行  必须配合 下面两个插件
  },
  plugins: [
    new webpack.hotModuleReplacementPlugin(),
    new webpack.NamedModulePlugin()  // 这个不是必须  只是为了看相对路径输出
  ]
}

// 单单这样 我们看到确实能够检测到变化  但是不能实时生效

// 这里还需要在module 设置 module.hot = true
```

devtool

css sourcemap
  要在style/css/post-css loader中的options 得设置 sourcemap: true
js sourcemap
  如果是uglifyJS 压缩后也得在options中加入 sourcemap: true, 否则只需要设置 devtool 就OK