> 原文地址：[https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20%26%20closures/ch5.md](https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20%26%20closures/ch5.md)

# 你不知道的JS：作用域与闭包
# 第五章：作用域闭包

我们对于作用域工作原理有了一个非常健康坚实的理解，充满了希望来到了这里。

我们要把注意力集中到一个非常重要，但是持续地难以捉摸，*几乎有些神话色彩*，的语言一部分：**闭包**。如果你到目前为止一直跟随着我们关于词法作用域的讨论，回报就是闭包即将大体上，虎头蛇尾地，几乎不言自明了。*有一个人在巫师的窗帘后边，我们就要看到他了*。不，它的名字不是克劳福德！

然而如果你仍然对词法作用域有些问题，你最好在往下进行之前复习一下第二章。

## 启迪

有些人对于JavaScript有些经验，但是可能从未真正理解闭包的概念，*理解闭包*就像一次特殊的涅磐，人需要努力和付出才能实现。

我想起了几年前，当我对于JavaScript有了坚实的理解，但是依旧不懂闭包是什么。这个语言的*另一面*暗示，那个承诺比我已经拥有更多能力的人，打趣我，嘲弄我。我记得阅读早期框架的源代码来了解它是如何工作。我记得第一次有一些“模块模式”出现在我脑海里。我十分生动地记得*a-ha!*时刻。

**我曾经并不知道的**，花了我很多年来懂得，但是我希望现在传达给你的一个秘密是：**在JavaScript中，闭包无处不在，你只需要认识和拥抱它**。闭包不是一个特殊的，你必须学习新的语法和模式的选择工具。不，闭包甚至不是一件就像原力中Luke训练使用的，你必须学来掌握的武器。

闭包出现在依赖词法作用域的书写代码结果中。它们只是发生了。你甚至不需要有意地创建闭包来利用它们。闭包在你的代码中随处可以创建和使用。你真正*缺少*的是，随心所欲地认识、接受和利用闭包的适当的心理背景。

启迪的时刻应该是：**噢，闭包已经出现在我的代码中，我最终可以*看到*它们了**。理解闭包就像是Neo第一次看见Matrix。（黑客帝国？）

## 事实真相

OK，已经有足够多夸张和无耻的电影引用了。

这里有一个关于理解和识别闭包需要知道的定义：

> 闭包是，即使一个函数在它的词法作用域外执行，它依旧可以记住并访问它的词法作用域。

让我们看一段代码来解释这个定义。

```js
function foo() {
	var a = 2;

	function bar() {
		console.log( a ); // 2
	}

	bar();
}

foo();
```

我们在讨论嵌套作用域时对这段代码应该十分熟悉。由于词法作用域的查找规则（这个例子中，这是一个RHS引用查找），函数`bar()`可以在其外层包裹作用域中*访问*变量`a`。

这是『闭包』吗？

呃，从技术上说，*也许吧*。但是从我们上面提到的你必须知道的定义来讲，*并不确切*。我认为解释`bar()`引用`a`的准确方式是通过词法作用域的查找规则，并且这些规则*只是*闭包（重要的）**一部分**。

从一个纯粹的学院派视角，上面这个代码片段是说，函数`bar()`在作用域`foo()`（实际上，也包括其他可以访问的作用域，在这个情况中比如全局作用域）上有一个*闭包*。换一个说法就是，`bar()`可以完全地访问`foo()`作用域。为什么？因为`bar()`干净利落地嵌套出现在`foo()`内部。

但是，这样的闭包并不是直接*可见的*，我们也没有看到闭包在那段代码中有*运用*。我们可以清晰地看到词法作用域，但是闭包依旧在这段代码的阴影中，保留了一些神秘感。

让我们考虑这段完全暴露闭包的代码：

```js
function foo() {
	var a = 2;

	function bar() {
		console.log( a );
	}

	return bar;
}

var baz = foo();

baz(); // 2 -- Whoa, closure was just observed, man.
```

