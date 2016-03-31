---
layout: post
title:  "第三章: 函数 vs. 块级作用域"
date:   2016-03-30 14:01:53 +0800
categories: jekyll update
---

# 你不知道的JS:作用域和闭包
# 第三章: 函数 vs. 块级作用域

我们在第二章探索过，作用域由一系列“泡泡”组成，这些“泡泡”就像装着标识符（变量、方法）的桶。这些“泡泡”整齐地嵌套，并且这些嵌套在写代码时就定义好了。

但到底是什么构造了一个新的泡泡？只能是函数吗？JavaScript中还有别的什么结构能创造作用域泡泡呢？

## 函数作用域

对于这些问题，最普通的答案莫过于JavaScript拥有基于函数的作用域了。就是说，你声明的每个函数都为它自己创造了一个泡泡，但没别的结构能为自己创建泡泡了。我们来看看为什么这么说，尽管这不是特别准确。

首先，我们来看看函数作用域和它的实现。

有如下代码：

```js
function foo(a) {
	var b = 2;

	// some code

	function bar() {
		// ...
	}

	// more code

	var c = 3;
}
```

在这段代码当中，`foo(..)`的作用域泡泡包含了标识符`a`, `b`, `c` 和 `bar`。不论在作用域的那个地方声明变量，变量和方法都属于这个作用域泡泡。这一点我们会在下一章具体讨论。

`bar(..)`拥有它自己的作用域泡泡。全局作用域也有一个属于自己的泡泡，它只包含一个名叫`foo`的标识符。

因为`a`, `b`, `c`, 和 `bar`都属于`foo(..)`的作用域泡泡，他们是无法从`foo(..)`外面访问的。也就是说，以下的代码会引起`ReferenceError`报错，因为这些标识符是无法从全局作用域访问的：

```js
bar(); // 失败

console.log( a, b, c ); // 三个都失败
```

然而, 所有的这些标识符 (`a`, `b`, `c`, `foo`, 和 `bar`)是可以从`foo(..)`的 *外部*访问的, 而且 `bar(..)`内部也可以访问 (假设`bar(..)`内没有声明覆盖他们).

函数作用域鼓励一直思想：所有变量属于函数，函数内部（甚至在内部的嵌套作用域中）处处可以访问他们。这种设计方式是非常有用的，它可以把JavaScript按需处理不同类型值得动态特性发挥到极致。

另一方面，如果你疏于防范，存在于整个作用域中的变量会产生一些意想不到的隐患。

## 藏在作用域中

通常对函数的想法就是声明一个函数，然后再在其中加入代码。其实反过来也同样强大和有用：选择一段你写过的代码，然后用一个函数声明把它包裹起来，这样就“隐藏”了这段代码。

这样实际上是创造了一个环绕代码的作用域泡泡，也就是说，任何代码中的声明（变量和方法）现在都会被新的函数作用域包裹起来。换句话说，你可以通过用函数作用域包裹的方法吧变量“隐藏”起来。

为什么说“隐藏”变量和函数是一种有用的技术呢？

有几个原因。它们遵循了软件设计原则中的“最小权限原则”。在软件设计中，例如一个对象的API，这个原则表现为向外暴露最少量的东西以满足需求，然后隐藏其他所有的东西。

这个原则扩展了用哪个作用域来包含变量和函数。如果所有的变量和函数都在全局作用域中，它们当然可以被所有的嵌套作用域访问。但这违背了“最小...”原则。这暴露了很多本该私有化的变量和方法，这种完全的用法会让变量/方法的访问变得不美。

例如:

```js
function doSomething(a) {
	b = a + doSomethingElse( a * 2 );

	console.log( b * 3 );
}

function doSomethingElse(a) {
	return a - 1;
}

var b;

doSomething( 2 ); // 15
```
在这段代码中 `b` 变量和 `doSomethingElse(..)` 方法从他们所做的事来看应该是“私有的”。把他们的入口暴露在外不但是不必要的，而且可能会是“危险”的，他们可能被一些不希望的方式有意无意地调用， 而且还可能打乱`doSomething(..)`执行的前提条件。

把这些私密的细节隐藏在`doSomething(..)`的作用域中，将会是个更恰当的设计：

