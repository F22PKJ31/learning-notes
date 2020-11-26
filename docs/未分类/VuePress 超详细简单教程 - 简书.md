> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/7a2cc8a7f40c)

1）VuePress 是什么？
---------------

先让我们看看 VuePress 能干什么？有什么效果？

> ![](http://upload-images.jianshu.io/upload_images/6825620-28e35cce30aa26a7.png) image.png

很像 vue 官网的文档页面，因为 vuePress 就是尤大大的一个力作  
[vuePress 官网介绍](https://links.jianshu.com/go?to=http%3A%2F%2Fcaibaojian.com%2Fvuepress%2Fguide%2F)介绍的很详细，这里只做搭建 VuePress 项目教程

2）安装
----

> 注意 VuePress 基于 node 8.0+

```
npm install -g vuepress
```

> 创建工作文件夹 study  
> 进入文件夹  
> 快速初始化 package.json

```
npm init -y
```

### 2.1 新建文件夹 docs

配置 package.json，添加下述兩行

```
{
  "scripts": {
    "docs:dev": "vuepress dev docs",
    "docs:build": "vuepress build docs"
  }
}
```

进入 docs 文件夹 创建 README.md 文件

此时运行命令

```
npm run  docs:dev
```

此时文件夹结构

```
study
+--docs
+----README.md
+--package.json
```

运行访问 [http://localhost:8080/](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8080%2F)

> ![](http://upload-images.jianshu.io/upload_images/6825620-8d12ecc68038f75d.png) image.png

结束运行 ，执行命令

```
npm run docs:build
```

然后看文件变化 多了个 node_modules  
docs 多了个 .vuepress 文件夹

```
study
+--docs
+----.vuepress
+------ dist   //打包后的文件夹
+----README.md
+--package.json
+--node_modules
```

我们在. vuepress 创建  
config.js 文件  
添加

```
module.exports = {
  title: 'Adroi媒体API 接口文档',  // 设置网站标题
  description : 'Adroi',
  base : '/v1/adroi-h5/adroiapi/',
  themeConfig : {
    nav : [
        { text: '接口定义', link: '/apiword' },
        { text: '接口字段定义', link: '/api' },
        { text: '附录：错误码', link: '/error' }
    ],
    sidebar: {
        '/' : [
      "/", //指的是根目录的md文件 也就是 README.md 里面的内容
            "apiword",  根目录创建 apiword.md文件
            "api",根目录创建 api.md文件
            "error" 根目录创建 error.md文件
        ]
    },
    sidebarDepth : 2
  }
}
```

> ![](http://upload-images.jianshu.io/upload_images/6825620-3b3c28127120e28e.png) image.png

3）添加静态图片
--------

```
![An image](./baner.png)
```

4) 添加指定样式
---------

> 添加样式 分两种 一种不用预编译处理、一种使用

```
//不使用预编译处理
//直接在md文件底部添上
<style></style>
```

```
//使用预编译处理
首先需要安装所需的模块  举例：stylus
npm i stylus stylus-loader -D
<style lang="stylus"></style>
```

5)vuePress 可添加 js 代码
--------------------

> 既然是尤大大的作品 那肯定是支持 vue 语法的  
> 在 MD 文件中直接写入 js 的语法

```
<script>
  export default{
    //...do something
}
</script>
```

6) 代码发布或上传至服务器
--------------

> 关于发布问题：首先我们知道在打包后的文件都是静态文件之前的 MD 文件都被打包成 html 静态文件了, 其次在文件 config.js 中 base 至关重要

#### 6-1 发布在云盘中如百度云盘 github 上可直接通过链接访问

在云盘上创建根目录如 vuepress，然后在 config 中 bese 这部分就填写`/vuepress`, 然后进行打包操作，再然后把打包后的文件上传至云盘上的 vuepress 文件中

#### 6-2 通过服务器发布

在服务器上安装 Apache 或者 nginx 这里拿 nginx 举例：  
至于 nginx 安装 以及配置文件的解读使用 我这里就不详述了，不了解的童鞋可以自己查阅相关文件

> 同样 base 的填写很重要

```
//为了简单明了 我们把打包好的文件放入nginx中html文件下  之前的文件可以清空
server {
        listen  8081;  //监听8081端口
        server_name  localhost; //localhost:8081即可指向也可写别名如local.vuepress.com.cn  那我们访问这个端口的别名加端口就可以
        location / {
            root   /nginx/nginx-1.9.15/html; //至关重要你的入口文件在本机的位置
            index  index.html index.htm; //入口文件
        }
    }
```

#### 6-3 可以通过 koa 启用服务发布

具体代码不详述：可通过 koa 来学习