函数`bar()`可以访问`foo()`内部作用域。但是，我们单独拿出函数`bar()`本身，将其*保存为*一个值。在这个例子中，我们*返回*了函数对象本身`bar`引用。

在我们执行完`foo()`后，我们将它的返回值（我们内部的`bar()`函数）赋予了一个叫做`baz`的变量，然后我们调用`baz`，即调用了我们的内部函数`bar()`的一个不同的标识符引用。

`bar()`确实是被执行了。但是在这个例子中，它在它被定义的词法作用域*之外*被执行了。

在`foo()`执行后，正常情况下，我们会期望`foo()`的整个内层作用域会被释放，因为我们知道*引擎*使用了*垃圾回收器*释放它不再使用的内存。由于`foo()`的资源不会再被使用，我们很自然地认为它们应该被*释放*。

但是闭包的『魔法』没有让这些发生。这个内部作用域实际上*依旧*被『使用』，并没有释放。谁在用？**函

`bar()`由于在那里定义的缘故，在`foo()`的内部作用域上有一个词法作用域闭包，这会使这个作用域为了`bar()`而保持存活，这样就可以在之后引用它了。

**`bar()`仍然对作用域有一个引用，这个引用被称为闭包。**

所以，几毫秒之后，当变量`baz`被调用（调用在内层函数，我们初始化成为`bar`），它的确可以*访问*书写时词法作用域，因此正如我们期望，它可以访问变量`a`。

这个函数在它的书写时词法作用域之外被调用，可以运行地很好。**闭包**让这个函数可以继续访问书写时词法作用域。

当然，函数可以作为值*传递*,，并且在其他位置调用可能有多种形式，这其中的任何一种都是观察/执行闭包的示例。

```js
function foo() {
	var a = 2;

	function baz() {
		console.log( a ); // 2
	}

	bar( baz );
}

function bar(fn) {
	fn(); // look ma, I saw closure!
}
```

我们将内部函数`baz`传递给`bar`，然后调用了内部函数（现在标记为`fn`），我们一旦这样做，它在内层`foo()`作用域的闭包会被观察到，可以访问到`a`。

这种函数传递也可以非直接传递。

```js
var fn;

function foo() {
	var a = 2;

	function baz() {
		console.log( a );
	}

	fn = baz; // assign `baz` to global variable
}

function bar() {
	fn(); // look ma, I saw closure!
}

foo();

bar(); // 2
```

无论我们使用什么方法来*传输*在词法范围之外的内部函数，它都会保持对最初声明作用域的引用，并且无论我们在何处执行它，都将执行该闭包。

## 现在我可以看到

之前的代码片段在某种程度上是学院派的，并且是人工构造的，来说明*如何使用闭包*。但我向你承诺的不仅仅是一个很酷的新玩具。我承诺在你现有的代码中，闭包是随处可见的东西。现在让我们*看看*这个事实。

```js
function wait(message) {

	setTimeout( function timer(){
		console.log( message );
	}, 1000 );

}

wait( "Hello, closure!" );
```

我们将一个内部函数（名为`timer`）传递给`setTimeout(..)`。但是`timer`在`wait(..)`的作用域内有一个作用域闭包，可以保留并使用对变量`message`的引用。

我们执行过`wait(..)`1000毫秒之后，它的内部作用域并没有被释放，内部函数`timer`仍然在该作用域内有闭包。

在*Engine*的内部深处，内置程序`setTimeout(..)`引用了一些参数，可能称为`fn`或`func`或其他东西。*Engine*调用该函数，即是调用我们的内部`timer`函数，并且词法作用域引用仍然可以访问。

**闭包.**

或者，如果你对jQuery比较熟（或任何JS框架，就此而言）：

```js
function setupBot(name,selector) {
	$( selector ).click( function activator(){
		console.log( "Activating: " + name );
	} );
}

setupBot( "Closure Bot 1", "#bot_1" );
setupBot( "Closure Bot 2", "#bot_2" );
```

