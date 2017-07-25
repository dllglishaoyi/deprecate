---
title: "JavaScript的error信息和调试"
categories:
  - JavaScript
tags:
  - JavaScript
---

> * 原文地址：[Introducing React Loadable](https://codeburst.io/javascript-error-messages-debugging-d23f84f0ae7c)
* 作者：Diogo Spínola
* 翻译人：李绍懿

&emsp;作为一个开发者，你的大部分时间都花在了阅读和调试代码上，最有可能的场景就是阅读或解决一个“意外的功能”（只是开个玩笑，准确的说应该是个“bug”）。

![图片](/assets/20170724/1.png)


&emsp;这是一个error的例子，绿框中的是整个错误信息，蓝线部分说的是错误是否被处理了，暗黄线部分说的是error的种类，红框中则是代码调用栈。

&emsp;相应的代码如下（代码仅做示例用）。
<br>
```js
(function testing(){
  var obj = {
    add
  }
  function add(a, b) {
    var result = a + b
    return result.split('')
  }
  var stringResult = obj.add("1", "2")
  var result = obj.add(1, 2)
})()
```
&emsp;我将会在调用栈部分看见这些代码，让我们先看看error都有那些类型。

### error信息的类型

&emsp;第一个告诉你代码出现了什么问题的就是我们刚才看见的著名的error信息，它经常出现在你的控制台中。
<br>
<br>
&emsp;对于那些已经习惯了编程的人来说，阅读error信息就像是条件反射。对于其他人，不管你喜不喜欢，都可以来挨个说一下这些error信息。
<br>
#### 引用错误

&emsp;接下来有个例子，当你试着使用一个并未声明的变量时，你会得到这个类型的error信息。

```js
console.log(foo) // Uncaught ReferenceError: foo is not defined
```
&emsp;这在使用*const*和*let*时也是很常见的，但它们不存在声明提升，倘若在声明之前使用变量，就会出现error，这在JavaScript里被叫做“暂时性死区”（Temporal Dead Zone（TDZ））。
```js
foo = 'Hello' // Uncaught ReferenceError: foo is not defined
let foo
```
&emsp;不论你使用什么（*var*,*const*还是*let*）修复方法很简单，只要在使用之前先声明就好了。
```js
let foo;
foo = 'Hello'
```

#### 语法错误
顾名思义，这种error会在有一些不能被语法解析的东西时出现，比如当你试着用*JSON.parse*去解析一个无效的object时。
```js
JSON.parse( {'foo': 'bar'} ) // Uncaught SyntaxError: Unexpected token o in JSON at position 1
```
这只要修正语法就可以解决，在这个例子中，object应该是一个JSON
```js
JSON.parse('{"foo":"bar"}')
```
有一些语法错误在如今大多浏览器中都会被无错处理，比如说在函数调用时传入一个逗号，但还是要在一些老浏览器中注意一下。

#### 范围错误
试着用一些无效的length去修改object就会出现这种error。
```js
var foo= []
foo.length = foo.length -1 // Uncaught RangeError: Invalid array length
```
数组长度是不可以为负数的，为什么你会弄乱数组的长度呢？有的人用它来清空数组，就像这样：
```js
var foo = [0, 0]
foo.length = foo.length - 2 // (or foo.length - foo.length)
foo // would log [] instead of [0, 0]
```
虽然我尽可能的不去修改变量（让它们作为常亮而不是变量），但这确实是一种可行的方法，而且现在你也知道了。

#### 类型错误
顾名思义，这个类型的error在操作类型（number、string等）使用有冲突时会出现，例如访问变量的一个类型为*undefined*的属性时。
```js
var foo = {}
foo.bar // undefined
foo.bar.baz // Uncaught TypeError: Cannot read property 'baz' of undefined
```
这可能是JS中最为常见的error了，认为bar是object类型，进而尝试访问其属性/方法，事实上它还没有被声明，它是undefined，并没有什么可用的baz属性。

修复很简单，只要访问之前确定bar存在，创建bar或者检查是否undefined都可以。

```js
var foo = { bar: {} }
foo.bar.baz // undefined but you avoid the error
```
或者
```js
var foo = {}
if (typeof foo.bar !== 'undefined') {
  foo.bar.baz // this will never be executed while bar does not exist
}

```
- 还有一些warning，用来告诉你一些反对使用的方法，可以在Firefox的开发者工具中经常见到。

完整的error列表可见[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Errors)

### 调试

![图片](/assets/20170724/2.png)

要调试JS代码，最简单和或许是最常规的做法就是*console.log()* 出你想要检查的变量，通过chrome开发者工具，打开你的页面和JS代码（在macOS中cmd+o，windows中Ctrl+o），选择要调试的文件，点击要调试的代码行然后刷新页面（F5）。

如果你选择的行被执行了，你将能够看见在断点之前都发生了什么并且检查之后的代码是否如你所想的结果运行。

断点也可以通过在你想要打断的代码行前添加*debugger*语句来添加。

![图片](/assets/20170724/3.png)

你也可以通过鼠标右键断点来添加条件断点，这能够让你编程的在指定条件下断点，这在当你调试大循环时想要停留在某个特定值的情况下是非常赞的。在这个例子中，断点将会在索引到达40时停下来。

在 Visual Studio Code 中使用 Node.js 时你可以按 debug tab ，然后添加一个类似这样的配置。
![图片](/assets/20170724/4.png)
<center style="font-size:12px;color:#9a9a9a">
    program项应该是你作为应用入口点的文件
</center>

你可以通过按F5或者绿色的play按钮来启动调试器。

### 调用栈

在我们首个例子中的红框部分就是调用栈，展示的是你程序断点或者错误发生的每一步的路径。

想想我们早先看到的例子：
```js
(function testing(){
  var obj = {
    add
  }
  function add(a, b) {
    var result = a + b
    return result.split('')
  }
  var stringResult = obj.add("1", "2") // stringResult becomes "12"
  var numberResult = obj.add(1, 2) // numberResult is never set, an error is thrown
})()

```

让我们顺序来看这个例子：
- 由于testing是一个IIFE (立即执行函数表达式)，它是自动执行的；
- obj被声明了，它有一个add方法（利用ES6的在object中加function的简略写法，相当于var obj = { add: add } ）；
- add方法被obj变量调用，传入了两个string作为参数，在这里它们被相加变成“12”，然后split成 [“1”, “2”]返回；
- add函数再次被调用，这传入number类型参数，相加结果是3，对其调用split（对number类型变量不可用）后抛出错误；

在chrome的开发者工具“sources”tab中配对断点和可用调用栈是很容易的。
![图片](/assets/20170724/5.png)

我加了一条debugger语句，你可以看见代码执行到断点前的“历史”。这个case中并不是个很长的调用栈，但你能开始看到为什么它这么有用，你知道add方法在14行被执行，带着什么参数（你可以在call stack下的scope里看见），而且你可以evaluate任何你点击的并显示在控制台中，在这个例子中，选中*result.split(‘’)* 然后右键点击后选择evaluate in console，你将会看到将要返回的值[“1”, “2”]在控制台中打印出来。

如果你点击继续，调用栈将会显示变化，这次add在第十五行被调用，传入的参数是number类型，evaluate *result.split(‘’)* 后，error将会被抛出。

希望这次能让你很快意识到你应该把函数名改成“concatenateAndSplit”之类的，并且校验参数类型。

调用栈在function有名字时能更好的导航，这意味着如果你使用了匿名函数，就很可能会因为调用栈不显示父函数的名字而难于调试。

举个例子，假设我们把函数从object中移除，直接调用它，并且去掉函数名，姑且把它叫做测试函数，调用栈不再能指出原始的调用了（你依旧能点击下划线来显示，但这显然是可避免的多余步骤）。
![图片](/assets/20170724/6.png)
<center style="font-size:12px;color:#9a9a9a">
    去掉函数名字的前后对比示例
</center>

我并不坚持把function挂载到object中，但我建议任何时候都给function都起上明确的名字以便调用栈更加可读。

查看栈跟踪的另一种方式就是在你的代码中想看日志的地方加上*console.trace()* 。

### 错误处理

关于我们最早那个例子中错误信息的浅蓝线标注部分，显示了我们并没有正确的处理错误，这意味着错误之后的一切都不会被执行。要避免这种情况我们通常用try catch 去捕获错误，以便我们可以得体的降级显示一个默认状态而不是错误信息（降级状态可以是一个404页面，虽然不是特别优雅，但至少比页面直接停止工作要好）。

再看看我们一直用的那个例子
```js
(function testing(){
  function add(a, b) {
    var result = a + b
    return result.split('')
  }
  var stringResult = add("1", "2")
  var numberResult = add(1, 2)
})()

```

我们现在已经知道当函数传入number时会出错，如我们前面所说，这可以用检查参数类型然后返回对应的信息的处理方式。在我们只关心参数是否是字符串的情况下这么做是可行的，但是在其他要检查所有类型的情况下，把我们的函数变成“一半校验，一般逻辑”会使它阅读和维护起来显得很笨拙。

另一个可替代方案就是把我们的函数用<b>try…catch</b>包裹起来，虽然依旧会抛出错误，但已经不是“uncaught”了。所以我们可以把它传给一个错误日志以便之后排查，同时使用一个降级预案以便我们的函数可以继续执行。
```js
(function testing(){
  function add(a, b) {
    try {
      var result = a + b
      return result.split('')
    } catch (error) {
      console.error('add went wrong ->', error)
      return [] // default value
    }
  }
  var stringResult = add("1", "2")
  var numberResult = add(1, 2)
})()
```
console.error会让你的error信息显示有略微不同，你也可以像之前一样使用某些日志系统以便你可以检查客户的报错并修复，而无需等待报告。
![图片](/assets/20170724/7.png)

使用<b>try…catch</b>最有价值的地方在于它能使你的应用保持运行，可能会有一些副作用，实际上这里传入number的结果返回一个空数组，但至程序少没有崩溃。

在类型检查变得比函数逻辑还要长时，在发送了请求但不确定相应是否改变时，在代码持续带来问题但你还没打算重构它时，我推荐使用<b>try…catch</b>。

### 避免运行时错误的工具

JS不像Java一样是编译执行的语言，所以在运行时会发生错误，这意味着任何错误信息你只有在运行代码之后才能看见。

幸运的是，使用一些工具能节约我们不少时间：
- [quokka](https://quokkajs.com/)按约定类型对代码评估
- [eslint](http://eslint.org/)用来保证你的代码风格是一致的
- 为了让JS拥有强类型的开发体验，你可以试试我们的产品[TypeScript](https://www.typescriptlang.org/)

### 结论
作为一个开发者，能够阅读错误信息并且实践debug是你最大的武器之一，经常去做的话你会注意到你花费在排查错误上的时间大大减少了。并且要记住，在commit/push之前，请移除所有的debug代码，我们并不想让客户或其他开发者卡在debugger代码上🤣。
