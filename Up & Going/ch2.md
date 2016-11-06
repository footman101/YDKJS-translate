原文地址：[https://github.com/getify/You-Dont-Know-JS/blob/master/up%20&%20going/ch2.md](https://github.com/getify/You-Dont-Know-JS/blob/master/up%20&%20going/ch2.md)

# 你不知道的JS: 起步
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

第二个函数表达式是*具名的*（`bar`），即使它的引用也被赋给`x`变量。虽然*匿名函数表达式*仍然非常常见，但是*命名函数表达式*通常是更优的选择。

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

围绕函数表达式`(function IIFE(){ .. })`的外部`（..）`只是一个细微的JS语法，用来防止它被当作一个正常的函数声明。

在表达式末尾的最后一个`()` -- `})();`这行 -- 代表了实际执行紧跟在它之前的函数表达式。

这可能看起来很奇怪，但它并不是特别。在这里考虑`foo`和`IIFE`之间的相似之处：

```js
function foo() { .. }

// `foo` function reference expression,
// then `()` executes it
foo();

// `IIFE` function expression,
// then `()` executes it
(function IIFE(){ .. })();
```

正如你可以看到的，在执行`()`之前列出`（function IIFE（）{..}）`基本上与在执行`()`之前的`foo`相同; 在这两种情况下，函数的引用都是在紧跟其后的`()`中执行。

由于立即调用函数表达式只是一个函数，也会创造出*作用域*，使用这种形式可以在作用域内部定义一些变量，这些变量不会影响作用域的外部。

```js
var a = 42;

(function IIFE(){
	var a = 10;
	console.log( a );	// 10
})();

console.log( a );		// 42
```

立即调用函数表达式也可以返回值:

```js
var x = (function IIFE(){
	return 42;
})();

x;	// 42
```

命名函数`IIFE`执行并返回了值`42`，这个值被赋予给了`x`。

### 闭包

*闭包*是JavaScript中最重要的，通常也是不容易被理解的概念之一。 我不会在这里详细地介绍它，你可以参考本系列书的*作用域和闭包*主题。但我想说一些关于它，让你理解它的一般概念。它将会成为你的JS技能树中最重要的技术之一。

你可以认为闭包是“记住”并可以访问函数作用域（它的变量）的一种方式，甚至是在函数执行完毕后。

考虑:

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

内部函数`add(..)`的引用被外层函数`makeAdder(..)`返回，这个引用记住了外层`makeAdder(..)`函数的参数`x`。现在，我们来使用`makeAdder(..)`：

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

更多解释:

1. 我们调用`makeAdder(1)`后，得到它内部`add(..)`函数的引用，并且记住了`x`的值为`1`。我们将这个函数引用称为`plusOne(..)`。
2. 我们调用`makeAdder(10)`后，得到它内部`add(..)`函数的引用，并且记住了`x`的值为`10`。我们将这个函数引用称为`plusTen(..)`。
3. 我们调用`plusOne(3)`，它把`3`（内部的`y`）增加`1`（被记住的`x`），我们最终得到结果为`4`。
4. 我们调用`plusTen(13)`，它把`13`（内部的`y`）增加`10`（被记住的`x`），我们最终得到结果为`23`。

如果最初这些看起来似乎奇怪和混乱，不要担心 -- 它就是会这样！ 这种机制需要大量的实践来充分理解。

但相信我，一旦你这样做，它是编程中最强大和有用的技术之一。让你的大脑一点一点理解它，你的努力绝对值得。 在下一节中，我们将进一步练习闭包。

#### 模块

JavaScript中闭包最常见的用法是模块模式。模块允许你定义从外部无法直接访问的私有的实现细节（变量，函数），以及可从外部访问的公共API。

考虑:

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

`User()`作为外层作用域，包含`username`和`password`变量，以及内部函数`doLogin()`；它们都是`User`模块的内部实现细节，不能直接被外部世界访问。

**警告：** 我们在这里故意没有调用`new Use()`，虽然事实上这种形式对于很多读者更常见。 `User()`只是一个函数，而不是一个要实例化的类，所以它只是被正常调用。使用`new`是不合适的，并且事实更浪费资源。

执行`User()`创建了一个`User`模块的*实例* -- 一个新的作用域被创建了，于是全部这些内部变量和函数被复制了一份。我们将这个实例赋值给`fred`。如果我们再执行`User()`一遍，我们会得到一个和`fred`完全不同的实例。

内部的`doLogin()`函数对`username`和`password`有一个闭包，这意味着即使`User()`函数执行完毕后，它仍然可以访问它们。

`publicAPI`是一个对象，它有一个属性/方法，`login`，它是一个内部函数`doLogin()`的引用。当`User()`返回时，`publicAPI`变成了我们称为`fred`的实例。

此时，外部的`User()`函数已经执行完毕。 通常，你会认为像`username`和`password`这样的内部变量已经消失了。但是在这里，它们并没有，因为`login()`函数有一个闭包，保证了它们没有被销毁。

这就是为什么我们调用`fred.login(..)`时 -- 和调用内部函数`doLogin(..)`时一样，仍然可以访问内部变量`username`和`password`。

这是一个很好的机会，初试闭包和模块模式，其中有些东西仍然比较混乱。没关系！你的大脑要完全消化它尚需时日。

从这里开始，继续阅读本系列书的*作用域和闭包*主题，进行更深入的探索。

## `this`标识符

在JavaScript中另一个常常被误解的概念是`this`标识符。在这个系列书的*this和对象原型*主题中有几个章节会详细讲解，在这里我们将简要介绍这个概念。

虽然`this`可能经常与『面向对象模式』相关，但在JS中，`this`是一个不同的机制。

如果一个函数中有`this`引用，这个`this`引用通常指向一个`object`。但是这个`object`究竟是什么取决于函数的调用方式。

很重要的一点是，`this`*不是*指向这个函数本身，这是最常见的误解。

这里有一个快速说明：

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

这个代码段的最后四行展示了`this`被设置的四种情况

1. `foo()`中的`this`在非严格模式下是全局对象 -- 在严格模式下，`this`会变成`undefined`，当访问`bar`属性时会报错 -- 因此`"global"`是`this`的值。
2. `obj1.foo()`中的`this`是`obj1`。
3. `foo.call(obj2)`中的`this`是`obj2`。
4. `new foo()`中的`this`是一个新的空对象。

底线：要了解`this`指向什么，你必须明白函数究竟是如何调用的。调用方式一定是刚才展示的四种方式之一，然后你就可以回答`this`是什么了。

**注意：** 有关`this`的更多信息，请参阅本系列书的*this与对象原型*主题的第1章和第2章。

## 原型

JavaScript中的原型机制是非常复杂的。我们这里只是简单介绍看一下。你要想了解全部细节，需要花费大量的时间复习本系列书的*this和对象原型*的4-6章。

当你访问一个对象的某个属性时，如果这个属性不存在,JavaScript会自动在对象的内部原型对象上继续查找。你可以认为这是一种在属性缺失时的后备机制。

对象与内部原型对象的连接关系发生在对象创建时，最简单的实现方式是使用内置的`Object.create(..)`方法。

考虑:

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

`foo`和`bar`对象以及它们之间的关系图像化说明如下：

[![](https://github.com/getify/You-Dont-Know-JS/raw/master/up%20&%20going/fig6.png)](https://github.com/getify/You-Dont-Know-JS/blob/master/up%20&%20going/fig6.png)

`a`属性并不是真正存在于`bar`对象上，但是由于`bar`通过原型链连接到`foo`，JavaScript自动在`foo`对象上查找`a`，并且最终可以找到。

原型链看起来是一种古怪的语言特性。使用这个特性的最常见的方法 -- 我认为是滥用 -- 是试图用『继承』来模拟/伪造『类』机制。

但是一种更自然的应用原型的方式是『行为委托』模式，在这种模式下，你可以有意识地设计你的链接对象，以便把一部分需要的行为『委托』给另外的对象。

**注意：** 更多关于原型和行为委托的信息，请参看本系列书的*this和对象原型*的4-6章.

## 旧与新

我们已经介绍的一些JS特性，以及本系列其余部分中介绍的许多特性，都是较新的特性，并不一定在旧版本的浏览器中可用。事实上，规范中的一些最新功能甚至还没有在任何稳定的浏览器中实现。

那么，这么多新的东西你要怎么做？你只是等待十年或几十年，等所有的旧浏览器数退出历史舞台？

这是很多的想法，但对于JS来说它真的不是一个好的想法。

有两种主要的技术可以在旧的浏览器上使用新的JavaScript特性：polyfilling和transpiling（后面两节会解释这两个词）。

### Polyfilling

词语『polyfill』是新发明的术语（由Remy Sharp提供）（https://remysharp.com/2010/10/08/what-is-a-polyfill），用于指代采用较新特性的定义，写出一段等价的代码，使之能够在旧的JS环境中运行。

例如，ES6中定义了`Number.isNaN(..)`函数，用于精准地检查是否是`NaN`值，废弃了原有的`isNaN(..)`函数。但是很容易填充这个新的函数，以便您可以开始在代码中使用它，无论最终用户是否在ES6浏览器。

考虑：

```js
if (!Number.isNaN) {
	Number.isNaN = function isNaN(x) {
		return x !== x;
	};
}
```

`if`语句保证了在存在`Number.isNaN(..)`函数的ES6浏览器中不使用填充定义，如果浏览器中不存在时，我们才定义`Number.isNaN(..)`。

**注意：** 在上述我们定义的检查中，我们利用了`NaN`的一个怪癖，即它是真个语言中唯一自身不等于自身的值。因此`NaN`是唯一一个能够保证`x !== x`为`true`的值。

并不是所有的新特性可以被完全填充。有时大多数行为可以被填充，但仍有小的偏差。你应该非常小心自己实现的polyfill，确保严格遵守规范。

或者更好的是，使用已经经过检验的，您可以信任的一组polyfills，如ES5-Shim（https://github.com/es-shims/es5-shim）和ES6-Shim（https://github.com/es-shims/es6-shim）。

### Transpiling

对于语言的新语法，是没有办法polyfill的。新的语法将在旧的JS引擎中抛出无法识别/无效错误。

因此一个更好的选择是，使用工具将新语法转换为旧的代码实现。这个过程通常被称为『transpiling』，是转换（transforming）和编译（compiling）的合体。

实质上，您的源代码是以新的语法写的，但最终部署到浏览器的是以旧语法形式转换后的代码。转换过程通常实在构建过程中，类似于代码检查或代码压缩。

你可能想知道为什么要麻烦来写新的语法然后转换为老的代码 -- 为什么不直接写老代码？

这里有几个重要的原因，关于为什么你要关注transpiling：

* 新加入语言的语法在设计之初，是为了让你的代码更加具有可读性和可维护性。老代码的等价物通常比较错综复杂。你应该倾向于写更新更干净的语法，不仅仅为了你自己，更是为了开发组的其他成员。
* 如果仅仅为了旧的浏览器来transpile，而在新的浏览器上直接使用新的语法，那么你就可以利用新浏览器针对新语法的优化来提高代码性能。同时，浏览器制造者也可以获得更多的代码来测试他们的新语法实现和优化。
* 更早地使用新语法，可以使这些代码经过更加可靠的测试，这为JavaScript委员会（TC39）提供了早期反馈。 如果问题足够早，可以在这些语言设计错误定型前进行更改/修改。

这里有个transpiling的例子。ES6增加了称为『默认参数值』,它看起来是这样的：

```js
function foo(a = 2) {
	console.log( a );
}

foo();		// 2
foo( 42 );	// 42
```

很简单，是吗？同时也非常有用！但是它在ES6前的语言引擎中是非法的。那么转换器做了什么可以让它在旧浏览器中运行？

```js
function foo() {
	var a = arguments[0] !== (void 0) ? arguments[0] : 2;
	console.log( a );
}
```

正如你所看到的，首先检查`arguments[0]`是否是`void 0`（即`undefined`），如果是的话，将`2`作为默认值赋给它；否则，把传进来的参数赋给它。

除了能够在旧浏览器使用新语法外，查看转换后的代码能够是我们对于语法的预期行为更加清楚。

你可能没有注意到，在ES6中`undefined`是唯一不能显式传递给默认参数的值，但是查看转换后的代码，你就会清楚这一点。

需要强调的最后一个转换器重要细节是，转化器应该被认为是JS开发生态系统和过程的一个标准部分。JS将继续越来越快地发展，因此每几个月就会添加新语法和新特性。

如果你默认使用转换器，无论何时你觉得新语法有用时就可以使用它，而不是等待现在旧的浏览器慢慢过时，这可能需要好多年。

这里有很多非常好的转换器供你选择。此时此刻有以下好的选择：

* Babel (https://babeljs.io) (以前的6to5): 转换ES6+至ES5
* Traceur (https://github.com/google/traceur-compiler): 转换ES6，ES7及之外至ES5

## 非JavaScript

到目前为止，我们所覆盖的唯一的东西是在JS语言本身。现实是大多数JS是在浏览器等环境中运行和交互。在你的代码中写的很多东西，严格来说，不是由JavaScript直接控制。这可能听起来有点奇怪。

您会遇到的最常见的非JavaScript的JavaScript是DOM API。例如：

```js
var el = document.getElementById( "foo" );
```

`document`变量是浏览器中的一个全局变量。它不是由JS引擎提供的，也不受JavaScript规范控制。它的形式看起来像一个普通的JS对象，但它不是真的那样。它是一个特殊的`object`，通常称为『宿主对象』。

此外，`document`上的`getElementById(..)`方法看起来是一个JS函数，但是它仅仅是一个浏览器DOM内置方法的接口。在有些（新一代）浏览器中，这一层函数有可能也是由JS实现的，但是传统上，DOM以及它的行为函数更多由C/C++实现。

另一个例子是关于输入/输出（I/O）。

每个人都喜爱的`alert(..)`函数会在用户浏览器窗口弹出一个消息盒子。浏览器的JS程序提供了`alert(..)`方法，而不是JS引擎本身。你调用这个函数时，会发送消息给浏览器内部，浏览器会绘制和显示这个消息盒子。

`console.log(..)`也是同样的道理；你的浏览器提供了这种机制，并将它们hook（	
钩）到开发者工具。

这本书，以及整个系列书，聚焦于JavaScript语言本身。这就是为什么你看不到这些非JavaScript的JavaScript的大量介绍。然而，你需要知道它们，因为它们会在你写的每个JS程序中！

## 复习

学习JavaScript风格编程的第一步是学习其核心机制（如值，类型，函数闭包，`this`和原型）的基本内容。

当然，这里你所看到关于这些主题的介绍是远远不够的，这就是为什么本系列还有其他章节和其他书来介绍它们。一旦你掌握了本章所讲的概念了代码实例，本系列书的其他部分等待你进一步去挖掘，你将会更深入地理解这门语言。

本书的最后一章将会简单总结本系列其他书的主题以及我们已经探讨过之外的一些其他概念。