我不确定你写的是什么样的代码，但我经常编写代码，负责控制整个全局无人机闭包机器人军队，所以这是完全现实的！

（有些）开玩笑，基本上*无论何时*以及*无论何地*你将函数（可以访问它们各自的词法作用域）视为一等值并传递它们时，你可能会看到这些函数在执行闭包。当你传入一个*回调函数*时，比如定时器，事件处理，Ajax请求，跨窗口消息传递，web workers或任何其他异步（或同步！）任务，准备带上一些闭包！

**注意：** 第三章介绍了IIFE模式。虽然人们经常说IIFE（单独）是观察到的闭包的一个例子，但根据上面的定义，我对此有点不同意。

```js
var a = 2;

(function IIFE(){
	console.log( a );
})();
```

这段代码『可以运行』，但是这不是严格意义上的闭包。为什么？因为函数（我们命名为"IIFE"）并不是在它的词法作用域意外运行。它仍然在它定义的作用域（包裹/全局作用域，同时拥有`a`）中被调用。`a`可以通过正常的词法作用域查找被找到，并不是真正通过闭包。

虽然关闭技术上可能在定义时发生，但它*不是*严格可观察的，因此，正如他们所说，*它是一棵落在森林里的树，周围没有人听过它。*

虽然IIFE本身并不是闭包的一个例子，但它确实创造了作用域，它是我们用来创建可以关闭的作用域的最常用工具之一。 因此，即使自身不是闭包，IIFE确实与闭包密切相关。

亲爱的读者，请立即放下这个本书。我有一个任务给你。打开一些最近的JavaScript代码。寻找你的函数作为值，并确定你已经在使用闭包的地方，甚至可能以前都不知道的地方。

我会等着。

现在...你看到了！

## 循环 + 闭包

用于说明闭包的最常见的规范示例是简单的for循环。

