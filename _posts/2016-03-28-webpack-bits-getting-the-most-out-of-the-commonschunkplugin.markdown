---
title: "webpack 小札: 充分利用 CommonsChunkPlugin()【译】"
categories:
  - 前端工具
tags:
  - Webpack
date:   2017-03-28 10:01:53 +0800
---
> * 原文地址：[webpack bits: Getting the most out of the CommonsChunkPlugin()](https://medium.com/webpack/webpack-bits-getting-the-most-out-of-the-commonschunkplugin-ab389e5f318#.o691521qn)
* 作者：Sean T. Larkin
* 翻译人：[李绍懿](https://github.com/dllglishaoyi)

webpack 核心团队喜欢不时地参与到Twitter社区中，通过[一些有趣并且有营养的方式分享知识](https://twitter.com/TheLarkInn/status/842817690951733248)。

![图片](/assets/demo.gif)

![图片](/assets/20170328/twitter1.png)

这一次，“游戏规则”很简单。安装 webpack-bundle-analyzer ，生成一张精美多彩的图片，包含了你所有的bundle，然后分享给我。作为回报，webpack团队会帮你指出一些我们能辨认的潜在问题。

## 我们发现了什么？

最常见的问题就是代码重复：重复的类库、组件、代码存在于多个（同步或异步）的bundle里！

## 例子1：很多带有重复代码的 vendor bundle
<br>
![图片](/assets/20170328/case1.png)
![图片](/assets/20170328/twitter2.png)
<center style="font-size:12px;color:#9a9a9a">
    这是一个典型例子。Swizec谢谢你让我分享它。
</center>
<br>
[Swizec Teller](https://medium.com/@swizec)人很好，分享了他的一个构建（实际上是包含了8-9个独立单页的应用）。我从中挑选了这个例子因为可以从中辨认出很多很棒的技术。让我们具体来看看：
<br>
<br>
![图片](/assets/20170328/case1.png)
<center style="font-size:12px;color:#9a9a9a">
    除了“FoamTree”图表之外都是应用的代码，与此同时，所有用了node_modules资源的部分都以"\_vendor.js" 结尾
</center>
<br>
我们可以从中推断出不少东西（在不看具体配置的情况下）。
<br>

每个单页应用都仅仅在入口和vendor代码中用了 *CommonsChunkPlugin* 插件。这创建了一个只包含来自node_modules文件代码的bundle，并且其他的bundle只包含了应用代码。以下是部分相关配置：
![图片](/assets/20170328/twitter3.png)

{% highlight javascript %}
Object.keys(activeApps)
  .map(app => new webpack.optimize.CommonsChunkPlugin({
    name: `${app}_vendor`,
    chunks: [app],
    minChunks: isVendor
  }))
{% endhighlight %}

*activeApps* 变量代表了每个独立的入口点。

## 机会区域

下面是我圈出来可以做一些改善的区域。
<br>
<br>
![图片](/assets/20170328/case1_2.png)
## “元” 缓存

我们上面看到的是各种诸如 momentjs, lodash 和 jquery 的庞大类库穿插在6个甚至更多的vendor bundle中使用。把所有vendor打包到一个单独bundle的策略固然是好的，但我们也应该相同的策略应用到所有的 vendor bundle内部。

{% highlight javascript %}
new webpack.optimize.CommonsChunkPlugin({
  children: true,
  minChunks: 6
})
{% endhighlight %}

这相当于我们告诉webpack:
>嘿，webpack, 看看所有的 chunk （包括已生成的vendor），然后把出现在6个以上chunk中的模块放到一个单独的文件中。

![图片](/assets/20170328/case1_3.jpg)
![图片](/assets/20170328/twitter4.png)
<center style="font-size:12px;color:#9a9a9a">
    在这个例子中这个文件似乎被命名为“manifest.js”
</center>
![图片](/assets/20170328/twitter5.png)
<br>
<br>
正如你所见，所有的特定模块被抽离到一个独立的文件，并且据
Swizec说，这让他们的应用大小整体下降了17%!
## 例子2：异步 chunk 之间的重复 vendor
![图片](/assets/20170328/twitter6.png)
<center style="font-size:12px;color:#9a9a9a">
    这是个令人印象深刻的代码拆分用法。瞧瞧这些漂亮的颜色 💓
</center>
<br>
<br>
所以这个代码的重复量相对整体代码不那么严重吧，然而，当你看到下面全量代码体积的图时，你会发现有三个同样的模块存在于每个异步chunk中。
<br>
<br>
![图片](/assets/20170328/case2.jpeg)
<center style="font-size:12px;color:#9a9a9a">
    异步chunk就是文件名为“[number].[number].js”的文件
</center>
<br>
<br>
正如你在上面看见的，有2，3个相同的组件用到了四五十个异步bundle中。那么我们怎么通过 *CommonsChunkPlugin* 解决这个问题呢？
## 创建异步的 Commons Chunk
这个技术和第一个非常相似，我们要把配置文件中的 *async* 属性设为 *true* ，像这样：

{% highlight javascript %}
new webpack.optimize.CommonsChunkPlugin({
  async: true,
  children: true,
  filename: "commonlazy.js"
});
{% endhighlight %}

用同样的方式 - webpack 会扫码所有的 chunk 来查找通用模块。当async为true时，只有拆分代码的bundle才会被扫描。
由于我们并没有指定*minChunks*，所以默认值是3。webpack相当于被告知：
>嘿，webpack, 查看所有的常规（又称懒加载）chunk，如果你找到了出现在3个以上chunk中的相同模块，那么请把它抽离到一个单独的异步通用chunk中。

最终的结果就是：
![图片](/assets/20170328/case2_2.jpg)
![图片](/assets/20170328/twitter7.png)
<center style="font-size:12px;color:#9a9a9a">
可能有更大的minChunks值，来产生一个更小的commonlazy.js包
</center>
<br>
<br>
现在的异步chunk已经非常小了，并且所有的代码都聚合到了一个叫 *commonlazy.js* 的文件中。由于这些bundle已经很小了，所以大小的影响直至第二次访问都不是很明显。
现在，被分隔到每个bundle中的代码少了很多，我们通过将这些通用模块放入单独的可高速缓存的chunk中来节省用户加载时间和数据的消耗。

## 更进一步控制: minChunks 函数
![图片](/assets/20170328/twitter8.png)
<br>
<br>
如果你想要更多控制会怎么样？有时候你并不想要单个共享的bundle，因为并不是每个 *懒/入口* chunk 都会用到它。 *minChunks* 属性也支持 function ！！这可以让你筛选哪些模块会被加入你新的bundle中。举个例子：

{% highlight javascript %}
new webpack.optimize.CommonsChunkPlugin({
  filename: "lodash-moment-shared-bundle.js",
  minChunks: function(module, count) {
    return module.resource && /lodash|moment/.test(module.resource) && count >= 3
  }
})
{% endhighlight %}

上面的例子就是说：

>哟，webpack, 当你扫描模块时候，如果模块的绝对路径匹配到了lodash或者momentjs并且出现在了三个入口文件中，那么把这些模块打包到一个bundle中。

你可以通过设置“async：true”将同样的行为应用于异步bundle。



## 再多的控制
![图片](/assets/20170328/twitter9.png)

通过这个 *minChunks* 你可以为特定的入口和bundle创建更小的可缓存vendor集合。最后，你看到的可能会是这样：

{% highlight javascript %}
function lodashMomentModuleFilter(module, count) {
  return module.resource && /lodash|moment/.test(module.resource) && count >= 2;
}
function immutableReactModuleFilter(module, count) {
  return module.resource && /immutable|react/.test(module.resource) && count >=4
}
new webpack.optimize.CommonsChunkPlugin({
  filename: "lodash-moment-shared-bundle.js",
  minChunks: lodashMomentModuleFilter
})
new webpack.optimize.CommonsChunkPlugin({
  filename: "immutable-react-shared-bundle.js",
  minChunks: immutableReactModuleFilter
})
{% endhighlight %}

## 没有银弹！

*CommonsChunkPlugin()* 或许很强大，但请记住，这些例子都有它们所适用的场景。所以当你复制粘贴这些代码片段之前，从[Sam Saccone](https://medium.com/@samccone)、[Paul Irish](https://medium.com/@paul_irish)还有[MPDIA](https://youtu.be/6m_E-mC0y3Y?t=11m38s)那先获得一些建议，以确保你使用的是正确方案。

![图片](/assets/20170328/advice.png)
<center style="font-size:12px;color:#9a9a9a">
在运用解决方案之前，请务必了解你的进程
</center>

## 从哪找到更多的例子？
这些只是使用和配置 *CommonsChunkPlugin()* 的例子。要想了解更多，在我们的 webpack/webpack GitHub 核心仓库中查看 */examples* 目录！如果你还有更好的想法,欢迎提交PR!
