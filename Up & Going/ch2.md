原文地址：[https://github.com/getify/You-Dont-Know-JS/blob/master/up%20&%20going/ch2.md](https://github.com/getify/You-Dont-Know-JS/blob/master/up%20&%20going/ch2.md)

# 第二章: 进入JavaScript

在前面的章节中，我介绍了基本的编程部件，比如变量，循环，条件和函数。当然，所有的代码都是用JavaScript写的。但是在这一章里，我们准备聚焦JavaScript中一些比较特殊的东西，来成为一位真正的JavaScript开发者。

我们在这一章里会介绍很多概念，这些概念会随后的*YDKJS*系列书籍中介绍。你可以认为本章是后面几本书的概览。

特别是如果你是新学习JavaScript，你应该花大量的时间来复习这些概念和代码，反复练习。要打下良好的地基，需要一块砖一块砖的来夯实，所以不要期望你会看一遍就会立马学会这些东西。

你的JavaScript深入学习之路，从这里开始。

**注意：**就像我在第一章中所说的，你应该在你阅读和学习这一章时，切实自己体验一下这些代码。注意这里的一些代码假定使用本书写作时的最新版本的JavaScript（通常称为『ES6』，第六版ECMAScript -- JS规范里的官方名称）。如果你是用一个比较老的，ES6之前的浏览器，这些代码将不会工作。你应该升级你的浏览器到一个现代浏览器（比如Chrome, Firefox, 或者IE）。

## 值与类型

正如我们在第一章中提到的，JavaScript的值是有类型的，但是变量没有类型。以下是几种JavaScript内置类型：

* `string`
* `number`
* `boolean`
* `null` and `undefined`
* `object`
* `symbol` (ES6新类型)

JavaScript提供了一个叫做`typeof`的操作符，它可以用来检查一个值的类型：

```js
var a;
typeof a;				// "undefined"

a = "hello world";
typeof a;				// "string"

a = 42;
typeof a;				// "number"

a = true;
typeof a;				// "boolean"

a = null;
typeof a;				// "object" -- weird, bug

a = undefined;
typeof a;				// "undefined"

a = { b: "c" };
typeof a;				// "object"
```

`typeof`操作符的返回值是六种字符串（ES6中有七种）中的的一个。比如`typeof "abc"`返回`"string"`，而不是`string`.

注意在这段代码中，变量`a`装载了不同类型的值，因此尽管看起来`typeof a`是询问`a`的类型，但实际上是询问`a`当前装载的值的类型。在JavaScript中，只有值有类型；变量只是这些值的容器。

`typeof null`是一个有趣的情况，因为它错误地返回了`"object"`，实际上我们期望它返回`"null"`。

**警告：**这是在JS中一个长期存在的bug，但是这一个问题似乎将永远不会被修复。因为当前Web上有太多的代码依赖这个bug，修改这个bug会引发更多的bug。

同时，注意`a = undefined`。我们显式地设置`a`为`undefined`，但是这同于一个变量没有被设置任何值，就像代码在一开始的`var a;`。一个变量有很多方法可以变成"undefined"，包括没有返回值的函数和使用`void`操作符。

### 对象

`object`类型表示一组值的集合，这些值被称为对象的属性。它可能是JavaScript中最有用的类型了。

```js
var obj = {
	a: "hello world",
	b: 42,
	c: true
};

obj.a;		// "hello world"
obj.b;		// 42
obj.c;		// true

obj["a"];	// "hello world"
obj["b"];	// 42
obj["c"];	// true
```

`obj`的图形化表示如下图所示：

[![](https://github.com/getify/You-Dont-Know-JS/raw/master/up%20&%20going/fig4.png)](https://github.com/getify/You-Dont-Know-JS/blob/master/up%20&%20going/fig4.png)

访问属性的方式有两种，使用*点符号*（例如`obj.a`）或者使用*方括号符号*（例如`obj["a"]`）。点符号比较短小精悍，而且通常易于阅读，因此推荐使用这种方式。

方括号符号在属性名中有特殊符号时比较有用，比如`obj["hello world!"]` -- 属性在使用方括号表示时，通常指的是*键（keys）*。`[ ]`符号内部需要一个变量（稍后解释）或者一个`string`*字面量*（需要用`" .. "`或者`' .. '`包裹）。

当然，方括号符号在你访问属性的名称存储在另外一个变量中时同样有用，例如：

```js
var obj = {
	a: "hello world",
	b: 42
};

var b = "a";

obj[b];			// "hello world"
obj["b"];		// 42
```

**注意：**更多关于`object`的信息，参考本系列书的*this和对象原型*，特别是其中的第三章。

还有另外一些类型你在JavaScript中会经常使用：*array*和*function*。虽然它们也是通用的内置类型，但是它们应该被视为子类型 -- `object`类型的特殊版本。

#### 数组

一个数组是含有一组值（任意类型）的对象，这些值没有特别的属性名/键名，但是按照数字索引的位置排列。例如

```js
var arr = [
	"hello world",
	42,
	true
];

arr[0];			// "hello world"
arr[1];			// 42
arr[2];			// true
arr.length;		// 3

typeof arr;		// "object"
```

**注意：**编程语言从零开始计数，JS也是这样的，使用`0`作为数组中第一个元素的索引。

`arr`的图形化展示如下所示：

[![](https://github.com/getify/You-Dont-Know-JS/raw/master/up%20&%20going/fig5.png)](https://github.com/getify/You-Dont-Know-JS/blob/master/up%20&%20going/fig5.png)

由于数组是特殊的对象（正如`typeof`操作符表示的那样），他们可以拥有属性，包括自动更新的`length`属性。

理论上，你可以把数组当成一个普通对象来使用，使用自己命名的属性。或者你可以给一个`object`赋予数字属性(`0`, `1`, 等等)，类似一个数组。然而，这两种用法通常都是不合理的用法。

最好的，最自然的方式是，在数组上，使用数字索引定位，在`object`上使用命名的属性。

#### 函数

函数是`object`的另外一个子类型，也是你在JS中一致会使用的东西：

```js
function foo() {
	return 42;
}

foo.bar = "hello world";

typeof foo;			// "function"
typeof foo();		// "number"
typeof foo.bar;		// "string"
```

再次说明，函数是`objects`的子类型 -- `typeof`会返回`"function"`，这意味着`function`是一种主要的类型 -- 同事拥有属性，但是你应该有限度地使用函数对象的属性（比如`foo.bar`）。

**注意：** 更多关于JS值和它们的类型，请看本系列书的*类型和语法*的前两章。

### 内置类型方法

我们刚才讨论过的这些内置类型和子类型暴露出了很多属性和方法，这些属性和方法非常的健壮和有用。

例如:

```js
var a = "hello world";
var b = 3.14159;

a.length;				// 11
a.toUpperCase();		// "HELLO WORLD"
b.toFixed(4);			// "3.1416"
```

在`a.toUpperCase()`语句中，为什么`a`上会有这样一个方法？这背后的原因非常复杂。

简单来说，有一种包装形式为`String`(大写`S`)对象，它会原生地包装基本类型`string`；在这种包装对象类型的原型上定义了`toUpperCase()`方法。

当你把一个基本类型的值比如`"hello world"`当做一个对象来用时，你需要使用一个属性或者方法（例如`a.toUpperCase()`），JS会自动地『包装』这个值，并使用包装后的副本。

一个`string`类型的值可以被一个`String`对象包装，一个`number`类型的值可以被一个`Number`对象包装，一个`boolean`类型的值可以被一个`Boolean`对象包装。重要的的一点是，你不需要担心或者直接使用包装形式对象 -- 几乎在所有情况下，你只需使用基本数据类型，JavaScript会帮你搞定一切。

**注意：** 更多关于JS原生以及『包装』的信息，请参看*类型与工具*的第三章。要更好地理解对象原型，请参看*this和对象原型*的第五章。

### 值的比较

在JS程序中，你主要会使用的有两种比较类型：*相等*和*不等*。无论比较的值为何种类型，比较的结果必然都会是一个`boolean`的值(`true`或者`false`)。

#### 类型转换

我们在第一章中简单介绍了类型转换，我们再重新『拜访』一下它。

JavaScript中的类型转换会有两种形式：*显示*和*隐式*。你在代码中会明显地看到显示的类型转换，一种类型会转换为另一种类型。但是隐式的类型转换会在不太明显的情况下发生，往往是一些操作的附带影响。

你可能听说过『类型转换是魔鬼』，因为类型转换会产生一些令人疑惑的结果。当开发者被编程语言搞得疑惑时，他们就会沮丧起来。

类型转换并不是魔鬼，也没有什么令人惊奇的。实际上，大多数你能构造的类型转换是非常明智和容易被理解的，甚至可以被用来*提高*你的代码的可读性。但是在这里我们不会更深入地讨论 -- *类型和语法*的第四章会覆盖到方方面面。

这里有一个*显式的*类型转换的例子：

```js
var a = "42";

var b = Number( a );

a;				// "42"
b;				// 42 -- the number!
```

这里有一个*隐式式的*类型转换的例子：

```js
var a = "42";

var b = a * 1;	// "42" 隐式地转换为 42

a;				// "42"
b;				// 42 -- the number!
```

#### 真值和假值

在第一章中，我们简单提到了值天然地具有『真』或『假』的属性。当一个非`boolean`类型的值转换为`boolean`类型时，它们终归要变成`true`或`false`中的一个。

JavaScript中的详细『假』值，如下所示

* `""` (空字符串)
* `0`, `-0`, `NaN` (非法`number`)
* `null`, `undefined`
* `false`

不在上面列表中的值，是『真』值。这里有一些例子：

* `"hello"`
* `42`
* `true`
* `[ ]`, `[ 1, "2", 3 ]` (数组)
* `{ }`, `{ a: 42 }` (对象)
* `function foo() { .. }` (函数)

重要的一点是，一个非`boolean`类型的值，在真正要转换为`boolean`类型时才会遵守上述规则。

#### 相等

一共有四种比较操作符：`==`, `===`, `!=`, 以及 `!==`。带`!`的形式是不带`!`的形式对应的『不相等』版本；注意不要将*不相等（non-equality）*和*不等式（inequality）*混淆。

`==`和`===`的区别在于，`==`检查的是值是否相等，`===`会同时检查值和类型是否相等。然而，这种说法并不准确。更合适的说法是，`==`会先进行强制类型转换，然后比较值是否相等，而`===`并不会进行强制类型转换，因此它通常被称为『严格相等』。

考虑松散的相等`==`，它会进行隐式类型转换以及严格的相等`===`，它并不会进行类型转换。

```js
var a = "42";
var b = 42;

a == b;			// true
a === b;		// false
```

在比较表达式`a == b`中，JS注意到它们的类型并不匹配，因此会进行一系列有顺序的类型转换，转换两个值的一个或两个的类型知道它们的类型匹配，然后会简单地比较两者的值即可。

如果你思考一下会想到，`a == b`会有两种类型转换的方式，都会得到`true`的结果。这两种情况分别是`42 == 42`或者`"42" == "42"`。那么到底是哪一种呢？


答案是：`"42"`会变成`42`，然后进行`42 == 42`的比较。在这个简单的例子中，到底是哪一种转换方式实际上并不影响比较的结果。在复杂的情况下，重要的不仅是比较的最终结果，而是*为什么*会得到这样的结果。

`a === b`结果是`false`，因为没有进行类型化转换，因此简单值的比较显然是无法进行的。许多开发人员认为`===`是更可预测的，因此他们主张始终使用该形式而不使用`==`。 我认为这个观点很短视。 我相信*如果你花时间来了解它是如何工作的*，`==`会是你程序的一个强大的工具。

我们不会介绍关于`==`比较中的类型转换的工作细节。它们大部分是相当妥当的，但也有一些重要的角落要小心。 您可以阅读ES5规范（http://www.ecma-international.org/ecma-262/5.1/）的第11.9.3节来查看确切的规则，您会惊讶于相比它的负面炒作，这种机制是多么简单。

把很多细节归纳为几个简单的规则，帮助你知道在各种情况下是使用`==`还是`===`，这里是我简单的规则：

* 如果比较中的任一值（任何一边）可能是`true`或`false`值，请避免`==`并使用`===`。
* 如果比较中的任一值可能是这些特定值（`0`，`“”`或`[]` - 空数组），避免`==`并使用`===`。
* 在*所有*其他情况下，你可以安全地使用`==`。 它不仅安全，而且在许多情况下，它简化了您的代码，提高可读性。

这些规则要求你仔细思考你的代码，以及什么类型的值可以进行变量比较。如果你可以确定，同时`==`是安全的，使用它！ 如果你不能确定，使用`===`。 就是这么简单。

`！=`非等式形式与`==`相对，`！==`形式与`===`相对。我们刚刚讨论的所有规则，同样适用于它们。

如果你要比较两个非原始值，比如两个`object`（包括`function`和`array`）类型的值，你应该特别注意`==`和`=== `，因为这些值实际上保存的是引用，`==`和`===`只是检查引用是否匹配，而不是引用背后的值。

例如，`array`在默认情况下要转换成`string`，是用逗号（`，`）连接所有的值。你可能会认为两个具有相同内容的数组用`==`比较是相等的，而它们并不是：

```js
var a = [1,2,3];
var b = [1,2,3];
var c = "1,2,3";

a == c;		// true
b == c;		// true
a == b;		// false
```

**注意：** 有关“==”等式比较规则的更多信息，请参阅ES5规范（第11.9.3节），并参阅本系列书的*类型和语法*的第4章; 有关值与引用的更多信息，请参阅第2章。

#### 不等式

`<`，`>`，`<=`和`> =`运算符用于不等式，在规范中称为『关系比较』。 通常，它们与`number`类型的值一起使用。 很容易理解`3 <4`。

但是JavaScript中的'string'类型的值也可以进行不等式比较，使用典型的字母规则（`“bar” < “foo”`）。

关于不等式的类型转换呢？规则与`==`差不多（并不是太准确！）。要注意的是，没有与不允许类型转换的『严格相等』`===`类似的『严格不等式』操作符。

考虑:

```js
var a = 41;
var b = "42";
var c = "43";

a < b;		// true
b < c;		// true
```

这里发生了什么？在ES5规范的11.8.5小节中说，如果`<`比较中的两个值都是`string`类型，如上例中的`b < c`，比较是按字典顺序进行的。但是如果其中的一个或两个值不是`string`类型，如上例中的`a < b`，那么两个值都被强制转换为`number`类型，进行典型的数字比较。

最大的问题是你可能会遇到不同类型的值之间的比较 -- 记住，没有“严格不等式”形式 -- 当一个值不能被转换成有效的数字时，如：

```js
var a = 42;
var b = "foo";

a < b;		// false
a > b;		// false
a == b;		// false
```

等一等，为什么所有的这三个比较都会是`false`？因为`b`值在`<`和`>`比较中，被转换成了『非法数字值』`NaN`，规范中规定`NaN`既不大于任何数，也不小于任何数。

`==`比较失败的原因则不同。`a == b`可能被转换成`42 == NaN`或者`"42" == "foo"` -- 正如之前我们解释过，实际中被转换成前者。

**注意：** 有关不等式比较规则的更多信息，请参阅ES5规范的第11.8.5节，并参阅本系列书的*类型和语法*的第4章。

## 变量

在JavaScript中，变量名称（b）必须是合法的*标识符*。当考虑非传统字符（如Unicode）时，标识符中严格而完整的有效字符规则有点复杂。 如果你只考虑典型的ASCII字母数字字符，规则就很简单。

标识符必须以`a`-`z`，`A` -`Z`，`$`或`_`开头。它可以包含任何这些字符加数字`0` -`9`。

一般来说，变量标识符的规则同样适用于属性名称。然而，某些单词不能用作变量，但可以作为属性名。这些单词被称为『保留字』，包括JS关键字（`for`，`in`，`if`等）以及`null`，`true`和`false`。

**注意：** 有关保留字的更多信息，请参阅本系列书的*类型和语法*的附录A.

### 函数作用域

当你使用`var`关键字来声明一个变量时，这个变量属于当前函数的作用域，或者在函数之外进行顶层声明时，它属于全局作用域。

#### 作用域提升

无论`var`出现在一个作用域的任何地方，该声明都被认为属于整个作用域，并且可以在任何地方访问。

从直观感觉上讲，就像`var'声明被“移动”到了作用域顶部，这种行为被称为*作用域提升*。从技术上讲，这个过程更准确地原因是编译代码的方式，但我们现在可以跳过这些细节。

考虑:

```js
var a = 2;

foo();					// works because `foo()`
						// declaration is "hoisted"

function foo() {
	a = 3;

	console.log( a );	// 3

	var a;				// declaration is "hoisted"
						// to the top of `foo()`
}

console.log( a );	// 2
```

**警告：** 使用*作用域提升*，可以在`var`声明前使用这个变量，但是这种做法并不是很常见而且不是一个好主意；这样会造成混乱。函数声明的*作用域提升*则更加常见和容易被接受，就像上例中我们对`foo()`调用出现在它的正式声明之前。

#### 嵌套作用域

当你声明一个变量后，它既可以在当前作用域的任何位置被访问，又可以在它内部的作用域中被访问，例如：

```js
function foo() {
	var a = 1;

	function bar() {
		var b = 2;

		function baz() {
			var c = 3;

			console.log( a, b, c );	// 1 2 3
		}

		baz();
		console.log( a, b );		// 1 2
	}

	bar();
	console.log( a );				// 1
}

foo();
```

注意`c`在`bar()`内部并不能被访问，因为它仅仅是在`baz()`内部作用域被声明，同时由于同样的原因，`b`在`foo()`内部也不能被访问。

如果你试图在一个变量不可用的作用域中访问它的值，你会得到一个`ReferenceError`类型的错误被抛出。如果你试图设置一个未声明的变量，你将会在顶层全局作用域内创建一个变量（这样不好！），在『严格模式』，你会得到一个错误。让我们来看看：

```js
function foo() {
	a = 1;	// `a` not formally declared
}

foo();
a;			// 1 -- oops, auto global variable :(
```

这是一种非常坏的实践方式。不要这样做！永远要正式声明你的变量。

除了在函数级别创建变量声明之外，ES6*允许*使用`let`关键字声明变量，这个变量属于单个块（`{..}`）。 除了一些细微的细节，作用域规则的行为大致与我们刚说过的函数相同。

```js
function foo() {
	var a = 1;

	if (a >= 1) {
		let b = 2;

		while (b < 5) {
			let c = b * 2;
			b++;

			console.log( a + c );
		}
	}
}

foo();
// 5 7 9
```

由于使用`let`而不是`var`，`b`只属于`if`语句，不属于整个`foo()`函数的作用域。 同样，`c`只属于`while`循环。 块范围以更细的粒度管理变量作用域，这可以使您的代码随着时间的推移更容易维护。

**注意：** 有关作用域的更多信息，请参阅本系列书的*作用域和闭包*主题。 有关`let`块作用域设定的更多信息，请参阅本系列书的*ES6及其之外*主题。

## 条件

除了我们在第1章中简要介绍的`if`语句，JavaScript提供了一些其他的条件机制，我们应该看看。

有时你可能会发现自己写了一系列的`if..else..if`语句：

```js
if (a == 2) {
	// do something
}
else if (a == 10) {
	// do another thing
}
else if (a == 42) {
	// do yet another thing
}
else {
	// fallback to here
}
```

这种写法可以，但它有点啰嗦，因为你需要为每种情况指定`a`的条件。这种情况下，有另外一个选择，`switch`语句：

```js
switch (a) {
	case 2:
		// do something
		break;
	case 10:
		// do another thing
		break;
	case 42:
		// do yet another thing
		break;
	default:
		// fallback to here
}
```

如果你只想在一个case中执行语句，那么`break`很重要。在`case`中省略`break`，如果`case`匹配并执行其中的语句，之后程序将继续执行下一个`case`中语句，无论下一个`case`是否匹配。这种所谓的“fall through”，它有时是有用的/期望的：

```js
switch (a) {
	case 2:
	case 10:
		// some cool stuff
		break;
	case 42:
		// other stuff
		break;
	default:
		// fallback
}
```

在这里，如果`a`是`2`或者是`10`，"some cool stuff"语句都会被执行。

JavaScript中的另一种条件语句形式是『条件运算符』，通常称为『三元运算符』。 它像一个更简洁的单个`if..else`语句，如：

```js
var a = 42;

var b = (a > 41) ? "hello" : "world";

// similar to:

// if (a > 41) {
//    b = "hello";
// }
// else {
//    b = "world";
// }
```

如果测试表达式（这里的`a> 41`）的计算结果为`true`，则会将第一个子句（`hello`）的值赋给`b`，否则将第二个子句（`world`） 的值赋给`b`。

条件运算符不一定必须在赋值中使用，但这绝对是最常见的用法。

**注意：** 有关测试条件，以及`switch`和`? :`的其他模式请参见本系列书的*类型和语法*主题以获取更多信息。

## 严格模式

ES5向语言添加了“严格模式”，这样可以收紧某些行为的规则。 通常，这些限制使代码更加安全和更加合理。此外，坚持严格模式使您的代码通常更加有利于引擎的优化。严格模式对代码是一个很大的好处，你应该在所有的程序中式用它。

您可以针对单个函数或整个文件选择严格模式，具体取决于您放置严格模式pragma（编译标注）的位置：

```js
function foo() {
	"use strict";

	// this code is strict mode

	function bar() {
		// this code is strict mode
	}
}

// this code is not strict mode
```

对比于:

```js
"use strict";

function foo() {
	// this code is strict mode

	function bar() {
		// this code is strict mode
	}
}

// this code is strict mode
```

使用严格模式的一个关键差异（改进！）是不允许省略`var`时的隐式自动全局变量声明：

```js
function foo() {
	"use strict";	// turn on strict mode
	a = 1;			// `var` missing, ReferenceError
}

foo();
```

你在你的代码中打开严格模式，如果你得到错误，或者代码开始运行出错，你可能会潜意识地避免使用严格模式。 但这种本能地纵容是一个坏主意。如果严格模式导致程序问题，几乎可以肯定你的程序中的这些问题应该被修复。

严格模式会让你的代码更安全，同时使你的代码更优化，而且它也代表了语言的未来方向。从现在开始熟悉严格模式会更容易，而不是把它放在一边 -- 越往后它只会变得更难！

**注意：** 有关严格模式的详细信息，请参阅本系列书的*类型和语法*主题的第5章。

## 作为值的函数

到目前为止，我们已经讨论了作为JavaScript中*作用域*主要机制的函数。 你应该记得典型的`function`声明语法如下：

```js
function foo() {
	// ..
}
```

从语法角度看起来可能不是很明显，`foo`只是外部封闭作用域中的一个变量，它是声明的`function`的引用。也就是说，`function`本身是一个值，就像`42`或`[1,2,3]`一样。

这听起来像一个奇怪的概念，所以要花一点时间来思考它。你不仅可以传递一个值（参数）*给*一个函数，而且*一个函数本身作为一个值*可以赋给变量，或传递给其他函数，或从其他函数返回。


因此，函数值应被视为表达式，非常类似于任何其他值或表达式。

考虑:

```js
var foo = function() {
	// ..
};

var x = function bar(){
	// ..
};
```

分配给`foo`变量的第一个函数表达式被称为*匿名函数*，因为它没有`name`。

第二个函数表达式是*具名的*（`bar`），即使它的引用也被赋给`x`变量。虽然*匿名函数表达式*仍然非常常见，*命名函数表达式*通常是更优的选择。

更多信息请参见本系列书的*作用域和闭包*主题。

### 立即调用函数表达式（IIFEs）

In the previous snippet, neither of the function expressions are executed -- we could if we had included `foo()` or `x()`, for instance.

There's another way to execute a function expression, which is typically referred to as an *immediately invoked function expression* (IIFE):

```js
(function IIFE(){
	console.log( "Hello!" );
})();
// "Hello!"
```

The outer `( .. )` that surrounds the `(function IIFE(){ .. })` function expression is just a nuance of JS grammar needed to prevent it from being treated as a normal function declaration.

The final `()` on the end of the expression -- the `})();` line -- is what actually executes the function expression referenced immediately before it.

That may seem strange, but it's not as foreign as first glance. Consider the similarities between `foo` and `IIFE` here:

```js
function foo() { .. }

// `foo` function reference expression,
// then `()` executes it
foo();

// `IIFE` function expression,
// then `()` executes it
(function IIFE(){ .. })();
```

As you can see, listing the `(function IIFE(){ .. })` before its executing `()` is essentially the same as including `foo` before its executing `()`; in both cases, the function reference is executed with `()` immediately after it.

Because an IIFE is just a function, and functions create variable *scope*, using an IIFE in this fashion is often used to declare variables that won't affect the surrounding code outside the IIFE:

```js
var a = 42;

(function IIFE(){
	var a = 10;
	console.log( a );	// 10
})();

console.log( a );		// 42
```

IIFEs can also have return values:

```js
var x = (function IIFE(){
	return 42;
})();

x;	// 42
```

The `42` value gets `return`ed from the `IIFE`-named function being executed, and is then assigned to `x`.

### 闭包

*Closure* is one of the most important, and often least understood, concepts in JavaScript. I won't cover it in deep detail here, and instead refer you to the *Scope & Closures* title of this series. But I want to say a few things about it so you understand the general concept. It will be one of the most important techniques in your JS skillset.

You can think of closure as a way to "remember" and continue to access a function's scope (its variables) even once the function has finished running.

Consider:

```js
function makeAdder(x) {
	// parameter `x` is an inner variable

	// inner function `add()` uses `x`, so
	// it has a "closure" over it
	function add(y) {
		return y + x;
	};

	return add;
}
```

The reference to the inner `add(..)` function that gets returned with each call to the outer `makeAdder(..)` is able to remember whatever `x` value was passed in to `makeAdder(..)`. Now, let's use `makeAdder(..)`:

```js
// `plusOne` gets a reference to the inner `add(..)`
// function with closure over the `x` parameter of
// the outer `makeAdder(..)`
var plusOne = makeAdder( 1 );

// `plusTen` gets a reference to the inner `add(..)`
// function with closure over the `x` parameter of
// the outer `makeAdder(..)`
var plusTen = makeAdder( 10 );

plusOne( 3 );		// 4  <-- 1 + 3
plusOne( 41 );		// 42 <-- 1 + 41

plusTen( 13 );		// 23 <-- 10 + 13
```

More on how this code works:

1. When we call `makeAdder(1)`, we get back a reference to its inner `add(..)` that remembers `x` as `1`. We call this function reference `plusOne(..)`.
2. When we call `makeAdder(10)`, we get back another reference to its inner `add(..)` that remembers `x` as `10`. We call this function reference `plusTen(..)`.
3. When we call `plusOne(3)`, it adds `3` (its inner `y`) to the `1` (remembered by `x`), and we get `4` as the result.
4. When we call `plusTen(13)`, it adds `13` (its inner `y`) to the `10` (remembered by `x`), and we get `23` as the result.

Don't worry if this seems strange and confusing at first -- it can be! It'll take lots of practice to understand it fully.

But trust me, once you do, it's one of the most powerful and useful techniques in all of programming. It's definitely worth the effort to let your brain simmer on closures for a bit. In the next section, we'll get a little more practice with closure.

#### Modules

The most common usage of closure in JavaScript is the module pattern. Modules let you define private implementation details (variables, functions) that are hidden from the outside world, as well as a public API that *is* accessible from the outside.

Consider:

```js
function User(){
	var username, password;

	function doLogin(user,pw) {
		username = user;
		password = pw;

		// do the rest of the login work
	}

	var publicAPI = {
		login: doLogin
	};

	return publicAPI;
}

// create a `User` module instance
var fred = User();

fred.login( "fred", "12Battery34!" );
```

The `User()` function serves as an outer scope that holds the variables `username` and `password`, as well as the inner `doLogin()` function; these are all private inner details of this `User` module that cannot be accessed from the outside world.

**Warning:** We are not calling `new User()` here, on purpose, despite the fact that probably seems more common to most readers. `User()` is just a function, not a class to be instantiated, so it's just called normally. Using `new` would be inappropriate and actually waste resources.

Executing `User()` creates an *instance* of the `User` module -- a whole new scope is created, and thus a whole new copy of each of these inner variables/functions. We assign this instance to `fred`. If we run `User()` again, we'd get a new instance entirely separate from `fred`.

The inner `doLogin()` function has a closure over `username` and `password`, meaning it will retain its access to them even after the `User()` function finishes running.

`publicAPI` is an object with one property/method on it, `login`, which is a reference to the inner `doLogin()` function. When we return `publicAPI` from `User()`, it becomes the instance we call `fred`.

At this point, the outer `User()` function has finished executing. Normally, you'd think the inner variables like `username` and `password` have gone away. But here they have not, because there's a closure in the `login()` function keeping them alive.

That's why we can call `fred.login(..)` -- the same as calling the inner `doLogin(..)` -- and it can still access `username` and `password` inner variables.

There's a good chance that with just this brief glimpse at closure and the module pattern, some of it is still a bit confusing. That's OK! It takes some work to wrap your brain around it.

From here, go read the *Scope & Closures* title of this series for a much more in-depth exploration.

## `this` Identifier

Another very commonly misunderstood concept in JavaScript is the `this` identifier. Again, there's a couple of chapters on it in the *this & Object Prototypes* title of this series, so here we'll just briefly introduce the concept.

While it may often seem that `this` is related to "object-oriented patterns," in JS `this` is a different mechanism.

If a function has a `this` reference inside it, that `this` reference usually points to an `object`. But which `object` it points to depends on how the function was called.

It's important to realize that `this` *does not* refer to the function itself, as is the most common misconception.

Here's a quick illustration:

```js
function foo() {
	console.log( this.bar );
}

var bar = "global";

var obj1 = {
	bar: "obj1",
	foo: foo
};

var obj2 = {
	bar: "obj2"
};

// --------

foo();				// "global"
obj1.foo();			// "obj1"
foo.call( obj2 );	// "obj2"
new foo();			// undefined
```

There are four rules for how `this` gets set, and they're shown in those last four lines of that snippet.

1. `foo()` ends up setting `this` to the global object in non-strict mode -- in strict mode, `this` would be `undefined` and you'd get an error in accessing the `bar` property -- so `"global"` is the value found for `this.bar`.
2. `obj1.foo()` sets `this` to the `obj1` object.
3. `foo.call(obj2)` sets `this` to the `obj2` object.
4. `new foo()` sets `this` to a brand new empty object.

Bottom line: to understand what `this` points to, you have to examine how the function in question was called. It will be one of those four ways just shown, and that will then answer what `this` is.

**Note:** For more information about `this`, see Chapters 1 and 2 of the *this & Object Prototypes* title of this series.

## Prototypes

The prototype mechanism in JavaScript is quite complicated. We will only glance at it here. You will want to spend plenty of time reviewing Chapters 4-6 of the *this & Object Prototypes* title of this series for all the details.

When you reference a property on an object, if that property doesn't exist, JavaScript will automatically use that object's internal prototype reference to find another object to look for the property on. You could think of this almost as a fallback if the property is missing.

The internal prototype reference linkage from one object to its fallback happens at the time the object is created. The simplest way to illustrate it is with a built-in utility called `Object.create(..)`.

Consider:

```js
var foo = {
	a: 42
};

// create `bar` and link it to `foo`
var bar = Object.create( foo );

bar.b = "hello world";

bar.b;		// "hello world"
bar.a;		// 42 <-- delegated to `foo`
```

It may help to visualize the `foo` and `bar` objects and their relationship:

<img src="fig6.png">

The `a` property doesn't actually exist on the `bar` object, but because `bar` is prototype-linked to `foo`, JavaScript automatically falls back to looking for `a` on the `foo` object, where it's found.

This linkage may seem like a strange feature of the language. The most common way this feature is used -- and I would argue, abused -- is to try to emulate/fake a "class" mechanism with "inheritance."

But a more natural way of applying prototypes is a pattern called "behavior delegation," where you intentionally design your linked objects to be able to *delegate* from one to the other for parts of the needed behavior.

**Note:** For more information about prototypes and behavior delegation, see Chapters 4-6 of the *this & Object Prototypes* title of this series.

## Old & New

Some of the JS features we've already covered, and certainly many of the features covered in the rest of this series, are newer additions and will not necessarily be available in older browsers. In fact, some of the newest features in the specification aren't even implemented in any stable browsers yet.

So, what do you do with the new stuff? Do you just have to wait around for years or decades for all the old browsers to fade into obscurity?

That's how many people think about the situation, but it's really not a healthy approach to JS.

There are two main techniques you can use to "bring" the newer JavaScript stuff to the older browsers: polyfilling and transpiling.

### Polyfilling

The word "polyfill" is an invented term (by Remy Sharp) (https://remysharp.com/2010/10/08/what-is-a-polyfill) used to refer to taking the definition of a newer feature and producing a piece of code that's equivalent to the behavior, but is able to run in older JS environments.

For example, ES6 defines a utility called `Number.isNaN(..)` to provide an accurate non-buggy check for `NaN` values, deprecating the original `isNaN(..)` utility. But it's easy to polyfill that utility so that you can start using it in your code regardless of whether the end user is in an ES6 browser or not.

Consider:

```js
if (!Number.isNaN) {
	Number.isNaN = function isNaN(x) {
		return x !== x;
	};
}
```

The `if` statement guards against applying the polyfill definition in ES6 browsers where it will already exist. If it's not already present, we define `Number.isNaN(..)`.

**Note:** The check we do here takes advantage of a quirk with `NaN` values, which is that they're the only value in the whole language that is not equal to itself. So the `NaN` value is the only one that would make `x !== x` be `true`.

Not all new features are fully polyfillable. Sometimes most of the behavior can be polyfilled, but there are still small deviations. You should be really, really careful in implementing a polyfill yourself, to make sure you are adhering to the specification as strictly as possible.

Or better yet, use an already vetted set of polyfills that you can trust, such as those provided by ES5-Shim (https://github.com/es-shims/es5-shim) and ES6-Shim (https://github.com/es-shims/es6-shim).

### Transpiling

There's no way to polyfill new syntax that has been added to the language. The new syntax would throw an error in the old JS engine as unrecognized/invalid.

So the better option is to use a tool that converts your newer code into older code equivalents. This process is commonly called "transpiling," a term for transforming + compiling.

Essentially, your source code is authored in the new syntax form, but what you deploy to the browser is the transpiled code in old syntax form. You typically insert the transpiler into your build process, similar to your code linter or your minifier.

You might wonder why you'd go to the trouble to write new syntax only to have it transpiled away to older code -- why not just write the older code directly?

There are several important reasons you should care about transpiling:

* The new syntax added to the language is designed to make your code more readable and maintainable. The older equivalents are often much more convoluted. You should prefer writing newer and cleaner syntax, not only for yourself but for all other members of the development team.
* If you transpile only for older browsers, but serve the new syntax to the newest browsers, you get to take advantage of browser performance optimizations with the new syntax. This also lets browser makers have more real-world code to test their implementations and optimizations on.
* Using the new syntax earlier allows it to be tested more robustly in the real world, which provides earlier feedback to the JavaScript committee (TC39). If issues are found early enough, they can be changed/fixed before those language design mistakes become permanent.

Here's a quick example of transpiling. ES6 adds a feature called "default parameter values." It looks like this:

```js
function foo(a = 2) {
	console.log( a );
}

foo();		// 2
foo( 42 );	// 42
```

Simple, right? Helpful, too! But it's new syntax that's invalid in pre-ES6 engines. So what will a transpiler do with that code to make it run in older environments?

```js
function foo() {
	var a = arguments[0] !== (void 0) ? arguments[0] : 2;
	console.log( a );
}
```

As you can see, it checks to see if the `arguments[0]` value is `void 0` (aka `undefined`), and if so provides the `2` default value; otherwise, it assigns whatever was passed.

In addition to being able to now use the nicer syntax even in older browsers, looking at the transpiled code actually explains the intended behavior more clearly.

You may not have realized just from looking at the ES6 version that `undefined` is the only value that can't get explicitly passed in for a default-value parameter, but the transpiled code makes that much more clear.

The last important detail to emphasize about transpilers is that they should now be thought of as a standard part of the JS development ecosystem and process. JS is going to continue to evolve, much more quickly than before, so every few months new syntax and new features will be added.

If you use a transpiler by default, you'll always be able to make that switch to newer syntax whenever you find it useful, rather than always waiting for years for today's browsers to phase out.

There are quite a few great transpilers for you to choose from. Here are some good options at the time of this writing:

* Babel (https://babeljs.io) (formerly 6to5): Transpiles ES6+ into ES5
* Traceur (https://github.com/google/traceur-compiler): Transpiles ES6, ES7, and beyond into ES5

## Non-JavaScript

So far, the only things we've covered are in the JS language itself. The reality is that most JS is written to run in and interact with environments like browsers. A good chunk of the stuff that you write in your code is, strictly speaking, not directly controlled by JavaScript. That probably sounds a little strange.

The most common non-JavaScript JavaScript you'll encounter is the DOM API. For example:

```js
var el = document.getElementById( "foo" );
```

The `document` variable exists as a global variable when your code is running in a browser. It's not provided by the JS engine, nor is it particularly controlled by the JavaScript specification. It takes the form of something that looks an awful lot like a normal JS `object`, but it's not really exactly that. It's a special `object,` often called a "host object."

Moreover, the `getElementById(..)` method on `document` looks like a normal JS function, but it's just a thinly exposed interface to a built-in method provided by the DOM from your browser. In some (newer-generation) browsers, this layer may also be in JS, but traditionally the DOM and its behavior is implemented in something more like C/C++.

Another example is with input/output (I/O).

Everyone's favorite `alert(..)` pops up a message box in the user's browser window. `alert(..)` is provided to your JS program by the browser, not by the JS engine itself. The call you make sends the message to the browser internals and it handles drawing and displaying the message box.

The same goes with `console.log(..)`; your browser provides such mechanisms and hooks them up to the developer tools.

This book, and this whole series, focuses on JavaScript the language. That's why you don't see any substantial coverage of these non-JavaScript JavaScript mechanisms. Nevertheless, you need to be aware of them, as they'll be in every JS program you write!

## Review

The first step to learning JavaScript's flavor of programming is to get a basic understanding of its core mechanisms like values, types, function closures, `this`, and prototypes.

Of course, each of these topics deserves much greater coverage than you've seen here, but that's why they have chapters and books dedicated to them throughout the rest of this series. After you feel pretty comfortable with the concepts and code samples in this chapter, the rest of the series awaits you to really dig in and get to know the language deeply.

The final chapter of this book will briefly summarize each of the other titles in the series and the other concepts they cover besides what we've already explored.
