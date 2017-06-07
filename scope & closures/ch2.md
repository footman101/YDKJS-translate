> 原文地址：[https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20%26%20closures/ch2.md](https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20%26%20closures/ch2.md)

# 你不知道的JS：作用域与闭包
# 第二章：词法作用域

在第一章中，我们将『作用域』定义为*引擎*用标识符查找变量的规则，包括当前*作用域*，以及其被包裹的*嵌套作用域*。

作用域的工作模式主要有两种。第一种是目前为止最普遍的，被大量的编程语言所使用。它称为**词法作用域**，我们将深入探究它。另外一种模式依旧被很多语言所使用（比如Bash脚本，Perl中的一些模式等等），称为**动态作用域**。

附录A中介绍了动态作用域，我在这里提到它只是为了与词法作用域对比，词法作用域是JavaScript采用的作用域模型。

## 词法时间

正如我们在第一章中讨论的那样，在传统上，标准的语言编译器第一个阶段被称为分词（又称，标记）。如果你记得之前学的，分词阶段会检查一串源代码字符，并且进行一些有状态的解析，对标记赋予语法意义。

正是这个概念提供了理解词法作用域是什么以及其名字来自何处的基础。

要想定义它有些绕，词法作用域定义在词法分析时。换句话说，词法作用域是基于变量和作用域块来定义的，对于你来说，是在写代码的时候，然后在词法解析你的代码时变成板上钉钉的作用域。


**注意：** 我们将会看到有一些欺骗词法作用域的方法，从而在词法解析通过后修改作用域，但这些方法都不建议使用。最好的实践就是把词法作用域看做，实际上，仅仅是词法而已，并且全部自然地定义。

让我们来看看这段代码：

```js
function foo(a) {

	var b = a * 2;

	function bar(c) {
		console.log( a, b, c );
	}

	bar(b * 3);
}

foo( 2 ); // 2 4 12
```

在这段代码示例中有三个嵌套的作用域。把它们想象成泡泡可能会有助于理解。

<img src="https://github.com/getify/You-Dont-Know-JS/raw/master/scope%20&%20closures/fig2.png" width="500">

**泡泡1** 包含全局作用域，其中只有一个标识符：`foo`。

**泡泡2** 包含`foo`作用域，包括三个标识符`a`，`bar`以及`b`。

**泡泡3** 包含`bar`作用域，其中只有一个标识符：`c`。

哪里写了作用域块，作用域泡泡就在哪里定义，这些作用域块是一个嵌套一个的。在下一章中，我们会讨论作用域的不同单元，但是现在，我们只是假设每一个函数创造了一个新的作用域泡泡。

`bar`泡泡完全被包括在`foo`中，因为（并且仅仅因为）我们选择在其内部定义了`bar`函数。

要注意到，这些嵌套的泡泡是严格地嵌套着。我们不是在讨论维恩图中那些可以可以跨越边界的泡泡。换句话说，不存在某些函数作用域泡泡可以同时（部分地）存在于两个其他外层的作用域泡泡，正如没有函数可以部分地存在于两个父函数中。

### 查找

这些作用域泡泡的结构和相对位置可以完全地解释*引擎*查找一个标识符时要检查的所有位置。

在之前的代码片段中，*引擎*执行`console.log(..)`语句，然后查找三个引用变量`a`，`b`和`c`。它会从最里边的作用域泡泡，即`bar(..)`函数的作用域，开始查找。引擎在这里并不能找到`a`,因此它会去上一层级查找，紧邻的外层作用域泡泡，`foo(..)`函数的作用域。引擎在这里会找到`a`，因此它可以使用`a`了。查找`b`的过程基本类似，但是`c`确是不同，在`bar(..)`内部就能找到。

如果在`bar(..)`以及`foo(..)`中都有`c`，`console.log(..)`会找到并使用`bar(..)`中的`c`，永远不会找到`foo(..)`中。

**作用域查找在找到第一个匹配项后立即停止**。相同名称的标识符可能出现在嵌套作用域的多个层级中，这种现象叫做『阴影（shadowing）』（内层的标识符的影子笼罩了外层的标识符）。不管阴影如何，作用域查找总是从当前执行的最内层作用域开始，并且向外/向上查找，直到第一个匹配，然后停止。

**注意：** 全局变量会自动成为全局对象（浏览器中的`window`，等等）的属性。因此，要想不通过词法名称来引用全局变量*是*可能的，但是要使用全局变量的属性来引用。

```js
window.a
```

这种机制使我们可以绕开阴影，直接访问到全局变量。然而，非全局变量在阴影的影响下不能被访问到。

无论一个函数*在哪里*，甚至无论*怎样*被调用。它的词法作用域**仅仅**在函数声明的地方被定义。

