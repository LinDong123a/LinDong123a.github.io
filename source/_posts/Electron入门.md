---
title: Electron入门
date: 2018-01-16 16:59:25
tags: electron
---

> 最近在Linux捣鼓着怎么用微信时，看到很多解决方案都是用electron做的壳，就很好奇其实现原理，就去官网了解了下文档，觉得这是个很有意思的东西。这里就把一些了解把很浅层的理解记录一下，等把锅都忙完再看能不能用这个做点东西吧。
>
> [官方文档入口](https://electronjs.org/docs)

<!-- more -->

### 简介

感觉这可以用官方的一句话来说明：”使用Javascript, HTML和CSS构建跨平台的桌面应用。“

就个人目前的理解而言，我觉得它的理念有点像虚拟机。把应用程序的开发环境变成与网页构建类似的平台（其实它本身就是基于node.js和chromium的）。在这种环境下，每个桌面窗口显示的内容可由html和css和定制，人与窗口的交互响应由javascript来实现。

### 快速入门

> 这一段其实就是官方文档的翻版，我只是把它复述一遍来加深印象而已

#### 环境搭建

在简介中我们提到，这个框架是基于nodejs和chromium的，因此要想正常使用electron，需要先安装npm和nodejs

> 考虑到目前国内的情况，npm很多module要么不能下载，要么下载速度不怎样，因此推荐在下载了npm之后，通过`npm install -g cnpm`后，使用cnpm来下载相应的模块（cnpm是淘宝做的镜像）

之后运行`cnpm install -g electron`就可以了。

#### 程序构建

现在假设你的应用程序代码都放在app文件夹下，那么你的app文件夹至少需要有以下结构：

> \- app
> |_ main.js
> |_ package.json
> |_ index.html

上述的结构其实跟我们写网页是十分相似的。

package.json: 主要是用来记录程序的一些相关信息，如作者，版本号，以及入口文件等。示例如下：

```json
{
  'name': 'app', //your app name
  'version': '0.1.0', //the version of your app
  'main': 'main.js' //the entry file of your app, default: index.js
}
```

main.js: 入口文件，有些类似于C/C++的主函数，而且所有与GUI有关的操作都应当由该文件来完成，其它分支是		   不能直接对系统 GUI操作的，因为这样很容易造成内存的泄露。示例如下：

```javascript
'use strict';

// app用来进行整个程序的生命周期控制， BrowserWindow来产生窗口
const {app, BrowserWindow} = require('electron');

let mainWindow = null;

app.on('ready', () => {
  //创建一个新窗口
  mainWindow = new BrowserWindow({
    height: 800,
    width: 600
  });
  
  //这里使用index.html文件来对文件窗口进行渲染
  mainWindow.load("file://" + __dirname + "/index.html");
  
  mainWindow.on('closed', () => {
    mainWindow = null;
  });
});

app.on('window-all-closed', () => {
  app.quit();
});

```

index.html: 这其实就是一个普通的html文件而已，在写的时候当做一个网页的构架就行了。示例如下:

```html
<!DOCTYPE html>
<!-- 简单地显示 Hello world! -->
<html>
  <header>
  	<meta charset="utf-8" />
  </header>
  
  <body>
    <h1>
       Hello world!
    </h1>
  </body>
</html>
```

当文件写完后，在当前文件夹下，运行`electron .`即可看到这样的效果：

![electron_output](./img/electron_output.png)

#### 应用分发

electron应用的分发其实也很简单。在[该网页](https://github.com/electron/electron/releases)下下载对应平台的文件，把你的app代码放在resource目录下的app文件夹内(app文件夹可能得自己创建)，然后直接点击对应的electron.exe(windows)/electron(linux)就可以了。

当然，如果为了减小体积或者保护源码，可以先将代码打包成asar档案文件。