```js
function doSomething(a) {
	function doSomethingElse(a) {
		return a - 1;
	}

	var b;

	b = a + doSomethingElse( a * 2 );

	console.log( (b * 3) );
}

doSomething( 2 ); // 15
```
现在，`b` 和 `doSomethingElse(..)` 无法从外部访问了,取而代之的是只能对 `doSomething(..)`的控制。这样做对功能性和结果并没有什么影响，但这种设计把细节私有化，这是一种更好的软件实践。

### 冲突避免

隐藏变量和方法的另一个好处就是可以避免两个同名标识符的冲突。这种冲突常常是因为意外覆盖了值引起的。

例如：

```js
function foo() {
	function bar(a) {
		i = 3; // 改变了for循环中 `i` 的值
		console.log( a + i );
	}

	for (var i=0; i<10; i++) {
		bar( i * 2 ); // oops, 死循环!
	}
}

foo();
```
在 `bar(..)`中`i = 3`的赋值意外覆盖了`foo(..)`中for循环里声明的`i`。此时，引发了一个死循环，因为`i`被赋予了一个固定值 `3` ，永远小于10。

`bar(..)`里的赋值需要声明一个本地变量，无所谓标识符叫什么名字。 `var i = 3;`能解决这个问题（会为`i`创建一个我们前面提到的“阴影变量”声明）。

#### 全局命名空间

在你程序中引入的多个类库如果不完全隐藏它们的内部/私有的方法和变量，和容易引起冲突。

这些类库通常会创建一个单独的变量声明，经常是一个在全局作用域中拥有唯一名字的对象。这个对象会运用是一个属于这个类库自己的“命名空间”，所有对外暴露的函数都表现为对象（命名空间）的属性，而不是顶层作用域中的标识符。

例如:

```js
var MyReallyCoolLibrary = {
	awesome: "stuff",
	doSomething: function() {
		// ...
	},
	doAnotherThing: function() {
		// ...
	}
};
```

#### 模块管理

另一种避免冲突的方法就是模块化，运用一些依赖管理器。通过运用这些工具，就再也不会有类库向全局作用域添加标识符了，但需要通过依赖管理器提供的机制指定引入标识符到具体的作用域中。

你可能会注意到这些工具并没有什么可以摆脱词法作用域规则的魔法。它们只是运用了作用域规则来强制了标识符不能出现在共享的作用域中，保持标识符私有化，杜绝冲突污染的作用域，以此确保没有意外的作用域冲突。

因此，你自己也可以写出像模块管理器一样的代码，如果你要这么做。在第五章的模块模式中会有更多信息。

## 作用域函数

我们已经知道可以通过用函数包裹代码的方式有效地隐藏变量和方法。

例如:

```js
var a = 2;

function foo() { // <-- insert this

	var a = 3;
	console.log( a ); // 3

} // <-- and this
foo(); // <-- and this

console.log( a ); // 2
```
虽然这在技术上可行，但却不是一个好主意。这里有一些小问题。首先我们要声明一个名叫`foo()`的方法，这意味着叫做 `foo` 的标识符污染了它自己的作用域（这个例子中是全局作用域）。而且我们要明确地通过名字（`foo()`）来调用这个方法。

试想一下如果这个方法不需要名字（或者名字不会污染作用域），而且它可以自动执行的话会怎么样呢。

幸运的是，JavaScript为这个问题提供了解决方案。

```js
var a = 2;

(function foo(){ // <-- insert this

	var a = 3;
	console.log( a ); // 3

})(); // <-- and this

console.log( a ); // 2
```

让我们看看这都发生了什么。

首先，要注意的是包装函数表达式是以 `(function...`开始的而不是 `function...`。这个看似微小的细节，实际上有很大的不同。相较于标准的函数声明，这个函数被当做一个函数表达式来对待。

**注释:**一个区别声明和表达式的最简单方法就是"function"单词在语句中的位置。如果"function"在语句最前端，那他就是函数声明。不然就是一个函数表达式。

我们可以发现函数声明和函数表达式的主要不同在于他们的名字在何处绑定为标识符。

对比前面的两段代码。第一段中的`foo`在环绕作用域中绑定。第二段中，`foo`不在环绕作用域中绑定，而是在他自己内部的方法中绑定。

