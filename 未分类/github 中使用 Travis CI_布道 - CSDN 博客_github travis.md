> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/alex_xfboy/article/details/85626365)

编写代码只是软件开发的一小部分，更多的时间往往花在构建（build）和测试（test）。为了提高软件开发的效率，构建和测试的自动化工具层出不穷。Travis CI 就是这类工具之中，市场份额最大的一个。

**什么是持续集成？**

持续集成指的是只要代码有变更，都会触发 CI 服务器自动对项目进行自动运行构建，自动运行测试，反馈运行结果，最终甚至自动部署到测试环境。这样做的好处就是，随时发现问题，随时修复。因为修复问题的成本随着时间的推移而增长，越早发现，修复成本越低。

**CI 就是持续集成的?**

这个是一个很大的误区，CI 是解决问题的手段而不是目的，目的要如何提高代码质量。在有些公司的项目，编译一次半小时，上传到启动一次十几分钟，测试一次几个小时。不能在短时间内完成编译、测试的代码都有问题，导致 CI 形同虚设。这里的 “短时间” 是指 5 分钟以内。

**持续集成的困难有哪些？**

说实话个人认为，持续集成要实现快速的流水线作业，难点在于测试，特别是微服务环境下的系统间相互依赖，版本众多，是提供的集成化单元测试还是单元测试（粒度太小也太弱了），在[之前](https://blog.csdn.net/alex_xfboy/article/details/85006654)已经探讨过。

**Travis CI 是什么东东？**

Travis CI 是在线托管的 CI 服务，用 Travis 来进行持续集成，不需要自己搭服务器，傻瓜式操作，用起来更方便。最重要的是，它对开源项目是免费的。如果是私有项目或非开源项目，可以使用收费的 [Travis Pro](https://travis-ci.com/) 。

**谁在用？**

因为 GitHub 和 Travis 是一对好基友，不用 GitHub 虽然也能用 Travis，但是配置起来太麻烦。其实大多开源项目都在使用，都可能见过这个图标 build:passing  ，如果你 push 过开源代码，都知道，它会给你一个构建反馈（通过 Travis 实现的）。

![](https://img-blog.csdnimg.cn/20190102173123609.png)

这里再介绍一个高效的工具，![](https://img-blog.csdnimg.cn/20190102173543399.png)它提供 IM 功能（[https://gitter.im](https://gitter.im)），也是可以与 github 集成的，方便在上面与作者沟通反馈问题。

**支持哪些语言？**

它支持市面上所有的语言，[了解更多](https://docs.travis-ci.com/user/tutorial/)

**如何使用？**

1.  使用 GitHub 账号登录 Travis CI 的网站：https://travis-ci.org/
2.  编写. travis.yml 文件来告诉 Travis 一些项目信息，并 push，它会默认触发（install: mvn install -DskipTests=true -Dmaven.javadoc.skip=true）。当然如果不能满足你的需要也可以编写 [script](https://docs.travis-ci.com/user/deployment/script/)。
3.  添加图片 [![Build Status](https://travis-ci.org/xxx/xxx.svg?branch=master)](https://travis-ci.org/xxx/xxx)

.travis.yml 的编写规则，具体可参考[官方指导](https://docs.travis-ci.com/user/tutorial/)。

**webhooks 是什么?**

webhooks 是 github 针对每个项目提供的通知服务（可以理解是个触发器），非常有用。[了解更多](https://developer.github.com/webhooks/)

![](https://img-blog.csdnimg.cn/20190102175948759.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2l0c29mdGNoZW5mZWk=,size_16,color_FFFFFF,t_70)