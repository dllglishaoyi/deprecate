---
title: "是时候再给TypeScript一次机会了【译】"
header:
  image: /assets/ts-cover.png
categories:
  - JavaScript
tags:
  - TypeScript
---

> * 原文地址：[It’s time to give TypeScript another chance](https://medium.freecodecamp.com/its-time-to-give-typescript-another-chance-2caaf7fabe61#.n95889k54)
* 作者：Jason Dreyzehne
* 翻译人：[李绍懿](https://github.com/dllglishaoyi)

&emsp;&emsp;自2012年起，TypeScript成为了一些相对结构化语言（像 C++ 或者 Java）的程序员转入JavaScript的一个流行的选择。但与此同时，它也受到了大量JS世界“原住民”们的无视。
<br>
<br>
&emsp;&emsp;你可能已经听说了Angular团队最近[为Angular 2 切换到了TypeScript](https://vsavkin.com/writing-angular-2-in-typescript-1fa77c78d8e8)。其他很多团队诸如 [RxJS](https://github.com/ReactiveX/rxjs)、
[Ionic](https://blog.ionic.io/announcing-ionic-2-0-0-final/)、
[Cycle.js](https://cycle.js.org/)、
[Blueprint](https://github.com/palantir/blueprint)、
[Dojo](https://dojotoolkit.org/community/roadmap/vision.html)、
[NativeScript](https://github.com/NativeScript/NativeScript)、
[Plottable](https://github.com/palantir/plottable)也这么做了。

&emsp;&emsp;倘若你已经接触 JavaScript/Node.js 有一段时间了，你很可能会认为上面这些项目的老大脑子进水了。或者他们可能已经被微软收买了。👀

![图片](/assets/ts-1.jpeg)
<br>

#### 有类型的JavaScript？
对于刚接触这个讨论的人来说，理解JavaScript世界对类型的厌恶是很重要的。除了JS的便携性以外，简单也是JS之所以流行的一个重要因素。

>“对于黑客来说，一门语言需要擅长编写各种他们想要的程序才够迷人。这意味着，它语言要擅长写一次性的程序。”—  Paul Graham （译者注：Paul Graham 是黑客与画家的作者，这段话在[《Being Popular》](http://paulgraham.com/popular.html)这篇文章中提到过）

对于选择JavaScript作为工具的开发者来说，他们这么做常常是因为JS的灵活性。没有标准库、很少的数据结构、无类型，JavaScript开发者在实施一些新想法的时候，不需要花费太多的时间去思考细节。

一些语言诸如C++在程序需要一大堆数据结构和其余的信息时，可能是和JavaScript差异最明显的时候。很多JavaScript程序员（特别是上面提到的那种），各种传统的类、样板、类型让他们感到枯燥，而且类型转换降低了他们的开发效率。

>“把你在过度保护编程语言中的疲惫、委屈，和蜷缩着对自由呼吸的渴望统统都交给我吧。”—JavaScript🕊️

透过这个观点，很容易明白为什么很多JavaScript用户对会对带类型的JavaScript如此抵触。


**这里有一些见解可能有助于缓解一些恐惧.**

#### TypeScript 是有更好代码纠错能力的 JavaScript

有一个可能是最常见的顾虑，那就是认为TypeScript不是纯的JavaScript。这种观点认为：因为TypeScript是一个单独的语言，所以会被转译成一大坨某一天还需要被迫去调试的糟糕代码。

![图片](/assets/ts-2.png)
<center style="font-size:12px;color:#9a9a9a">
    很多人都对TypeScript是这个印象
</center>
<br>

TypeScript 除了已被良好测试和广泛应用之外，还值得注意的是：基于你的配置，TypeScript代码事实上只会发生很少的“转译”（如果有的话）。TypeScript只是带了一些选项的Javascrip。

![图片](/assets/ts-3.png)

**TypeScript 就像一个高级的纠错器，能够检查文档，当代码没有按照预定意图使用时会发出警告。**

它为未来使用你代码的用户提供了及时的反馈和更好的开发体验。这同样也是对新项目很好的测试——如果你的项目值得去检查纠正来强制遵循编码规范，你的项目很可能从 TypeScript 中获得持久的收益。

TypeScript团队会在今后[致力于关注JavaScript](https://github.com/Microsoft/TypeScript/wiki/TypeScript-Design-Goals)。以便当JavaScript加入稳定的特性时，TypeScript 会进行相应的匹配和调整。

#### TypeScript 消除了运行时开支

另一个常见的错误观念就是 TypeScript 的类型检查会存在运行时环境中，这回带来复杂度和性能开支。

<br>
事实上，TypeScript 是一种避免运行时类型检查的好方法。

<br>

**TypeScript 是一个开发/编译时的工具**  — 它把标准的 JavaScript 加入了可选的类型提示，然后在产出时再移除。（它同样可以把ES6 和 ES7的特性转译成现在的标准）。

<br>
TypeScript的类型提示给我们带来了所有类型的优点，然后再让类型提示消失。

<br>

剩下仅存在运行时中关于对象类型的线索，和标准的 JavaScript 特性是一样的。（例如，当你从原型创建一个新对象时，你可能会用 *instanceof* 来检查它的类型）。

<br>
令人啼笑皆非的是，因为 JavaScript 并没有提供标准的开发时类型检查，很多 JavaScript 类库**重新开发了它们自己的运行时类型检测系统**。

![图片](/assets/ts-4.png)
<center style="font-size:12px;color:#9a9a9a">
    <a href="https://github.com/request/request">Request</a>类库中的运行时类型检查，它为用户在不正确的使用方法时提供了更好的debug体验。但它需要在运行时中需要更多的代码和更多的单元测试用例。<a href="https://github.com/request/request/blob/092e1e657326626da0b8ac4cfe8752751689313b/index.js#L43-L55">代码片段→</a>
</center>
<br>

这并不是这些类库的初衷，但作为好的开发体验的一部分，保证用户能在犯错时能看见清晰可执行的错误信息是必要的。

<br>
为了这个目标，很多类库在运行时大量检查传入方法的参数类型，然后抛出错误让开发者看见。

<br>
**这毫无疑问在两个世界都烂透了。** 这些运行时的类型检查让重要代码变得臃肿、可读性变差、单元测试代码覆盖率达到100%的难度提升。

![图片](/assets/ts-5.png)
<center style="font-size:12px;color:#9a9a9a">
    <a href="https://github.com/bcoin-org/bcoin/">Bcoin</a>通过提供了运行时抛出有用的错误信息提升了开发体验。但运行时的类型检查加大了维护和测试成本。这要用 TypeScript 来做的话会更高效和有帮助。 <a href="https://github.com/bcoin-org/bcoin/blob/4e7df6ef875e5936bea5139d922871498b4d9586/lib/primitives/tx.js#L84-L123">代码片段→</a>
</center>
<br>

不使用 Typescript 的话，不仅在开发时失去了类型检查，而且还可能经常把它带到运行时中。（我希望你能有100%的测试覆盖率。）

<br>

当你使用 TypeScript 时，你给用户提供了一个更好的开发体验、减少了了运行时的类型检查（除非必须的场景，如用户输入），让你的代码单元测试更容易完全覆盖。

<br>
#### TypeScript 已由来已久

可能是因为以上提到的原因，当我第一次听说 TypeScript 时，我跑的远远的。而且它是微软出品，站在了“JavaScript精华”（较少结构）的对立面。

<br>
但现在已经不再是2012年了。TypeScript 不是 JavaScript 的 [抽象漏洞](https://en.wikipedia.org/wiki/Leaky_abstraction)，TypeScript 项目中有很多顶级的黑客和工程师。（我很钦佩微软能够把项目管理得这么好。）

**自从 TypeScript 跟踪了 ECMAScript，使用 TypeScript 就不是把你的项目锁在了一门新语言中。** 很多人还没有意识到这一点，所以听到这种观点并不罕见：
>“维护一个 TypeScript 项目真难。”

这对于我听起来就像：

>“维护一个有代码检查纠错的项目真难。”

如果你的项目出于某种原因要停止从 TypeScript 中收益了，你可以用编译器跑一下你的项目，用以把所有类型从你的代码库中移除。

<br>

然后你就回到无类型的 JavaScript 了。

#### 长话短说
TypeScript 最近得到了很大的改进提升。如果你几年前就听说 TypeScript 了，但是在那以后却没有跟进，现在值得再看一看。

### 什么时候用 TypeScript


#### [Angular: 为什么是 TypeScript?](https://vsavkin.com/writing-angular-2-in-typescript-1fa77c78d8e8)

一个关于Angular团队到底为什么要选择用TypeScript构建Angular 2简短的技术讨论。

#### [所有应该在TypeScript中编写的JS类库](http://staltz.com/all-js-libraries-should-be-authored-in-typescript.html)

一篇为什么Typescript用构建JS类库是个好主意的总结，由Cycle.js 的作者、RxJS贡献者分享。

#### [深入 Typescript— 为什么是 TypeScript](https://basarat.gitbooks.io/typescript/content/docs/why-typescript.html)

一篇关于用TypeScript好处的总结。

### 学习 TypeScript

#### [Typescript教程](https://www.typescriptlang.org/docs/tutorial.html)

一篇 TypeScript 团队维护的简短教程。

#### [Typescript设计目标](https://github.com/Microsoft/TypeScript/wiki/TypeScript-Design-Goals)

一篇概括了TypeScript团队整体设计原则的wiki。

![图片](/assets/ts-6.png)

#### [typescript-starter](https://github.com/bitjson/typescript-starter)

一个开发JavaScript类库的脚手架工程。包含了单元测试、文档生成还有CommonJS和ES6模块引入（针对Node.js和浏览器环境）。

<br>

我抱着改变观念的希望写了这篇文章。如果有什么我可以改进这篇文章的想法，请
[告诉我](https://twitter.com/bitjson)
<br>
如果你觉得这篇文章有趣的话，请分享并为我点赞♡，谢谢阅读！


[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