换句话说，`(function foo(){ .. })` 是一个表达式意味着`foo`标识符 *只能*在`..`的内部被找到，而不能在外部作用域中找到。 

In other words, `(function foo(){ .. })` as an expression means the identifier `foo` is found *only* in the scope where the `..`。 把`foo`的名字隐藏在它自己内部意味着它不会无故的污染环绕作用域。

### 匿名 vs. 声明的

你可能最熟悉把函数表达式作为一个回调参数了吧，就像：

```js
setTimeout( function(){
	console.log("I waited 1 second!");
}, 1000 );
```

这被叫做“匿名函数表达式”，因为 `function()...`没有他自己的标识符。函数表达式可以是匿名的，但方法声明不能漏掉名字，因为那样会引起JS语法错误。

匿名函数表达式很简单，很多类库和工具都鼓励使用这种惯用的编码风格。
尽管它们需要考虑几个问题：

1. 匿名函数没有在追踪栈里留下有用的名字，这增大了调试难度。

2. 没有名字的情况下，当函数需要引用自己（诸如递归）时，就要不幸的用到`arguments.callee`这种已经**不推荐**的引用了。另一个函数需要自我引用的例子就是一个事件处理函数需要在触发之后解除绑定。

3. 匿名函数删掉了有利于代码可读性的名字。

**行内函数表达式**是强大和有用的————但依然也引出很多问题，为你的函数表达式起个名字就能很好的解决。所以始终给你的函数表达式命名是一个最佳的实践：

```js
setTimeout( function timeoutHandler(){ // <-- 瞧，我有一个名字!
	console.log( "I waited 1 second!" );
}, 1000 );
```

### 立即执行函数表达式

```js
var a = 2;

(function foo(){

	var a = 3;
	console.log( a ); // 3

})();

console.log( a ); // 2
```

现在我们为函数加上一对括号使之成为函数表达式，我们可以通过在它后面在加上`()` 使函数执行，就像`(function foo(){ .. })()`。第一对括号生成一个函数表达式，第二对括号使函数执行。

只是一个很普通的模式，几年前在业界就为此有了一个术语：**IIFE**，代表了**I**mmediately **I**nvoked **F**unction **E**xpression（立即执行函数表达式）。

当然，IIFE不一定需要名字，IIFE的通常格式就是用一个匿名函数表达式。虽然不是通常用法，但为IIFE命名也拥有上面所有匿名函数表达式的好处，所以不失为一个好的选择实践。

```js
var a = 2;

(function IIFE(){

	var a = 3;
	console.log( a ); // 3

})();

console.log( a ); // 2
```
这与通常的IIFE格式（`(function(){ .. }())`）相比有一些小差异。仔细看看有什么不同。第一个格式中，函数表达式被包裹在 `( )`中，然后执行括号对在它的外面。第二个格式中，执行括号对被移动到了包裹括号对里面。

这两种格式在功能上都是相同的。**用哪一种完全出于你的格式喜好。**

IIFE另一个在实际中更常用的方式就是调用函数，并给他传入参数。

例如:

```js
var a = 2;

(function IIFE( global ){

	var a = 3;
	console.log( a ); // 3
	console.log( global.a ); // 2

})( window );

console.log( a ); // 2
```

我们传入了`window`对象的引用，但我们给参数命名为`global`，从而在文法上相较于非全局引用更清晰，你可以传入在包裹作用域的任何东西，然后为参数来一个恰当的命名。这更是一种文法风格的选择。

这个模式的另一个应用就是防止因`undefined`标识符被不正确的重写而引起的意外结果。通过把参数命名为`undefined`而不传入参数，我们就能确保`undefined`标识符确实是undefined的值：

```js
undefined = true; // setting a land-mine for other code! avoid!

(function IIFE( undefined ){

	var a;
	if (a === undefined) {
		console.log( "Undefined is safe here!" );
	}

})();
```
还有一种用法就是把函数作为参数传入。这种模式被应用在UMD（Universal Module Definition）项目中。有的人会觉得这样做更清晰和利用理解，尽管它有一些冗长：

