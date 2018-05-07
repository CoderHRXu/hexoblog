
---
title: 如何使用Koa搭建静态资源文件服务器
---



## 1.node.js
Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境，执行Javascript的速度非常快，性能非常好。 
Node.js 使用了一个事件驱动、非阻塞式 I/O 的模型，使其轻量又高效。 
Node.js 的包管理器 npm，是全球最大的开源库生态系统。

## 2.Koa
[Koa](http://koajs.com/) 就是一种简单好用的 Web 框架。它的特点是优雅、简洁、表达力强、自由度高。本身代码只有1000多行，所有功能都通过插件实现，很符合 Unix 哲学。

本文讲述了如何最简单的使用Koa框架来实现静态资源发布服务器。

## 3.准备工作
Koa 必须使用7.6以上版本的，如果你的版本低于这个要求，就要先升级Node，操作方法不多说了。

## 4.正式开搞
### 4.1 快速开启http服务
在你的电脑上指定一个目录，打开终端cd到目录，npm install 安装依赖包，新建一个app.js 文件，作为入口。在js文件中写如下代码

```
const Koa = require('koa');
const app = new Koa();

app.listen(3000);
```
终端里输出 node app.js，一个服务就这么运行了。是不是so easy！
打开浏览器访问http://loacalhost:3000，你会看到页面显示"Not Found"，表示没有发现任何内容。这是因为我们并没有告诉 Koa 应该显示什么内容。



### 4.2 配置路由和静态资源
网站一般都有多个页面，多个页面就需要路径的路由。原生路由用起来不太方便，我们可以使用封装好的[`koa-route`](https://www.npmjs.com/package/koa-route)模块。
如果网站提供静态资源（图片、字体、样式表、脚本......），为它们一个个写路由就很麻烦，也没必要。[`koa-static`](https://www.npmjs.com/package/koa-static)模块封装了这部分的请求。

实际开发中，一个服务不仅能提供静态资源访问，也会提供一些接口给外界调用，为了统一管理这部分静态资源，一般都放在根目录public文件夹下。

其他的接口，就需要用路由统一分配路径

```
// 0.导入需要的资源包
const Koa = require('koa');
const app = new Koa();
const route = require('koa-route');
const serve = require('koa-static');

// 1.主页静态网页 把静态页统一放到public中管理
const home   = serve(path.join(__dirname)+'/public/');
// 2.hello接口
const hello = ctx => {
  ctx.response.body = 'Hello World';
};

// 3.分配路由
app.use(home); 
app.use(route.get('/', hello));
app.listen(3000);
```
这里用到了一个node.js的全局变量: __dirname,表示当前执行脚本所在的目录。然后把你之前写好的静态资源，比如html，css等文件，统统放到根目录的public文件夹下。运行一下app.js。在浏览器里面输如 [http://loacalhost:3000](http://loacalhost:3000/) 默认获取加载public目录下的index.html文件，如果没有，那就在浏览器端口号后面拼接路径。比如 [http://loacalhost:3000/index2.html](http://loacalhost:3000/index2.html)。

到此为止就可以访问public下所有的静态资源了。
鼓掌，散花！
