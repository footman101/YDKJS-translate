> 原文地址：[https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20%26%20closures/ch5.md](https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20%26%20closures/ch5.md)

# 你不知道的JS：*this* & 对象原型
# 第一章：`this`还是That?

`this`关键字是JavaScript中最令人困惑的机制之一。它是自动定义在每一个函数作用域中的特殊标识符，但是它到底指向啥依旧困惑着有经验的JavaScript的开发人员。

> 任何足够*先进的*技术都与魔术无法区分。-- Arthur C. Clarke

JavaScript的`this`机制实际上并不*那么*先进，但开发人员通常会通过插入“复杂”或“令人困惑”而在自己的脑海中引用这句话，毫无疑问，如果缺乏明确的理解，在*你的*困惑中，`this`可以看起来是彻头彻尾的魔术。

**注意：**“this”这个词在一般话语中是一个非常普遍的代名词。因此，确定我们是否使用“this”作为代词或使用它来引用实际的关键字标识符可能非常困难，尤其是口头上。为清楚起见，我将始终使用`this`来引用特殊关键字，并其他情况下使用“this”或*this*或this。

## 为什么`this`?

如果`this`是如此令人困惑，甚至是有经验的JavaScript开发人员，那么人们就会疑惑为什么它是有用的？它是麻烦更多吗？在我们跳入*怎样*之前，我们应该检查*为什么*。

让我们试着说明`this`的动机和效用：

```js
function identify() {
	return this.name.toUpperCase();
}

function speak() {
	var greeting = "Hello, I'm " + identify.call( this );
	console.log( greeting );
}

var me = {
	name: "Kyle"
};

var you = {
	name: "Reader"
};

identify.call( me ); // KYLE
identify.call( you ); // READER

speak.call( me ); // Hello, I'm KYLE
speak.call( you ); // Hello, I'm READER
```

如果这段代码*怎样*工作让你困惑，不要担心！我们很快就会弄清楚。先把这些问题放一边，这样我么就可以把*为什么*弄得更清楚。

这段代码可以让`identify()`和`speak()`函数在多个*上下文*（`me`和`you`）对象中重复使用，而不是每个对象需要一个独立版本的函数。

除了依赖`this`，你可以显式地将一个上下文对象传入`identify()`和`speak()`。

```js
function identify(context) {
	return context.name.toUpperCase();
}

function speak(context) {
	var greeting = "Hello, I'm " + identify( context );
	console.log( greeting );
}

identify( you ); // READER
speak( me ); // Hello, I'm KYLE
```

然而，`this`机制提供了一个更优雅的隐式『传递』对象引用的方式，这样API审计更加简洁，更加便于重用。

你的使用场景越复杂，你就会越清楚地看到，作为显式参数传递上下文通常比传递`this`上下文更加混乱。当我们探索对象和原型时，你将看到一组函数能够自动引用正确的上下文对象。

## 困惑

我们很快就会开始解释`this`*实际上*是如何工作的，但首先我们必须消除一些关于它实际上*不*是如何工作的误解。

如果开发者试图通过字面意思来理解，名称『this』会产生误解。有两种常见的猜测的意思，但是两种都是错的。

### 它自己

第一个常见的陷阱是假设`this`指的是函数本身。至少，这是一个合理的语法推断。

你为什么要在一个函数内部引用自身？最常见的原因就是比如递归（在函数内部调用自己）或者在一个事件处理函数内部地第一次调用后解绑定自己。

新接触JS机制的开发人员通常认为将函数作为对象引用（JavaScript中的所有函数都是对象！）允许你在函数调用之间存储*状态*（属性中的值）。虽然这当然是可能的并且具有一些有限的用途，但本书的其余部分将阐述许多其他模式，以便在函数对象之外*更好的*存储状态。

但是暂时，我们将探索这种模式，以说明`this`并没有让函数像我们假设的那样获得对自身的引用。

考虑以下代码，我们尝试跟踪调用函数（`foo`）的次数：

```js
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	this.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 0 -- WTF?
```

`foo.count`*依旧*是`0`，即使四次`console.log`明确显示`foo(..)`被调用了四次。没有成功的原因是*过于字面化*地解释`this`（`this.count++`中）的含义。

当这段代码执行`foo.count = 0`时，实际上给函数对象`foo`增加了一个`count`属性。但是对于函数内部的`this.count`引用，`this`实际上*并不是*函数对象，因此，即使属性名称相同，根对象却是不同的，由此会产生混淆。

**注意：**一个负责任的开发者*应该*追问这一点，『如果我没有在预期的对象上增加`count`的值，那么我*是*给哪一个对象的`count`增加了值？』实际上，如果她更深入一些，就会发现她意外地创建了一个全局变量`count`（这是*怎样*发生的见第二章！），然后这个变量当前值是`NaN`。当然，一旦她确定了这种特殊的结果，她就会有另外一套问题：“它是如何成为全局变量的，为什么它最终会是`NaN`而不是一些适当的计数值？” （见第2章）。

许多开发人员会绕开这个问题，并且提出其他一些解决方案，比如创建另一个对象来保存`count`属性，而不是停留在这一点并深入研究为什么`this`引用似乎没有表现如*预期*，并回答这些棘手但重要的问题，：