```js
for (var i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

**注意：** 当你把函数放在循环中时，Linters经常会提示错误，因为不了解闭包的错误**在开发人员中很常见**。我们在这里解释如何正确地做到这一点，充分利用闭包的全部功能。但Linters通常会丢失这种微妙之处，他们会不顾一切地提示错误，认为你*实际上*不知道你在做什么。

这段代码的含义是，我们通常会*期望*它的行为是数字“1”，“2”，......“5”将被打印出来，一次打印一次，每秒打印一次。

实际上，如果你运行这段代码，你会得到"6"被打印了5次，每秒打印一次。

**呵?**

首先，我们来解释`6`从哪里产生。循环的最终条件是`i`*不*`<=5`。第一次达到这个情况是`i`为6。因此，这个输出反应了循环结束后`i`的最终值。

这实际上在第二眼看上去很明显。循环完成后，超时函数回调都运行良好。实际上，随着定时器的推移，即使在每次迭代时它都是`setTimeout（..，0）`，所有这些函数回调仍然会严格在循环完成后运行，因此每次都打印`6`。

但是这里有一个更深层次的问题。 我们的代码中*缺少*什么来实际让它表现得像我们在语义上暗示的那样？

缺少的是我们试图*暗示*循环的每次迭代“捕获”它自己的`i`副本。但是，作用域发生作用，这些函数中的所有5个，虽然它们是在每个循环迭代中单独定义，但**都在相同的共享全局作用域上关闭**，实际上只有一个`i`在其中。

就这样，所有函数*理所当然*共享对同一个`i`的引用。关于循环结构的一些东西往往会让我们误以为在工作中还有更复杂的东西。 其实并没有。这与没有任何循环时，5个超时回调函数，一个接一个地声明是一样的。

好的，那么，回到我们棘手的问题。是什么丢失了？我们需要更多的闭包作用域。具体来说，我们需要为循环的每次迭代提供一个新的闭包作用域。

我们在第三章学过，IIFE可以通过定义一个函数并马上调用它来创造作用域。

我们来试试：

```js
for (var i=1; i<=5; i++) {
	(function(){
		setTimeout( function timer(){
			console.log( i );
		}, i*1000 );
	})();
}
```

这个行吗？试试看。我会继续等你。

我会打消你的疑虑。**并不能。**但是为什么？我们现在显然拥有了更多的词法作用域。每一个超时回调函数确实在它自己的每次迭代作用域中闭合，这些作用分别由每个IIFE创建

*如果该作用域为空*，就没啥用了。仔细看。我们的IIFE只是一个空的无所事事的作用域。它需要有一些*东西*才能对我们有用。

它需要属于它自己的变量，即每次迭代`i`值的副本。

```js
for (var i=1; i<=5; i++) {
	(function(){
		var j = i;
		setTimeout( function timer(){
			console.log( j );
		}, j*1000 );
	})();
}
```

**找到了！有用！**

更好的一种写法是：

```js
for (var i=1; i<=5; i++) {
	(function(j){
		setTimeout( function timer(){
			console.log( j );
		}, j*1000 );
	})( i );
}
```

理所当然，由于IIFE就是函数，我们可以传递`i`进去，我们喜欢的话，可以命名为`j`，我们也可以继续使用`i`。不管怎样，代码都有效。

在每次迭代中使用IIFE为每次迭代创建了一个新的作用域，这使得我们的超时函数回调有机会闭合每个迭代的新作用域，并且我们在每次迭代可以访问到一个恰当的值。

问题解决了！

### 再探块级作用域

仔细查看我们对先前解决方案的分析。我们每次迭代使用IIFE来创建新作用域。换句话说，我们实际上每次迭代都*需要*一个**跨级作用域**。第3章向我们展示了`let`声明，它可以劫持一个块并在块中声明变量。

**它实际上将一个块转换为我们可以关闭的作用域。**所以，以下很棒的代码“可以工作”：

```js
for (var i=1; i<=5; i++) {
	let j = i; // yay, block-scope for closure!
	setTimeout( function timer(){
		console.log( j );
	}, j*1000 );
}
```

*但是，还没完！*（以我最好的Bob Barker的声音）。在for循环的头部使用`let`声明，定义了一种特殊的行为。这种行为表明变量在循环中不仅仅声明一次，**而是每次迭代**。并且，它将在上一轮迭代值的基础上，重新初始化。

```js
for (let i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

多么酷啊？块级作用域和闭包联手工作，解决了所有的问题。我不知道你怎么样，反正这让我成为一个快乐的JavaScripter。

## 模块

还有一种代码模式，充分利用了闭包的能力，并且不是仅仅使用表面层次的回调函数。让我们一起看一看：*模块*。

```js
function foo() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}
}
```

如上述代码所示，并没有看到闭包的使用。我们只是简单的创建了私有数据变量`something`和`another`，以及两个内部函数`doSomething()`和`doAnother()`，它们同时在`foo()`的内部作用域上具有词法最作用域（闭包！）。

但是现在考虑：

```js
function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
}

var foo = CoolModule();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

这就是JavaScript里的模式，我们成为*模块*。实现模块模式的最常见方式通常称为“显示模块”，我们在此处提供的是变体。

我们来看看有关这段代码的一些细节。

首先，`CoolModule()`只是一个函数，但*必须被调用*才能创建模块实例。如果不执行外部函数，则不会创建内部作用域和闭包。

其次，`CoolModule()`函数返回一个对象，这个对象由对象字面量语法`{ key: value, ... }`创建。我们返回的对象只有内部函数的引用，但是*没有*内部数据变量的引用。这样我们可以将这些变量隐藏并保持私有。这个返回的对象可以被认为是一个**我们模块的公共API**。

这个返回的对象最终被赋值给外部变量`foo`，这样我么就可以访问这个API上的属性方法，比如`foo.doSomething()`。

**注意：**我们从我们的模块返回一个实际的对象（字面量）并不是必须的。我们可以直接返回一个内部的函数。jQuery就是一个这样的例子。`jQuery`和`$`标识符是jQuery『模块』的公共API，但是，它们本身只是一个函数（由于所有的函数就是对象，自身含有很多属性）。

函数`doSomething()`和`doAnother()`对于模块『实例』（调用`CoolModule()`时产生）内部作用域拥有闭包。当我们在词法作用域之外通过返回对象传递这些函数时，我们创造了观察和使用闭包的场景。

更简单地说，模块模式有两个『要求』：

1. 必须有一个外层包裹函数，同时它必须被调用一次（每次创建一个模块实例）。

2. 这个包裹函数必须至少返回一个内部函数，这样内部函数在私有作用域上拥有闭包，可以访问或者修改私有状态。

在可观察的意义上，仅具有函数属性的对象不是*真正的*模块。从函数调用返回的对象（其上只有数据属性而没有闭包函数）也不是*真正的*模块。

上面的代码片段显示了一个名为`CoolModule()`的模块创建者，每次调用可以创建一个新模块实例。这种模式的一个变种是，当你只需要一个实例时的『单例』形式：

```js
var foo = (function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
})();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