词法作用域的查找*仅仅*处理第一级的标识符，比如`a`，`b`和`c`。如果你在一段代码中引用`foo.bar.baz`，词法作用域只会查找`foo`标识符，但是一旦它找到了这个变量，对象属性访问规则会继续单独解析`bar`和`baz`属性。

## 欺骗词法

如果词法作用域仅仅在函数声明的地方被定义，完全是书写时决定的，那么是否有可能在运行时『修改』（又称，欺骗）词法作用域。

JavaScript有两个机制，它们在广大社区中是作为不良实践存在。但是反对它们的典型论据中往往缺少最重要的一点：**欺骗词法作用域导致了低性能。**

在我解释性能问题前，让我们先来看看这两种机制如何工作。

### `eval`

JavaScript中的`eval(..)`将一个字符串作为参数，并且将该字符串的内容视为程序此刻实际上的代码。换句话说，你可以通过编程，在代码内生成代码，并运行生成的代码，就像它早被写好一样。

在这方面评价`eval（..）`（双关意图），应该清楚`eval（..）`如何允许你通过作弊和假装书写时（又称词法）代码一直在那里的方式，修改词法作用域。

在`eval(..)`语句执行后，在其后的代码执行时，*引擎*并不『知道』或者『关心』前边的代码是否是动态解释，并且修改了词法作用域环境。*引擎*只会像往常一样，简单地进行词法作用域查找。

考虑如下代码：

```js
function foo(str, a) {
	eval( str ); // cheating!
	console.log( a, b );
}

var b = 2;

foo( "var b = 3;", 1 ); // 1, 3
```

字符串`"var b = 3;"`是有问题的，在`eval(..)`执行时，就像它（`"var b = 3;"`）之前一直在那里了。由于这行代码声明了一个新的变量`b`，它修改了`foo(..)`已经存在的词法作用域。实际上，正如之前提到过的，这段代码在`foo(..)`内部创建了`b`变量，它遮盖了外层（全局）作用域声明的`b`。

当`console.log(..)`执行时，它会发现`a`和`b`都存在于`foo(..)`作用域中，永远不会找到外层的`b`.因此，我们打印出了"1, 3"，而不是像往常一样打印出"1, 2"。

**注意：** 在这个例子中，为简单起见，我们传入的『代码』字符串是一串固定的文字。但它可以很容易通过程序逻辑来增加字符，达到动态编程的目的。`eval(..)`通常用来执行动态生成的代码，但是这种动态从一个字符串生成的代码比不上正常写作的代码。

一般的，如果`eval(..)`执行的代码字符串中包括一个或多个声明（变量或者函数），这种行为会修改`eval(..)`所在的当前已经存在的词法作用域。从技术上讲，`eval(..)`可以通过各种技巧（除了我们在这里的讨论）『间接』调用，这导致它在全局作用域的上下文中执行，从而修改它。但在任何一种情况下，`eval(..)`都可以在运行时修改一个书写时词法作用域。

**注意：** `eval(..)`在严格模式程序中使用时，它在自己的词法作用域中运行，这意味着在`eval(..)`里面做的声明不会实际修改它所在的作用域。

```js
function foo(str) {
   "use strict";
   eval( str );
   console.log( a ); // ReferenceError: a is not defined
}

foo( "var a = 2" );
```

JavaScript中有其他一些设施效果与`eval(..)`基本类似。`setTimeout(..)`和`setInterval(..)`*可以*将一个字符串作为第一个参数，字符串的内容可以作为代码运行在一个动态生成的函数中。这是旧的，遗留的行为和已经弃用很久了。不要这样做！

`new Function(..)`函数构造函数类似地将一个代码字符串作为它**最后**的参数，这个字符串会变成动态生成的函数体（前几个参数，如果有的话，就是新函数的参数名称）。这种函数构造器的语法比`eval(..)`更安全一些，但是它仍然最好不要出现在你的代码中。

在程序中动态生成代码的情况非常少，因为性能的下降程度引起的危害远远大于它们的功能。

### `with`

JavaScript中另外一个欺骗词法作用域的不好（现在已经废弃了）特性就是`with`关键词。有多个合理的角度可以解释`with`，但我这里选择从它如何与词法作用域交互和影响词法作用域的角度。

`with`通常被解释为对一个对象进行多个属性引用的简写，*而不是*每次都重复对象引用本身。

例如：

```js
var obj = {
	a: 1,
	b: 2,
	c: 3
};

// more "tedious" to repeat "obj"
obj.a = 2;
obj.b = 3;
obj.c = 4;

// "easier" short-hand
with (obj) {
	a = 3;
	b = 4;
	c = 5;
}
```

然而，除了对象属性访问简写的便利性之外，还有一些东西需要考虑：

