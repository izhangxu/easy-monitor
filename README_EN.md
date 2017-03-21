<div align="center">
  <img width="300" heigth="300" src="https://github.com/hyj1991/assets/blob/master/easy-monitor/logo.png" alt="easy-monitor logo">
</div>

[![npm version](https://badge.fury.io/js/easy-monitor.svg)](https://badge.fury.io/js/easy-monitor)
[![Package Quality](http://npm.packagequality.com/shield/easy-monitor.svg)](http://packagequality.com/#?package=easy-monitor)
[![npm](https://img.shields.io/npm/dt/easy-monitor.svg)](https://www.npmjs.com/package/easy-monitor)
[![license](https://img.shields.io/github/license/mashape/apistatus.svg)](LICENSE)

# Easy-Monitor
[中文版](README.md)

## Introduction

A lightweight Node.js performance monitoring tool, you just need to ```require``` it at the entrance of your project, then ```Easy-Monitor``` will show you your Node.js process details.

### I.Functions

* **Find 5 or more functions that take the longest execution time**
* **Find functions that perform more than expected**
* **Find the functions which V8 engine can not optimize**

The number of listings disaplayed above can be configured directly in URL.
You can also set your own rule that you can fiter **functions** and **filePaths** so that the results is what you really want.

The purpose of ```Easy-Monitor``` is to help you more in-depth understanding of our Node process, performance optimization can be more targeted, and ultimately enhance our project experience.


### II.Feature

* **LightWeight**：```Easy-Monitor``` is unconventional C/S physical separation model，all you need to to is ```require``` in your project，there is no additional server/agent deployment costs.

* **RunTime**：```Easy-Monitor``` can provide you runtime performance and memory details, and it can be used for production.

* **Stateless**：```Easy-Monitor``` always show you your node process status when you visit it.

## Compatibility
* Node v6.x
* Node v5.x
* Node v4.x

Temporary don't support Node v7.x.

## Quick Start

```Easy-Monitor``` is simple，only three steps you can start your own performance monitor.

### I.Install

Simply run as：

```bash
npm install easy-monitor
```

### II.Require

Require in your project，argument is your project name, such as:

```js
const easyMonitor = require('easy-monitor');
easyMonitor('your project name');
```

### III.Visit

Open browser, input as below, then you will see detail:

```bash
http://127.0.0.1:12333
```

### IV.A thorough example with Express

```js
'use strict';
const easyMonitor = require('easy-monitor');
easyMonitor('your project name');
const express = require('express');
const app = express();

app.get('/hello', function (req, res, next) {
    res.send('hello');
});

app.listen(8082);
```

Save the above as a JS file, then open your browser and visit ```http://127.0.0.1:12333```, just so simple!

## Customization

### I.Customization Parameters

```Easy-Monitor``` also remain some important
attributes so that we can customize conveniently, it's dependence on the object you set  when executing ```require('easy-monitor')(object)```, this object  can have below attributes as:

* **logLevel**：Number，default is 2，it used to set log level：
	* 0：don't output any log
	* 1：output error log
	* 2：output info log
	* 3：output debug log

* **appName**：String, default is getting from process.title, it used to set your project name.

* **httpServerPort**：Numver, default is 12333, it used to set monitor http server port.

* **filterFunction**：function，default filtering out the profiling results of the 'node_modules' and 'anonymous', and all file paths that don't have the string '/', because of these may be system functions.
Developer can write function to filter your own results, below is the params and returned value:
	* filePath: String, file path that functions at
	* funcName: String, function name
	* returned value: if true remain, if false filtering out

* **monitorAuth**：function，default don't authentication，it used for authentication, developer can write function for own authentication, below is the params and returned value:
	* user：String, it's username
	* pass：String, it's password
	* returned value：a Promise instance，resolve(true) mean authentication pass, resolve(false) or reject mean authentication failed.

### II.Customization Example

Below is a thorough example that ```Easy-Monitor``` with Express：

```js
'use strict';
const easyConfig = {
    logLevel: 3,
    appName: 'My Project 1',
    httpServerPort: 8888,
    filterFunction: function (filePath, funcName) {
        if (funcName === 'anonymous' || ~filePath.indexOf('node_modules')) {
            return false;
        }
        return Boolean(/^\(\/.*/.test(filePath));
    },
    monitorAuth: function (user, pass) {
        return new Promise(resolve => resolve(Boolean(user === 'admin' && pass === 'lifeishard')));
    }
};
const easyMonitor = require('easy-monitor');
easyMonitor(easyConfig);

const express = require('express');
const app = express();

app.get('/hello', function helloIndex(req, res, next) {
    let date = Date.now();
    while (Date.now() - date < 300) {
    }
    res.send('hello');
});

app.listen(8082);
```

这个例子中，日志级别被调整为3，监控服务器端口更改为8888，也设置了过滤规则和简单的鉴权规则。
并且 ```/hello``` 这个路由被设置成阻塞300ms后返回，大家可以打开 ```http://127.0.0.1:8888``` 进入 ```Easy-Monitor``` 首页，点击项目名称或者pid进行profiling操作，同时不停访问 ```http://127.0.0.1:8082/hello``` 这个路径，然后观察结果来自行尝试一番。


## 监控页面一览

### I.首页

#### 1.查看整个项目

<img width="550" heigth="300" src="https://github.com/hyj1991/assets/blob/master/easy-monitor/Index_Project.jpeg" alt="Index_Project">

如图，点击项目名称，则会对 **整个项目** 所有的进程进行profiling操作，这个所有进程包含：

* 单进程模式下则只有一个主进程
* cluster模式下所有的子进程

#### 2.查看项目下某一个子进程

<img width="550" heigth="300" src="https://github.com/hyj1991/assets/blob/master/easy-monitor/Index_Pid.jpeg" alt="Index_Pid">

如图，在cluster模式下项目会有多个子进程，点击某一个特定的pid，则只会对 **此pid对应的子进程** 进行profiling操作。

#### 3.多项目部署

<img width="550" heigth="300" src="https://github.com/hyj1991/assets/blob/master/easy-monitor/Index_Multi.jpeg" alt="Index_Multi">

如图，```Easy-Monitor``` **支持多项目部署**，用法和单项目是一模一样的，可以参考前面的快速开始。那么多项目启动后，监控页面会展示出不同的项目名称和对应的子进程pid。

### II.监控详情页

#### 1.执行时间超出预期的函数列表

<img width="550" heigth="300" src="https://github.com/hyj1991/assets/blob/master/easy-monitor/Detail_Long.jpeg" alt="Detail_Long">

如图，可以追加 ```querystring``` 参数的形式自定义预期时间以及展示的条数，如下：

* ```?timeout=你预期的时间(ms)```
* ```?long_limit=你想展示的条数```
* ```?timeout=你预期的时间(ms)&long_limit=你想展示的条数```

#### 2.耗费时间最久的函数列表

<img width="550" heigth="300" src="https://github.com/hyj1991/assets/blob/master/easy-monitor/Detail_Top.jpeg" alt="Detail_Top">

如图，可以追加 ```querystring``` 参数的形式自定义展示条数，如下：

* ```?top_limit=你想展示的条数```

#### 3.v8引擎无法优化的函数列表

<img width="550" heigth="300" src="https://github.com/hyj1991/assets/blob/master/easy-monitor/Detail_Bail.jpeg" alt="Detail_Bail">

如图，可以追加 ```querystring``` 参数的形式自定义展示条数，如下：

* ```?bail_limit=你想展示的条数```

## 测试

clone下本代码后，使用npm安装依赖，然后执行如下测试脚本：

```
npm run test
```

即可看到覆盖率测试报告。

# License

[MIT License](LICENSE)

Copyright (c) 2017 hyj1991