这里，我们将我们的模块函数变为一个IIFE（见第三章），我们*立即*调用了他，并将其返回值直接赋给了我们的单例模块实例`foo`。

模块仅仅是函数，所以它们可以接受参数：

```js
function CoolModule(id) {
	function identify() {
		console.log( id );
	}

	return {
		identify: identify
	};
}

var foo1 = CoolModule( "foo 1" );
var foo2 = CoolModule( "foo 2" );

foo1.identify(); // "foo 1"
foo2.identify(); // "foo 2"
```

另一个微小但是强大的模块模式变种是，将你的公共API返回值对象命名。

```js
var foo = (function CoolModule(id) {
	function change() {
		// modifying the public API
		publicAPI.identify = identify2;
	}

	function identify1() {
		console.log( id );
	}

	function identify2() {
		console.log( id.toUpperCase() );
	}

	var publicAPI = {
		change: change,
		identify: identify1
	};

	return publicAPI;
})( "foo module" );

foo.identify(); // foo module
foo.change();
foo.identify(); // FOO MODULE
```

通过在模块实例中保留对公共API对象的内部引用，你可以**从内部**修改该模块实例，包括添加和删除方法，属性，*和*更改它们的值。

### 现代模块

各种模块依赖加载器/管理器将这种模块定义模式包装成友好的API。这里不介绍任何一个特定的库，让我提出一个*非常简单的*，**仅用于说明目的**的概念证明：

```js
var MyModules = (function Manager() {
	var modules = {};

	function define(name, deps, impl) {
		for (var i=0; i<deps.length; i++) {
			deps[i] = modules[deps[i]];
		}
		modules[name] = impl.apply( impl, deps );
	}

	function get(name) {
		return modules[name];
	}

	return {
		define: define,
		get: get
	};
})();
```

这段代码的关键部分是`modules[name] = impl.apply(impl, deps)`。这里调用了模块声明包裹函数（传入了一些依赖），并且将返回值，模块的API，存储到了一个内部的模块列表，并且可以通过名称追踪到。

这里是我怎样使用它来定义一些模块：