```js
var a = 2;

(function IIFE( def ){
	def( window );
})(function def( global ){

	var a = 3;
	console.log( a ); // 3
	console.log( global.a ); // 2

});
```
`def`函数表达式在代码段的后半部分，被作为一个参数传入定义在代码段前半部分的`IIFE`函数。最后，参数`def`（函数）被执行，带着传入的参数`global`（`window`）。

## 块级作用域

函数是最普通的作用域单元，当然，在广泛传播的JS设计方法中，还有别的作用域单元，运用这些作用域单元，可以让代码变得更加清晰可维护。

很多JavaScript之外的语言都支持块级作用域，所以其他语言的开发者们对这个概念就有思维定势，而以JavaScript为主的开发者对这个概念可能会有点陌生。

但就算你一次也没写过块级作用域，你对JavaScript这段代码也肯定很熟悉。

```js
for (var i=0; i<10; i++) {
	console.log( i );
}
```
我们直接在循环的开头声明了变量 `i`，我们很可能只希望在这个循环中使用 `i`，但忽略了这个变量存在于包裹作用域（函数或全局对象）中。

这就是块级作用域。尽量在使用时就近声明变量。
另一个例子:

```js
var foo = true;

if (foo) {
	var bar = foo * 2;
	bar = something( bar );
	console.log( bar );
}
```

我们只在if代码中用到了变量`bar`，这让人意识到我们应该在if块中声明`bar`。尽管我们在哪用 `var`来声明变量都没关系，因为他们都属于包裹作用域。这段代码实际上是一个“假的”块级作用域，应该自我强制不要在作用域的其他地方用到`bar`。

块级作用域是一个扩展前面提到“最小权限暴露原则”的工具，通过把在函数中隐藏信息变为在代码块中隐藏信息。

再看看前面的循环例子：

```js
for (var i=0; i<10; i++) {
	console.log( i );
}
```
为什么只在循环中使用的变量`i`会污染整个函数作用域？

但更重要的是，开发者们喜欢靠自己检查来排查不在他们意图范围内的变量使用，例如在一个错误的地方使用一个未知的变量会抛出一个错误。块级作用域（如果有的话）可以让变量 `i`只在循环内有效，在函数的其他地方使用 `i`会引发错误。这样能确保变量不在其他不好排查的地方重复使用。

但是，令人伤心的事实是，表面上，JavaScript没有块级作用域场景。

除非，你更深入一点。

### `with`

我们在第二章学过`with`。尽管它是个令人皱眉的结构，但它确实是一个块级作用域的例子，作用域在`with`块中，而不在包裹作用域中。

### `try/catch`

这是个鲜为人知的事实，在ES3版本的JavaScript中的`try/catch`，`catch`块是一个块级作用域。

例如:

```js
try {
	undefined(); // 强制引起异常的非法操作!
}
catch (err) {
	console.log( err ); // 起作用了!
}

console.log( err ); // ReferenceError: `err` not found
```

如你所见， `err`只存在 `catch`分句子中，当你在别处引用它时会抛出异常。

### `let`
目前为止，我们在探索块级作用域的过程中看到了很多怪异的行为。如果只是这些的话，块级作用域对JavaScript开发者就没什么大用了。

幸运的是，ES6改变了这一切，它引入了`let`关键字，是`var`之外的另一种声明变量的方式。

`let`关键字声明的变量是块级作用域有效的。

```js
var foo = true;

if (foo) {
	let bar = foo * 2;
	bar = something( bar );
	console.log( bar );
}

console.log( bar ); // ReferenceError
```
用`let`来使变量依附块级作用域多少有些含蓄。当你没有太在意变量属于哪个作用域是可能会引起困惑，用新的块把它们包裹起来是一个进化你新代码的好习惯。

为块级作用域创建明确的块可以打消这些担忧:

```js
var foo = true;

if (foo) {
	{ // <-- explicit block
		let bar = foo * 2;
		bar = something( bar );
		console.log( bar );
	}
}

console.log( bar ); // ReferenceError
```
我们只需简单创建一对大括号来绑定`let`并且这是语法可行的。在这个例子中，我们在if分句中创造了一个明确的块，这样做既简单有不会影响if分句的语义。

**注释:** 另一种指定块级作用域的方法，见附录B。

在第四章中，我们要讨论提升，在声明所处的作用域中，处处可以得到它。

