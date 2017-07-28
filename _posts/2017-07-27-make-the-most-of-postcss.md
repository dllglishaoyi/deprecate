---
title: "如何选择postcss插件"
categories:
  - postcss
tags:
  - postcss 
---

### 前言
前端技术发展到今天，不断涌现出了各种优秀的自动化工具，让前端的工程化日渐成熟。在css领域，postcss无疑是其中的杰出代表。

### 预处理器 or 后处理器
目前主流的css自动化工具主要分为两大方向：预处理器（*preprocessor*）和后处理器（*postprocessor*）
#### 预处理器
css预处理器就是把非css（如less，sass）语言转化为css的工具。这些语言提供了一些css不具备的语法特性，相较于css更易读、更简洁、更像编程语言。
![图片](/assets/20170727/1.png)
#### 后处理器
后处理器的输入是css，输出也是css，听上去似乎并没有什么用，但其实及其有用和强大，目前技术社区有种类繁多的css后处理器，可以为css提供各种各样的额外功能。
![图片](/assets/20170727/2.png)
由于css后处理器的输入和输出都是css，这意味着，它们可以被串联起来，进行功能叠加。

### postcss
Postcss就是一个工具平台，提供了把各种css处理器作为平台插件串联起来的能力。

### postcss插件
看看官网上的插件[主页](https://www.postcss.parts/)，各种类型插件琳琅满目,我们应该如何选择呢？
![图片](/assets/20170727/3.png)

工具的选择通常是根据要解决的问题去进行的。那么我们围绕使用和编写css的一些常见痛点来挑选合适的插件吧。

>  postcss插件的使用方法可以阅读相关文档，本文默认读者已经搭建好各自的postcss使用环境

### 恼人的厂商前缀
为了保证样式对浏览器的兼容性，我们需要对flex、border-radius等很多常用的属性加上
-webkit-、-moz-、-o-、-ms-这些厂商属性，在一个庞大的应用里手动去做这就是的话，简直是个灾难。

#### [Autoprefixer] (https://github.com/postcss/autoprefixer)
 Autoprefixer是最流行的PostCSS插件，让你从手动加浏览器厂商前缀的额外工作中解放出来。

 <br>
编写

 ```css
:fullscreen a {
    display: flex
}
```
生成
```css
:-webkit-full-screen a {
    display: -webkit-box;
    display: flex
}
:-moz-full-screen a {
    display: flex
}
:-ms-fullscreen a {
    display: -ms-flexbox;
    display: flex
}
:fullscreen a {
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex
}
```
### css代码压缩
生产环境中，为了减少网络传输时间，提升用户体验，除了压缩js代码之外，css的压缩也是有必要的
#### [cssnano](http://cssnano.co/)
cssnano是postcss生态系统中最好的css压缩工具

### 使用css新语法特性
和JS一样，css也在不断发展，也一直在添加新的语法特性，但浏览器的支持速度参差不齐。对于爱搞事情的前端同学，很多css新特性看得见却用不了，很是苦恼。

#### [cssnext](http://cssnext.io/)
它的slogan就是 *Use tomorrow’s CSS syntax, today.* 在我看来，它就是css界的babel。
能支持这么多特性，还是挺酷的。
![图片](/assets/20170727/4.png)

### css质量和风格
css也需要代码检查，它可以维护代码的一致性，解析代码中的错误，减少冗余代码等等。
#### [stylelint](https://stylelint.io/)
如同js中的eslint一样，stylelint对于css检查也非常强大和灵活，使用者可以通过配置文件定义检查规则，此外，它还支持css最新的语法和less、sass等类css文件的检查。

### 图片和字体
平常在编写css过程中免不了很多跟图片/字体相关的工作，如何能把这些事情自动化，也成为开发者的一个痛点。
#### [postcss-assets](https://github.com/assetsjs/postcss-assets)
内联图片、计算图片尺寸、url解析等
#### [postcss-sprites](https://github.com/2createStudio/postcss-sprites)
帮助生成雪碧图
```css
/* Input */
.comment { background: url(images/sprite/ico-comment.png) no-repeat 0 0; }
.bubble { background: url(images/sprite/ico-bubble.png) no-repeat 0 0; }

/* ---------------- */

/* Output */
.comment { background-image: url(images/sprite.png); background-position: 0 0; }
.bubble { background-image: url(images/sprite.png); background-position: 0 -50px; }
```

#### [font-magician](https://github.com/jonathantneal/postcss-font-magician)
可以自动生成字体规则
```css
/* before */

body {
   font-family: "Alice";
}

/* after */

@font-face {
   font-family: "Alice";
   font-style: normal;
   font-weight: 400;
   src: local("Alice"), local("Alice-Regular"),
        url("http://fonts.gstatic.com/s/alice/v7/sZyKh5NKrCk1xkCk_F1S8A.eot?#") format("eot"),
        url("http://fonts.gstatic.com/s/alice/v7/l5RFQT5MQiajQkFxjDLySg.woff2") format("woff2"),
        url("http://fonts.gstatic.com/s/alice/v7/_H4kMcdhHr0B8RDaQcqpTA.woff")  format("woff"),
        url("http://fonts.gstatic.com/s/alice/v7/acf9XsUhgp1k2j79ATk2cw.ttf")   format("truetype")
}

body {
  font-family: "Alice";
}
```

### 编程语言的语法特性
 我需要像编写less/scss一样编写css，享受它们带来的一些编程语言特性，post也有相应插件支持，这意味着以后可以不用less/scss文件了，直接用css文件编写less/scss语法就OK。
#### [postcss-scss](https://github.com/postcss/postcss-scss)
#### [postcss-less](https://github.com/webschik/postcss-less)

### 其他
以上的一些插件已经满足我们日常的大部分开发场景，但postcss插件社区远比这些精彩，还有很多有趣的插件，值得去好好探究一番，希望大家都能早日找到适合自己团队项目的postcss工具集

#### 参考文章
[http://andrewhfarmer.com/how-to-style-react/](http://andrewhfarmer.com/how-to-style-react/)
[https://www.layerpoint.com/postcss-plugins/](https://www.layerpoint.com/postcss-plugins/)
[http://hao.jobbole.com/postcss/](http://hao.jobbole.com/postcss/)



