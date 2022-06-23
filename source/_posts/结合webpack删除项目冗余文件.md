---
title: 结合webpack删除项目冗余文件
date: 2022-3-25 20:00:00
tags:
  - js
---

## 背景

项目中有许多冗余文件，非常影响项目大小，因此需要删除不需要的文件

## 思考

手动删除非常麻烦并且无法保证清除效果，因此想办法找出某种规律批量删除。

前端项目一般使用 webpack 打包，打包的过程中 webpack 会根据入口文件遍历出所有文件，因此可以利用 webpack 来确定项目中哪些文件可用。而项目中的所有文件可以通过 nodejs 得到，因此可以通过 webpack 和 nodejs 做文件对比，然后自动清除未被编译引用的文件，思路如下：

1.在项目 webpack 编译时，会从入口处自动遍历需要的文件，并且在编译过程中，会形成一个 stats.json 文件，用来展示这种引用关系。

2.我们可以从 stats.json 中得到所有编译需要的文件的原始路径

3.我们可以得到项目中所有的文件的原始路径

4.我们可以通过对比两边的路径，从而得到未被编译的文件的路径

5.我们可以通过 shelljs 包，通过命令行删除该未被引用的路径文件

## 实现

#### 获取编译文件

在 webpack 执行时，我们可以在 webpack 命令后加上`--profile --json > stats.json`，这样就会最终的输出目录，额外输出一个`stats.json`文件，里面记录了 webpack 执行的一些 log 信息，或者执行过程等信息。比如：

```json
"ana": "webpack --config webpack.prod.js --profile --json > stats.json"
```

通过`npm run ana`命令执行后，即可在 webpack 命令执行输出目录额外输出`stats.json`文件。

在 vue cli 中，因为其内部集成了`webpack-bundle-analyzer`分析工具，所以我们可以使用以下命令获得`stats.json`文件

```json
"build:ana": "vue-cli-service build '--report-json'",
```

获得`stats.json`文件后，通过 fs 模块读取内容。如果文件过大（超过 512M）那么需要使用文件流读取内容：

```js
let chunkList = [];
let list = [];
let stream = fs.createReadStream(statPath, {
  encoding: "utf8",
});
const parser = JSONStream.parse("modules.*");
stream.pipe(parser);
parser.on("data", (data) => {
  chunkList = chunkList.concat(data);
});
parser.end("data", (data) => {
  chunkList = chunkList.map((item) => {
    return {
      name: item.name,
      modules: item.modules,
      id: item.id,
    };
  });
  // todo
});
```

#### 获取全部文件

我们通过 glob 模块，获得项目指定目录下的所有文件。

```js
function getAllFilesInSrc() {
  const pattern = "./src/**";

  return new Promise((resolve, reject) => {
    glob(
      pattern,
      {
        nodir: true,
      },
      (err, files) => {
        resolve(files);
      }
    );
  });
}
```

#### 获取待删除文件

我们直接通过对比所有文件和编译文件，来获得待删除文件（未被编译的文件）

```js
// allFile 所有文件数组
// compileFile 被编译使用的文件数组
// unFiles 需要被删除的文件数组
let unFiles = allFile.filter((item) => compileFile.indexOf(item) < 0);
```

#### 删除文件

得到待删除文件数组后，我们遍历删除文件即可：

```js
unFiles.forEach((file) => {
  console.log("已删除文件：" + file);
  shelljs.rm(file);
});
console.log("合计删除文件个数：" + unFiles.length);
```

## 遇到的问题

1.dashboard 通过设计态和运行态分两次打包，因此需要分析两份 stats.json（解决）

2.设计态生成的 stats.json 过于巨大（590M)，超过了 nodejs 能默认处理的数据上限（512M）：
采用 JSONStream 模块，通过创建文件流，使用 JSONStream 解析文件流，解决该问题

3.文件解析不准确问题
例如'./src/components/charts/yn-echart-bar-dt/index.js'文件用到了，但是没有被统计到编译文件中

该文件结构类似于：

```js
{
  "modules": [
    {
    ...,
    name: "xxx",
    ...,
    "modules": [
      {
      ...,
      name: "xxx",
      ...,
      }
    ]}
  ]
}
```

我们需要读取 modules 数组，如果下层对象中存在 modules 属性，说明该对象是聚合来的，需要把当前 modules 属性迭代出来，加入到编译文件数组中（这里需要递归迭代），这样我们就能拿到全量的编译文件了。

4.修复误删文件，后来直接添加了忽略列表，用来存放需要忽略删除的文件

## 参考文档

[利用 webpack 分析和删除冗余文件](https://github.com/ginobilee/blog/issues/10)
[包含统计数据的文件(stats data)](https://www.webpackjs.com/api/stats/#asset-objects)
