> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6844903935027707918) ![](https://user-gold-cdn.xitu.io/2019/9/4/16cfbb0e4bb449c7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 前言：  
> 希望将笔记、文档发布到网络上，以前最好的方式可能是 GitBook，由于 GitBook 已升级 2.0，不再是免费的午餐。然后，找到了 [VuePress](https://vuepress.vuejs.org/) 界面清新，很适合笔记、文档等。
> 
> 使用过程中发现需要手动设置侧边栏、导航栏，导入大量笔记就很费时了。准备动手写个插件来自动生成侧边栏。此时，发现了 [vuepress-bar](https://github.com/ozum/vuepress-bar) 一个土耳其小伙的作品，比较完美实现了自动生成目录和导航栏，而且与我的设计思路惊人的一致，欣喜。
> 
> 但是，这个插件并未按照 vuepress-plugin 的方式编写，所以引入还是需要一些配置，也没有使用 vuepress 的 API 接口，经过一顿魔改后。终于完美实现了需求。

* * *

目标
--

安装插件 [vuepress-plugin-autobar](https://github.com/boboidream/vuepress-bar)，会将 SourceDir (通常是 ./docs) 中所有文档按约定生成导航栏和侧边栏。

约定
--

1.  `nav` 开头的文件夹将会成为导航栏，如果手动配置了 `themeConfig.nav` 将自动合并。
2.  `nav.10.文件名`, `10.文件名`…… 会自动去掉前缀、排序数字。
3.  非 `nav` 开头文件夹会成为 Group。
4.  文件夹 `01.guide--nc` 以 `--nc` 结尾意为设置 Group 默认展开。
5.  文件夹 `01.guide--d2` 以 `--d数字` 结尾，为设置目录深度。

安装
--

```
npm install -D boboidream/vuepress-bar
复制代码
```

```
// 配置中添加插件
// .vuepress/config.js
// or
// .vuepress/theme/index.js

module.exports = {
  plugins: ['autobar']
}
复制代码
```

效果
--

线上体验：[note.wenboz.com/js/](https://note.wenboz.com/js/)

Tips
----

1.  如使用 [vuepress-plugin-permalink-pinyin](https://github.com/viko16/vuepress-plugin-permalink-pinyin) 此插件，请在`autobar`插件配置中设置 `pinyinNav: true` ，将自动完成拼音地址兼容。
    
    ```
    // 配置中添加参数
    // .vuepress/config.js
    // or
    // .vuepress/theme/index.js
    module.exports = {
      plugins: ['permalink-pinyin', ['autobar', {'pinyinNav': true}]],
    }
    复制代码
    ```
    
2.  如果觉得生成链接太丑，可以使用 [vuepress-plugin-rpurl](https://github.com/boboidream/vuepress-plugin-rpurl) 插件，此插件会默认将 autobar 生成链接优化：
    
    *   before use:  
        `/nav.10.js/10-core/mian-xiang-dui-xiang/mian-xiang-dui-xiang.html`
        
    *   after use:  
        `/js/core/mian-xiang-dui-xiang/mian-xiang-dui-xiang.html`
    
3.  更多高级配置：[github.com/boboidream/…](https://github.com/boboidream/vuepress-bar)

## 测试的测试测试测试test