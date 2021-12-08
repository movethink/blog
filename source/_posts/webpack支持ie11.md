---
title: webpack支持ie11
date: 2021-12-03 20:00:00
tags:
  - js
---

## 准备

对于某些项目来说，是有 ie11 支持需求的，一般 to b 的业务，得优先考虑是否需要支持 ie。对于 vue 项目而言，一般分为 webpack 集成 vue，和使用 vue cli 搭建。个人倾向于使用 webpack 集成 vue 的方式，这样遇到问题更容易有解决方案。vue cli 在 webpack 上封装了一层，使用起来还是有点束缚，当然，一般的支持 vue cli 还是支持度很高的。如果是高度自定义化的项目，个人喜欢用 webpack 集成 vue。

## 编译

首先需要借助 babel，来把 es6 代码编译为 es5，通常会借助 babel-loader 实现，比如在项目中引入`babel.config.json`，通常格式如下：

```json
{
  "presets": [
    [
      "@babel/preset-env"
      // 设置plugins后可以不写这个配置项，免得重复打包增加包大小
      // 这是core-js3用法，因为p1包的限制，项目整体只能使用corejs2
      // {
      //     "useBuiltIns": "entry",
      //     "corejs": {
      //         "version": 3
      //     },
      //     "targets": {
      //         "ie": "11"
      //     }
      // }
    ]
  ],
  "plugins": [
    // corejs3用法
    // [
    //     "@babel/plugin-transform-runtime",
    //     {
    //         "corejs": {
    //             "version": 3,
    //             "proposals": true
    //         },
    //         "useESModules": true
    //     }
    // ]
  ]
}
```

配置好 babel 配置文件后，我们需要在 webpack.config.js 文件中运用 loader 规则：

```js
module: {
    rules: [
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.vue$/,
        use: [{ loader: "vue-loader" }],
      },
      {
        test: /\.svg/,
        use: ["file-loader"],
      },
      {
        test: /\.html$/,
        use: ["html-loader"],
      },
      {
        test: /\.m?js$/,
        exclude: /(node_modules|bower_components)/, //排除node_modules中的文件
        use: {
          loader: "babel-loader",
        },
      },
    ],
  },
```

这样项目中的 js 文件就会从 es6 编译为 es5 代码。

有些库默认不支持 es5，他们编译在 node_modules 中的代码就是 es6 的语法，比如使用了一些箭头函数之类的语法，这时候需要单独在 webpack 中为其配置转换规则：

```js
module: {
    rules: [
      {
        test: /\.m?js$/,
        exclude: /(node_modules|bower_components)/, //排除node_modules中的文件
        include: [
          resolve("../node_modules/react-loadable"),
          resolve("../src"),
          resolve("./router.config.js")
        ],
        use: {
          loader: "babel-loader",
        },
      },
    ],
  },
```

需要注意的是，以上写法仅仅支持 babel7 以上，并且 babel 配置文件需要是`babel.config.json`，而不能是简写的`.babelrc`形式，因为`.babelrc`作为配置文件的时候默认不会编译`node_modules`文件夹中的代码。

## polyfill

babel-loader 只能把 es6 代码编译为 es5 代码，但是还有一些 es6 中的新 api，是无法转换的，需要提供合适的“垫片”----polyfill，简单来说，就是把新的 api 用老的方式实现以下，因为低级浏览器中无法识别这些新 api，比如 promise 函数在 ie11 中就无法支持，需要有合适的 polyfill 才能识别。

### webpack + vue

我们一般使用 vue 来加速前端代码构建，这里例举如何在 webpack 中直接集成 vue 项目，并使得项目支持 ie11.

关于 webpack 如何集成 vue，可以查看之前的文章，在此不再赘述。

#### core-js2

core-js2 方式中，一般需要安装`@babel/polyfill`包，可以直接通过 npm 方式安装。因为 polyfill 会在源代码安装前运行，所以需要安装成 dependencies 而不是 devDependencies，安装`@babel/polyfill`包后，把该包引入 main.js 文件（入口文件）即，除此以外，还需要在 webpack 的入口文件处加上如下配置：