```js
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	data.count++;
}

var data = {
	count: 0
};

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( data.count ); // 4
```

这样确实解决了问题，不幸的是这只是忽略了问题 -- 没有理解`this`的含义和工作机制 -- 而是在舒适区找到了一个熟悉的机制：词法作用域。

**注意：**词法作用域是一个非常好并且有用的机制；无论如何，我并不是在贬低它的使用（见本系列书的*作用域与闭包*）。但是总是*猜错*如何使用`this`，并不是一个退回到词法作用域并且永远不会学习*为什么*掌握不了`this`的理由。

要在一个函数内部引用它自身，`this`自身一般是不能满足的。你通常需要指向函数的词法标识符（变量）来引用函数对象。

考虑这两个函数：

```js
function foo() {
	foo.count = 4; // `foo` refers to itself
}

setTimeout( function(){
	// anonymous function (no name), cannot
	// refer to itself
}, 10 );
```

第一个函数，成为『命名函数』，`foo`就是可以被用来在函数内部引用函数自身的引用。

但是在第二个例子中，传入`setTimeout(..)`的回调函数并没有被命名（称为『匿名函数』），因此没有通常的办法引用函数对象自身。

**注意：**之前使用但现在已经弃用的`arguments.callee`引用*也*指向当前正在执行函数的函数对象。这个引用通常是从内部访问匿名函数对象的唯一方法。然而，最好是使用命名函数（表达式），而完全避免使用匿名函数，至少是对于那些需要自引用的函数来说。`arguments.callee`已弃用，不应使用。

因此，我们运行示例的另一个解决方案是使用`foo`标识符作为函数对象的引用，而不使用`this`，这种办法是*有效的*：

```js
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	foo.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 4
```

然而，这种办法似乎回避了对`this`*真正*的理解，并彻底依赖了词法作用域的`foo`变量。

另外一种解决办法是将`this`强制指向`foo`函数对象。

```js
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	// Note: `this` IS actually `foo` now, based on
	// how `foo` is called (see below)
	this.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		// using `call(..)`, we ensure the `this`
		// points at the function object (`foo`) itself
		foo.call( foo, i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 4
```

**我们拥抱使用`this`，而不是回避它。**我们将稍微更完善地解释一下这些技术*如何*工作，所以如果你仍然有点困惑，不要担心！

### 它的作用域

接下来的一个常见的对于`this`含义的误解是，它指向了函数的作用域。这是一个棘手的问题，因为这种说法在某种意义上说有些对，但从另一个意义上说，它是相当误导的。

要清楚，`this`不以任何方式指向函数的**词法作用域**。确实，在内部，作用域有点像是一个具有每个可用标识符属性的对象。但JavaScript代码无法访问这个作用域“对象”。它是*引擎*内部实现的一部分。
考虑这一段尝试（并且失败！）跨越边界并使用`this`隐式引用函数的词法作用域的代码：

```js
function foo() {
	var a = 2;
	this.bar();
}

function bar() {
	console.log( this.a );
}

foo(); //undefined
```

这段代码有不止一个错误。虽然它可能看似做作，但你看到的代码是在公共社区帮助论坛中交换的实际现实代码的片段。这是一个很好的（如果不是悲伤的）例证，`this`的假设是多么误导。

首先，通过`this.bar()`尝试引用`bar()`函数。这只是*偶然地*可以工作，我们会稍后简要介绍解释它*怎样*工作。调用`bar()`最自然的方式的方式去掉前导的`this.`，只是使用词法作用域中的引用即可。

然而，写这段代码的开发者尝试使用`this`来作为`foo()`和`bar()`作用域的桥，这样`bar()`就可以访问`foo()`内部作用域的变量`a`。**这种桥是不可能的。**你不能使用`this`引用来查找词法作用域中的变量，这是不可能的。

每一次你感觉你自己混淆了词法作用域查找和`this`，提醒你自己：*没有这样的桥。*

## 什么是`this`?

抛开各种不正确的假设后，让我们现在把注意力转向`this`机制如何运作。

我们之前说过，`this`不是书写时绑定，而是运行时绑定。它是会基于函数调用条件的上下文。`this`绑定与声明函数的位置无关，而是与调用函数的方式有关。

调用函数时，会创建激活记录，也称为执行上下文。此记录包含有关函数调用位置的信息（调用堆栈），调用函数的*方式*，传递的参数等。此记录的一个属性就是`this`引用，它在该函数的执行期间可以使用。

在下一章中，我们将学习如何找到一个函数的**调用点(call-site)**来确定它的执行将如何绑定`this`。

## 回顾 (TL;DR)

对于没有花时间学习机制实际工作原理的JavaScript开发人员来说，`this`绑定是一个常见的混乱来源。猜测，是错，来自Stack Overflow答案的复制粘贴不是利用*这个*重要的`this`机制的有效或正确方法。

要学习`this`，你首先得知道`this`*不是*什么，尽管有很多假设或误解可能会导致你走上这些道路。`this`既不是函数本身的引用，也不是函数*词法*作用域的引用。

`this`实际上是一个在函数调用时的绑定，并且它指向*什么*完全取决于函数调用时的调用点(call-site)。
