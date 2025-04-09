---
title: electron基础
date: 2025-03-14 10:00:00
tags:
  - electron
---

# 集成准备

在项目根目录创建 main.js 文件，并且在 package.json 文件中指定入口：

```js
  "main": "main.js" // 在node环境中运行项目时，此配置项为默认入口
```

在一般的前端项目中，比如 vite 创建的 vue3 应用，是默认没有 main 配置项的，因为前端应用项目通常不需要指定入口文件（nodejs 模块一般需要）。为了集成 electron，因此需要指定入口为 electron 主进程文件 main.js。

vite 创建的 vue3 项目为例，package.json 中没有 main 配置项，它的默认入口是 index.html，通过 script 标签引入 vue 的 入口文件，类似如下：

```js
  <div id="app"></div>
  <script type="module" src="/src/main.ts"></script>
```

# electron 创建

在主进程文件 main.js 中，首先引入两个模块，他们分别控制 electron 事件生命周期和窗口创建与管理

```js
const { app, BrowserWindow } = require("electron");
```

- app, which controls your application's event lifecycle.
- BrowserWindow, which creates and manages app windows.

### 创建窗口

```js
const { app, BrowserWindow } = require("electron");

const createWindow = () => {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
  });
  win.loadFile("index.html");
};

app.whenReady().then(() => {
  createWindow();
});
```

因为 electron 许多核心模块都是 node.js 的事件触发器，比如 app 和 BrowserWindow 就是 EventEmitter 的实例，因此遵循 Node.js 的异步事件驱动架构

In Electron, BrowserWindows can only be created after the app module's ready event is fired.
You can wait for this event by using the app.whenReady() API and calling createWindow() once its promise is fulfilled.

node 事件触发器示例代码：

```js
const EventEmitter = require("events");
const emitter = new EventEmitter();

// 监听事件
emitter.on("myEvent", () => {
  console.log("事件被触发了！");
});

// 触发事件
emitter.emit("myEvent"); // 输出: "事件被触发了！"
```

### 窗口生命周期

#### 窗口关闭时

electron 支持 3 个平台，Windows（win32）、Linux(linux)、macOS(darwin)，可以通过 nodejs 中 `process.platform` 变量来判断。在 win 和 linux 中，关闭窗口则退出程序，而在 macOS 中则不是。

```js
app.on("window-all-closed", () => {
  if (process.platform !== "darwin") {
    app.quit();
  }
});
```

#### 没有窗口时打开一个窗口

```js
app.whenReady().then(() => {
  createWin();

  app.on("activate", () => {
    if (BrowserWindow.getAllWindows().length === 0) {
      createWin();
    }
  });
});
```

### 调试

在根目录新建一个 .vscode 文件夹，然后在其中新建一个 launch.json 配置文件并填写如下内容：

```js
{
  "version": "0.2.0",
  "compounds": [
    {
      "name": "Main + renderer",
      "configurations": ["Main", "Renderer"],
      "stopAll": true
    }
  ],
  "configurations": [
    {
      "name": "Renderer",
      "port": 9222,
      "request": "attach",
      "type": "chrome",
      "webRoot": "${workspaceFolder}"
    },
    {
      "name": "Main",
      "type": "node",
      "request": "launch",
      "cwd": "${workspaceFolder}",
      "runtimeExecutable": "${workspaceFolder}/node_modules/.bin/electron",
      "windows": {
        "runtimeExecutable": "${workspaceFolder}/node_modules/.bin/electron.cmd"
      },
      "args": [".", "--remote-debugging-port=9222"],
      "outputCapture": "std",
      "console": "integratedTerminal"
    }
  ]
}
```

启动调试时，既可以调试主进程（main.js），也能调试渲染进程（比如 vue 页面 src/main.ts）。
为什么主进程仅仅通过`win.loadURL('http://localhost:5173')`即可调试渲染进程呢？
因为 Chromium 支持远程调试，VSCode 使用内置的 Chrome 调试器扩展，通过 CDP 协议（Chromium 提供的一套基于 JSON 的协议，允许外部工具控制浏览器行为（如设置断点、检查 DOM、监控网络请求等））与渲染进程通信

# 预处理脚本