```js
config.entry("main").add("@babel/polyfill");
```

其实很多情况下，只要在 main.js 下首行引入 polyfill 即可

```js
import "@babel/polyfill";
```

但是这种方式会全量引入 polyfill，对于不好排查新 api，并且比较大的项目，可以使用这种方式，大而全。

#### core-js3

core-js3 会支持很多 2 中没有的 api，并且配置相对来说更加方便，只需要在 babel.config.js 中这样配置就行：

```js
{
    "presets": [
        [
            "@babel/preset-env",
            // 设置plugins后可以不写这个配置项，免得重复打包增加包大小
            {
                "useBuiltIns": "entry",
                "corejs": {
                    "version": 3
                },
                "targets": {
                    "ie": "11"
                }
            }
        ]
    ],
    "plugins": [
        // corejs3用法
        [
            "@babel/plugin-transform-runtime",
            {
                "corejs": {
                    "version": 3,
                    "proposals": true
                },
                "useESModules": true
            }
        ]
    ]
}
```

并且可以实现 polyfill 的按需加载。

### vue cli

为什么会有 core-js2 和 core-js3 的方式呢？是因为某些项目中有些包限制，之前的版本可能是基于 core-js2 构建，所有这种情况下只能选择 core-js2。

当使用 vue cli 构建时，cli3 基于 core-js2;cli3 以上基于 core-js3,因此不同的 vue cli 版本也会有不同的处理。一般来说，在 babel.config.js 中，会这样编写：

```js
module.exports = {
  presets: [
    // [
    //   "@babel/preset-env",
    //   // 设置plugins后可以不写这个配置项，免得重复打包增加包大小
    //   // webpack直接集成vue方式
    //   {
    //     useBuiltIns: "entry"
    //   }
    // ],
    // [
    //   "@babel/preset-react",
    //   {
    //     throwIfNamespace: false
    //   }
    // ],
    // vue cli方式
    [
      "@vue/app",
      {
        // polyfills: [
        //   "es6.promise",
        //   "es6.symbol",
        //   "es6.string.includes",
        //   "es7.array.includes",
        //   "es6.string.repeat"
        // ]
        useBuiltIns: "entry",
      },
    ],
  ],
  plugins: [
    // ["@babel/plugin-proposal-class-properties"],
    [
      "import",
      { libraryName: "ant-design-vue", libraryDirectory: "es", style: "css" },
    ],
    [
      "import",
      {
        libraryName: "vant",
        libraryDirectory: "es",
        style: true,
      },
      "vant",
    ],
  ],
};
```

如果是 cli3 以上的版本，则可以通过如下配置来实现：

```js
module.exports = {
  presets: [
    [
      "@vue/app",
      {
        polyfills: ["es.promise", "es.symbol"],
      },
    ],
  ],
};
```

在上面两种方式中，均可以在 package.json 中定义目标浏览器`browserslist`，该值会被`@babel/preset-env`读取到，用来确定需要转译的 js 特性，比如在 package.json 中我们可以这样定义：

```json
"browserslist": [
    "> 1%",
    "last 2 versions"
  ],
```

在 vue cli 中，默认情况下 babel-loader 会忽略所有 node_modules 中的文件。如果你想要通过 Babel 显式转译一个依赖，可以在`transpileDependencies`选项中列出来,比如`@antv/g6`默认支持 es6，如果我们项目中有依赖该库，那就只能在 vue.config.js 文件的`transpileDependencies`字段中指定：

```js
module.exports = {
  transpileDependencies: ["@antv/g6"],
};
```

当没有使用 vue cli 时，我们可以像上文中所诉,在 webpack.config.js 中通过`include`字段定义：

```js
module: {
    rules: [
      {
        test: /\.m?js$/,
        exclude: /(node_modules|bower_components)/, //排除node_modules中的文件
        include: [
          resolve("../node_modules/react-loadable"),
          resolve("../src"),
          resolve("./router.config.js")
        ],
        use: {
          loader: "babel-loader",
        },
      },
    ],
  },
```