但是，用`let`的声明不会提升到整个块中。这种声明直到执行到声明语句时才会被认为是存在的。

```js
{
   console.log( bar ); // ReferenceError!
   let bar = 2;
}
```

#### 垃圾回收

块级作用域有用的另一个原因是对于闭包和垃圾回收的。我们只在这里简要提及，闭包机制我们会在第五章详细介绍。

且看:

```js
function process(data) {
	// do something interesting
}

var someReallyBigData = { .. };

process( someReallyBigData );

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
	console.log("button clicked");
}, /*capturingPhase=*/false );
```

`click`函数的点击处理回调完全不需要`someReallyBigData`变量。这就是说，理论上`process(..)`执行之后，这个占用很多内存的数据结构就可以被当做垃圾回收了。但是，JS引擎似乎（取决于引擎具体实现）还保存着这个数据。

块级作用域可以解决这个问题，只需告诉引擎 `someReallyBigData`不需要再保存了：

```js
function process(data) {
	// do something interesting
}

// 任何在块中声明的东西之后就可以滚了!
{
	let someReallyBigData = { .. };

	process( someReallyBigData );
}

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
	console.log("button clicked");
}, /*capturingPhase=*/false );
```

为变量声明明确的块用来本地绑定是一个强大的工具，足以加入你的代码工具箱了。

#### `let` 循环

`let`闪耀的另一个场景就是在前面讨论过的for循环例子中。

```js
for (let i=0; i<10; i++) {
	console.log( i );
}

console.log( i ); // ReferenceError
```

在for循环头部的`let`不仅把`i`绑定到了循环体中，事实上，它还在每次循环迭代中重建了变量`i`，以确保`i`被重新赋予前面迭代结果的值。

这是另一种每次迭代绑定行为的例子：

```js
{
	let j;
	for (j=0; j<10; j++) {
		let i = j; // re-bound for each iteration!
		console.log( i );
	}
}
```

每次迭代绑定为什么有趣的原因会在第五章讨论闭包时水落石出。

因为`let`声明依附于任意块而非闭合函数作用域，所以不难理解用`var`声明的隐藏代码需要依赖函数作用域，并且在重构代码时用 `let`代替 `var`要多加留意。

且看:

```js
var foo = true, baz = 10;

if (foo) {
	var bar = 3;

	if (baz > bar) {
		console.log( baz );
	}

	// ...
}
```

很容易就能把这段代码重构为:

```js
var foo = true, baz = 10;

if (foo) {
	var bar = 3;

	// ...
}

if (baz > bar) {
	console.log( baz );
}
```

但是，针对块级作用域变量这么做是就要多加小心了：

```js
var foo = true, baz = 10;

if (foo) {
	let bar = 3;

	if (baz > bar) { // <-- 移动时别忘了把`bar`也带上!
		console.log( baz );
	}
}
```

附录B中可以看到一种代替（更明确）的块级作用域代码风格，针对这种情况更健壮，更容易维护和重构。

### `const`

作为`let`的补充，ES6引入了`const`关键字，它也能创建块级作用域变量，但它的值是固定的。任何试图改变`const`值得操作都会引起系统报错。

```js
var foo = true;

if (foo) {
	var a = 2;
	const b = 3; // 存在于if的块级作用域中

	a = 3; // 没问题!
	b = 4; // 报错!
}

console.log( a ); // 3
console.log( b ); // ReferenceError!
```

## 总结

函数是JavaScript中最普通的作用域单元。在函数中声明的变量和函数会被对外“隐藏”起来，这是一种很好的软件设计原则。

但函数不是唯一的作用域单元。块级作用域就是用块（通常为 `{ .. }`对）包裹代码，而不单单是包裹函数。

从ES3开始，`try/catch`结构的`catch`就形成了一个块级作用域。

在ES6中，`let`关键字（`var`关键字的表亲），就被引入用来在任何代码块中声明变量。 `if (..) { let a = 2; }`声明了一个变量 `a`，它实际上劫持了`if`的 `{ .. }`块并且把自己依附在上面。

块级作用域不应该取代函数作域。两种功能应该并存，开发者们应该用好函数作用域和块级作用域在各自适合的场景去生产更好，更可读可维护的代码。