```js
function foo(obj) {
	with (obj) {
		a = 2;
	}
}

var o1 = {
	a: 3
};

var o2 = {
	b: 3
};

foo( o1 );
console.log( o1.a ); // 2

foo( o2 );
console.log( o2.a ); // undefined
console.log( a ); // 2 -- Oops, leaked global!
```

在这段代码示例中，`o1`和`o2`两个对象被创建。其中一个拥有`a`属性，另一个则没有。`foo(..)`函数接受一个对象`obj`作为参数，然后对其调用`with (obj) { .. }`。在`with`块内，我们对变量`a`进行正式的词法引用，实际上一次LHS引用（参见第一章），将其赋值为`2`。

当我们将`o1`传入时，赋值`a = 2`语句找到了`o1.a`然后将其赋值为`2`，正如接下来的`console.log(o1.a)`语句所反应的那样。然而，当我们传入`o2`时，由于它并没有`a`属性，此时不会创建这样一个属性，`o2.a`继续保持为`undefined`。

但是我们要注意到一个奇怪的副作用，在`a = 2`赋值语句中，一个全局变量`a`被创建了。这是怎么回事呢？

`with`声明需要一个对象，这个对象拥有零个或多个属性，并且**将该对象视为一个完全独立的词法作用域**，因此对象的属性被视为词法定义在这个『作用域』中的的标识符。

**注意：** 即使`with`块将一个对象视为一个词法作用域，`with`块中的一个正常`var`声明的变量并不属于它，而是属于包含它的函数作用域中。

虽然如果`eval（..）`接受了一个或多个声明的代码字符串作为变量，那么它就可以修改现有的词法作用域。而`with`语句实际上从你传递给它的对象创建了一个**全新的词法作用域**。

我们可以这样理解，当我们传入`o1`时，`with`语句声明的『作用域』是`o1`，并且在『作用域』中有一个『标识符』，它对应于`o1.a`属性 。但是当我们使用`o2`作为『作用域』时，它并不包含`a`标识符，因此发生了正常的LHS标识符查找规则（见第1章）。

在`o2`作用域，`foo（..）`作用域，甚至全局作用域中，都没有要找到的`a`标识符，所以当执行`a = 2`时，在全局变量被自动创建（因为我们在非严格模式）。

这里有些绕，在运行时，一个对象及其属性转换为一个作用域*以及*标识符。但这是我根据结果，能给出的最清楚的解释了。

**注意：** 如果非要使用它们的话，`eval（..）`和`with`都在严格模式下都受到了影响（限制）。`with`是完全禁止使用的，而不同形式的间接或不安全的`eval(..)`是不允许的，只保留了核心功能。

### 性能

`eval（..）`和`with`都可以在运行时修改或创建新的词法作用域，欺骗书写时定义好的词法作用域。

因此，你会问这有什么关系呢？如果他们提供了实用性和代码灵活性，难道不是*好*的特性吗？**并不是。**

JavaScript*引擎*具有许多在编译阶段执行的性能优化。其中一些归结为一般能够静态分析代码，预先确定所有变量和函数声明在哪里，使得在执行期间解析标识符花费更少的代价。

但是如果*引擎*在代码中找到了`eval(..)`或者`with`，它一般必须*假设*它之前看到的所有标识符位置可能是无效的，因为它无法在词法分析阶段确定你可能传入`eval(..)`的代码，这些代码修改了作用域，或者你可能传给`with`的对象，这个对象创建了一个新的词法作用域。

换句话说，在悲观的意义上，如果`eval（..）`或`with`存在，那么它所做的大多数优化*将会*是无意义的，所以它*根本不*执行优化。

你的代码几乎肯定会趋向于运行更慢，只要你在代码中的任何地方包含一个`eval（..）`或`with`。不管*引擎*如何聪明地试图限制这些悲观的假设的副作用，**不会绕过这样的事实，即没有优化，代码运行速度就会降低。**

## 回顾 (TL;DR)

词汇作用域意味着作用域由声明函数的编写时决定。编译的词法分析阶段基本上能够知道在何处以及如何声明所有标识符，从而预测如何在执行期间查找它们。

JavaScript中的两种机制可以“欺骗”词法作用域：`eval（..）`和`with`。前者可以通过运行具有一个或多个变量声明的“代码”字符串来修改现有的词法作用域（在运行时）。后者创建一个全新的词法作用域（再次，在运行时）通过将对象引用*作为*作用域，将该对象的属性作为作用域内的标识符。

这些机制的缺点是，它使得*引擎*无法执行作用域查找的编译时优化，因为*引擎*必须悲观地假定这样的优化将是无效的。使用它们中的任何一个都*将*使代码运行速度变慢。**不要使用它们。**