```js
MyModules.define( "bar", [], function(){
	function hello(who) {
		return "Let me introduce: " + who;
	}

	return {
		hello: hello
	};
} );

MyModules.define( "foo", ["bar"], function(bar){
	var hungry = "hippo";

	function awesome() {
		console.log( bar.hello( hungry ).toUpperCase() );
	}

	return {
		awesome: awesome
	};
} );

var bar = MyModules.get( "bar" );
var foo = MyModules.get( "foo" );

console.log(
	bar.hello( "hippo" )
); // Let me introduce: hippo

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

"foo"和"bar"模块都是通过一个返回公共API的函数定义。"foo" 甚至接受了作为一个依赖的"bar"的参数，随后就可以使用它了。

花一些时间检查这些代码片段，以充分了解为达到我们自己的目的而使用的闭包强大功能。关键的一点是，模块管理器并没有真正的“魔力”。它们只是实现了我在上面列出的模块模式的两个特性：调用函数定义包装器，并将其返回值保持为该模块的API。

换句话说，即使你在顶层将一个友好的工具包裹它，模块也仅仅是模块。

### 未来模块

ES6为模块概念提供了一等的语法支持。当通过模块系统加载时，ES6将一个文件视为一个独立的模块。每个模块可以导入其他模块或者指定的API成员，同时也可以导出它们自己的公共API成员。

**注意：** 以函数为基础的模块并不是一种静态可识别的模式（编译器直接可处理），因此它们的API语法直到运行时才确定。即，你可以在运行时修改模块的API（见之前的`publicAPI`讨论）。

相反，ES6模块API是静态的（这些API在运行时不会改变）。由于编译器直到*这一点*，它可以（确实!）在（文件加载以及）编译时检查一个导入模块API成员的引用*确实存在*。如果API引用并不存在，编译器会"提前"在编译时抛出异常（和错误，如果有一些），而不是直到传统的动态运行时解析。

ES6模块**并不**存在一种『内联』形式，它们必须在独立的文件（一个模块一个文件）中定义。浏览器/引擎有一个默认的『模块加载器』（可以被重载，但是这不在现在的讨论范围），它们可以同步地在导入模块时，加载一个模块文件。

考虑：

**bar.js**
```js
function hello(who) {
	return "Let me introduce: " + who;
}

export hello;
```

**foo.js**
```js
// import only `hello()` from the "bar" module
import hello from "bar";

var hungry = "hippo";

function awesome() {
	console.log(
		hello( hungry ).toUpperCase()
	);
}

export awesome;
```

```js
// import the entire "foo" and "bar" modules
module foo from "foo";
module bar from "bar";

console.log(
	bar.hello( "rhino" )
); // Let me introduce: rhino

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

**注意：**需要创建 **"foo.js"** 和 **"bar.js"** 独立的文件，文件内容是上述代码的前两段。然后你的代码可以加载/导入这些模块并使用，如第三段代码所示。

`import`可以从一个模块API中导入一个或多个成员到当前作用域中，每一个可以绑定到一个变量（我们例子中是`hello`）上。`module`可以讲整个模块API导入并绑定变量（我们例子中是`foo`, `bar`）。`export`可以导出一个标识符（变量，函数）到当前模块的公共API上。这些操作符可以在模块定义需要时多次使用。

*模块文件*中的内容，被当做一个闭合的作用域闭包，就像之前看到的函数闭包模块。

## 回顾 (TL;DR)

闭包似乎是一种不被人所知的，一个在JavaScript内部的神秘世界，只有少数最勇敢的灵魂可以达到。但它实际上只是我们如何在词法作用域的环境中编写代码的标准，并且几乎是显而易见的，其中函数是值并且可以随意传递。

**闭包是，一个函数可以记住并访问它的词法作用域，即使当它在它的词法作用域之外被调用时。**

如果我们没有小心识别出闭包如何工作，它可能欺骗我们，比如在循环中。但是它们也是一个非常强力的工具，可以实现不同类型的*模块*模式。

模块需要两个关键的特性：1）必须有一个外层包裹函数，同时它必须被调用一次（每次创建一个模块实例）。2) 这个包裹函数必须至少返回一个内部函数，这样内部函数在私有作用域上拥有闭包，可以访问或者修改私有状态。

现在我们可以看到我们的代码中到处都有闭包的身影，同时我们有能力来识别和发挥它们的能力。
