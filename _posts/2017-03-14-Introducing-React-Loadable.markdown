---
title: "React Loadable简介[译]"
categories:
  - React
tags:
  - React
---

> * 原文地址：[Introducing React Loadable](https://medium.com/@thejameskyle/react-loadable-2674c59de178)
* 作者：james kyle
* 翻译人：李绍懿

&emsp;当你的项目足够大时，把所有代码打包到一个bundle中的启动时间就会成为问题。这时就需要把app拆分为若干个bundle，然后根据需求动态加载它们。

![图片](/assets/image1.png)
<center style="font-size:12px;color:#9a9a9a">
    一个大bundle VS 若干个小bundle
</center>
<br>

&emsp;那如何把一个bundle拆成几个呢？这个问题其实已经被 [Browserify](https://github.com/substack/factor-bundle) 和 [Webpack](https://webpack.js.org/guides/code-splitting/) 这些工具解决得很好了。
<br>

&emsp;但还要做的是在项目中找到合适的地方拆分bundle，然后异步去加载。所以当项目中有东西在加载时的需要一种通信机制。
<br>

### 基于路由拆分 vs 基于组件拆分

&emsp;通常的推荐做法就是把app根据路由进行拆分，然后异步地去加载每一个。看上去这种做法对于大多数app已经足够好，例如点击一个连接然后加载一个新页面，这种体验还不赖。
<br>
<br>
&emsp;但是，我们可以做得更好。
<br>
<br>
&emsp;其实在大多数React的路由管理工具中，路由以组件的形式存在。它们并没有什么非常特别的地方。所以假设我们围绕着组件优化而不是把责任推给路由会怎么样？这样会给我们带来什么？

![图片](/assets/image2.png)
<center style="font-size:12px;color:#9a9a9a">
    基于路由 VS 基于组件代码拆分
</center>
<br>
&emsp;这会有很多结果。相比只是简单根据路由拆分app，这样做会有更多地方可以拆分。例如 Modals、tabs ，还有很多在用户做相应操作之前隐藏内容的组件。
<br>
<br>
&emsp;更别说那些需要推迟到高优先级内容加载完成后才加载的内容了。一个在页面底部而且依赖了一大串类库的组件为什么要和页面顶部的内容同时加载呢？
<br>
<br>
&emsp;你大可依然在路由只是简单组件时拆分他们。对于你的app，不管黑猫白猫，捉到老鼠就是好猫。
<br>
<br>
&emsp;但我们需要在让组件层面拆分app像在路由层面拆分一样简单。简单得只要改几行代码，其他的事就自动OK。

### React Loadable简介
<br>
&emsp;React Loadable 是一个很小的库，是我[厌烦了你们总说这个很难做](https://twitter.com/thejameskyle/status/839916840973299713) 之后写出来的。
<br>
<br>
&emsp;Loadable 是一个高阶组件（创建组件的function）用来轻易地在组件层面拆分bundle。
<br>
<br>
&emsp;我们试想一下有两个组件，其中一个引入并渲染了另一个。
<br>
{% highlight javascript %}
import AnotherComponent from './another-component';

class MyComponent extends React.Component {
  render() {
    return <AnotherComponent/>;
  }
}
{% endhighlight %}
<br>
&emsp;此时我们依赖了AnotherComponent并且通过import关键字同步引入。我们需要一种让它异步加载的方法。
<br>
<br>
&emsp;使用ECMA中**动态引用**（[一个T39提案，目前stage3](https://github.com/tc39/proposal-dynamic-import) ）的特性来修改我们的组件使之异步加载AnotherComponent。
<br>
{% highlight javascript %}
class MyComponent extends React.Component {
  state = {
    AnotherComponent: null
  };

  componentWillMount() {
    import('./another-component').then(AnotherComponent => {
      this.setState({ AnotherComponent });
    });
  }

  render() {
    let {AnotherComponent} = this.state;
    if (!AnotherComponent) {
      return <div>Loading...</div>;
    } else {
      return <AnotherComponent/>;
    };
  }
}
{% endhighlight %}
<br>
&emsp;然而，这只是手动做法，并不适用大量其他各种各样的场景。比如说当import()失败的情况，以及服务端渲染的情况。
<br>
<br>
&emsp;作为替代，你可以使用 Loadable 把问题抽象出来。Loadable的用法很简单。你仅仅要做的就是把要加载的组件和当你加载组件时的“Loading”组件传入一个方法中。
<br>
{% highlight javascript %}
import Loadable from 'react-loadable';

function MyLoadingComponent() {
  return <div>Loading...</div>;
}

const LoadableAnotherComponent = Loadable(
  () => import('./another-component'),
  MyLoadingComponent
);

class MyComponent extends React.Component {
  render() {
    return <LoadableAnotherComponent/>;
  }
}
{% endhighlight %}
<br>
&emsp;但是如果组件加载失败怎么办，我们还需要一个错误状态提示。
为了让你最大化控制要显示的东西，错误提示只是简单地作为LoadingComponent的一个prop传入。
<br>
{% highlight javascript %}
function MyLoadingComponent({ error }) {
  if (error) {
    return <div>Error!</div>;
  } else {
    return <div>Loading...</div>;
  }
}
{% endhighlight %}
<br>

#### 基于import()的自动代码拆分

&emsp;import()的牛X之处在于 Webpack 2 可以自动拆分代码，不论你在何时加入新代码，都不用做其他额外的工作。
<br>
<br>
&emsp;这意味着你在使用 React Loadable 时，你可以通过切换 import() 位置来轻易试验代码拆分点，以便让你的app达到最佳性能。[你可以在这查看示例工程](https://github.com/thejameskyle/react-loadable-example)。或者[查看 Webpack 2 文档](https://webpack.js.org/guides/code-splitting-import/)（提示：一些相关文档在[require.ensure()](https://webpack.js.org/guides/code-splitting-require//) 一节中）
<br>

#### 避免组件加载闪烁
&emsp;有时组件加载非常快(<200ms)，这时加载中的样式就会一闪而过。
<br>
<br>
&emsp;有大量用户研究表明，这样会让用户感觉到比实际加载更长的等待时间。如果什么都不显示的话，用户会感觉更快。所以Loading组件需要接收一个pastDelay prop。
<br>
<br>
&emsp;这样你的Loading组件只在加载时间比设定delay时间长时才会显示。
<br>
{% highlight javascript %}
export default function MyLoadingComponent({ error, pastDelay }) {
  if (error) {
    return <div>Error!</div>;
  } else if (pastDelay) {
    return <div>Loading...</div>;
  } else {
    return null;
  }
}
{% endhighlight %}
<br>
&emsp;这个 delay 默认200ms，但你也可以给Loadable传入第三个参数用来自定义这个值。
<br>
{% highlight javascript %}
Loadable(
  () => import('./another-component'),
  MyLoadingComponent,
  300
);
{% endhighlight %}
<br>

#### 预加载

&emsp;作为优化，你也可以在组件渲染之前对它进行预加载。举个例子，当你需要在点击按钮时加载一个新组建，可能需要用户hover在按钮上时就预加载它。
<br>
<br>
&emsp;Loadable 创建的组件向外暴露了一个用于预加载的静态方法，具体如下：
<br>
{% highlight javascript %}
let LoadableMyComponent = Loadable(
  () => import('./another-component'),
  MyLoadingComponent,
);

class MyComponent extends React.Component {
  state = { showComponent: false };

  onClick = () => {
    this.setState({ showComponent: true });
  };

  onMouseOver = () => {
    LoadableMyComponent.preload();
  };

  render() {
    return (
      <div>
        <button onClick={this.onClick} onMouseOver={this.onMouseOver}>
          Show loadable component
        </button>
        {this.state.showComponent && <LoadableMyComponent/>}
      </div>
    )
  }
}
{% endhighlight %}
<br>

#### 服务端渲染

&emsp;Loadable 通过控制最后一个参数同样支持服务端渲染。服务端运行时，通过传入要动态加载模块的绝对路径来允许 Loadable 同步 reqire() 模块。

<br>
{% highlight javascript %}
import path from 'path';

const LoadableAnotherComponent = Loadable(
  () => import('./another-component'),
  MyLoadingComponent,
  200,
  path.join(__dirname, './another-component')
);
{% endhighlight %}
<br>

&emsp;这意味着你的“异步加载”和“代码拆分”模块在服务端都是同步渲染。

<br>

&emsp;此时在客户端遇到的问题回来了。我们可以在服务端完整渲染应用，但在客户端，我们同一时间只需要加载一个bunle。

<br>
&emsp;设想一下如果我们能弄清楚服务端bundling进程中哪些bundle是我们所需的会怎样？这样我们就可以把这些bundle一下传给客户端并且带上服务端渲染的确切状态。

<br>
&emsp;今天你其实离这个目标很近了。

<br>
&emsp;因为我们在Loadable中掌握了所有server端依赖的路径，我们可以添加一个新的*flushServerSideRequires*方法用来返回所有在服务端渲染的路径。然后用*webpack --json*命令，我们就可以获得一个匹配了对应文件的bundle（[我的具体代码](https://gist.github.com/thejameskyle/abecfe8ec2a7ce1e312a904527a31908)）。

<br>
&emsp;仅剩的问题就是如何在客户端优美地使用 Webpack 。我会在发表完这篇文章后一直等待你们的消息。

<br>
&emsp;这就是所有的“酷玩意”（原文：“cool shit”），我们可以把它们优雅地结合在一起。*React Fiber*（译者注：[React Fiber是React核心算法的重写](https://github.com/acdlite/react-fiber-architecture)）让我们更智能指定哪些bundle需要直接传输而哪些需要推迟到更高优的工作完成后再加载。

<br>
&emsp;最后，请您把这玩意安装上再[帮给我的repo给颗星](https://github.com/thejameskyle/react-loadable)

<br>
{% highlight javascript %}
yarn add react-loadable
# or
npm install --save react-loadable
{% endhighlight %}
<br>

[我也在twitter上](https://twitter.com/thejameskyle)


[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
