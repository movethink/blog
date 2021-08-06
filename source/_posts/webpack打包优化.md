---
title: webpack打包优化
date: 2021-08-06 14:00:00
tags:
  - js
  - webpack
---

## 主要原因
在项目中，我们发现打包出来的源码比较庞大，并且引用关系十分混乱，具体表现在：
 + 引用多个版本的包
 + 项目中存在很多冗余文件
 + 项目中引用了无用的包
 + 多打包入口导致包重复引用

## 分析辅助工具
有一个npm工具包，叫做`webpack-bundle-analyzer`，安装后可以启动一个打包分析界面。
具体在webpack.config.js中配置：
```js
const BundleAnalyzerPlugin = require("webpack-bundle-analyzer")
  .BundleAnalyzerPlugin;
module.exports = {
  // devtool: "source-map",
  plugins: [
    // 打包构成分析器
    new BundleAnalyzerPlugin(),
    // 默认配置的具体配置项
    // new BundleAnalyzerPlugin({
    //   analyzerMode: 'server',
    //   analyzerHost: '127.0.0.1',
    //   analyzerPort: '8888',
    //   reportFilename: 'report.html',
    //   defaultSizes: 'parsed',
    //   openAnalyzer: true,
    //   generateStatsFile: false,
    //   statsFilename: 'stats.json',
    //   statsOptions: null,
    //   excludeAssets: null,
    //   logLevel: info
    // })
  ],
};
```
执行npm命令时，会自动打开起分析页面，界面类似：
![](http://qiniu.js-5.com/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20210806142838.png)

## 多入口打包交叉引用问题
比如有这样几个文件，均在`./src/bundle-test`目录下：
a.js
```js
const a = () => {
  console.log("isA");
  console.log(c());
};
import c from "./c";
const vue = require("vue");

export default a;
```
b.js
```js
const b = () => {
  console.log("isB");
  console.log(c());
};

import c from "./c";
const vue = require("vue");

export default b;
```
c.js
```js
const c = () => {
  console.log("isC");
};

export default c;
```
index.js
```js
import A from "./a.js";
import B from "./b.js";

const test = () => {
  console.log("isI");
  A();
  B();
};
test();
```
分别按入口打包：
```js
entry: {
    index: "./src/bundle-test/index.js",
    a: "./src/bundle-test/a.js",
  }, // 入口文件
  output: {
    filename: "[name].js", // 出口文件名称
    libraryTarget: "umd", //在 AMD 或 CommonJS require 之后可访问
    path: path.resolve(__dirname, "lib"), // 定义出口文件夹目录
  },
```
优化前：
![](http://qiniu.js-5.com/%E4%BC%98%E5%8C%96%E5%89%8D.png)

我们使用webpack的splitChunks规则，使得公共模块抽离出来
在webpack.config.js下配置：
```js
optimization: {
    minimize: false,
    // 抽离公共文件
    splitChunks: {
      minSize: 30, // 打包的最小大小
      cacheGroups: {
        default: {
          name: "common",
          chunks: "initial",
          minChunks: 2, //模块被引用2次以上的才抽离
          priority: 1, // 越大越优先
        },
      },
    },
  },
```
优化后：
![](http://qiniu.js-5.com/%E4%BC%98%E5%8C%96%E5%90%8E.png)

可以看到，优化后打包大小几乎减小了一半。