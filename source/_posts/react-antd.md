---
title: create-react-app 使用 antd，按需加载
date: 2018-12-04 23:10:06
tags: create-react-app antd 按需加载
---

# create-react-app 使用 antd，按需加载

#### 前言
使用 [create-react-app](https://github.com/facebook/create-react-app) 创建的 react 项目，结合使用 [antd](https://ant.design/index-cn) 时，官网文档介绍了[按需加载](https://ant.design/docs/react/getting-started-cn#%E6%8C%89%E9%9C%80%E5%8A%A0%E8%BD%BD)需要使用 [babel-plugin-import](https://github.com/ant-design/babel-plugin-import) 来配合，如果你的项目还没有执行 `npm run eject`，则使用[官网推荐的配置](https://ant.design/docs/react/use-with-create-react-app-cn#%E9%AB%98%E7%BA%A7%E9%85%8D%E7%BD%AE)即可，由于我已经 `eject` 过，记录下我的配置。



#### 过程
##### 1、安装 `yarn add babel-plugin-import` 或者 `npm -s install babel-plugin-import`



##### 2、给 webpack 的 `babel-loader` plugins 加上 `babel-plugin-import`

需要修改两个文件 `/config/webpack.config.prod.js` 和 `/config/webpack.config.dev.js` （修改的内容一样），找到加载 `babel-loader` 的地方，往他的 plugins 加入如下代码

```javascript
  [
    require.resolve('babel-plugin-import'),// 导入 import 插件
    {
      libraryName: 'antd',   //暴露antd
      style: 'css'
    }
  ]
```

最后形成的配置如下（create-react-app 版本不同最后的配置可能不一样，原理一样）：

```javascript
  // Process application JS with Babel.
  // The preset includes JSX, Flow, and some ESnext features.
  {
    test: /\.(js|mjs|jsx|ts|tsx)$/,
    include: paths.appSrc,
    loader: require.resolve('babel-loader'),
    options: {
      customize: require.resolve(
        'babel-preset-react-app/webpack-overrides'
      ),

      plugins: [
        [
          require.resolve('babel-plugin-named-asset-import'),
          {
            loaderMap: {
              svg: {
                ReactComponent: '@svgr/webpack?-prettier,-svgo![path]',
              },
            },
          },
        ],
        [
          require.resolve('babel-plugin-import'),// 导入 import 插件
          {
            libraryName: 'antd',   //暴露antd
            style: 'css'
          }
        ],
      ],
      // This is a feature of `babel-loader` for webpack (not Babel itself).
      // It enables caching results in ./node_modules/.cache/babel-loader/
      // directory for faster rebuilds.
      cacheDirectory: true,
      // Don't waste time on Gzipping the cache
      cacheCompression: false,
    },
  }
```






