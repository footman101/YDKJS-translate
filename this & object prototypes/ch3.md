# 你不知道的JS：*this* & 对象原型
# 第三章：对象

在第一章和第二章，我们解释了`this`指向的对象取决于函数调用的调用点。但是到底什么是对象？为什么我们需要指向它们？我们会在这章解释对象的细节。

## 语法

对象来自两种形式：定义（字面量）形式和构造形式。

对象的字面量语法看起来像这样：

```js
var myObj = {
	key: value
	// ...
};
```

构造形式看起来像这样：

```js
var myObj = new Object();
myObj.key = value;
```

构造形式和字面量形式产生的对象是一样的。唯一的不同就是你可以在字面量定义时天健一个或多个键值对，而在构造形式下，你必须一个个地添加属性。

**注意：** 使用『构造形式』来创建对象，如上所示，是非常不常见的。你可能经常会使用字面量形式。这对于内置对象（见下文）来说同样适用。

## 类型

对象是构建JS的基本的构建块。它们是JS中的6种基本类型（在规范中称为“语言类型”之一）：

* `string`
* `number`
* `boolean`
* `null`
* `undefined`
* `object`

注意*简单基本类型*(`string`,`number`,`boolean`,`null`和`undefined`)本身**不是**`objects`。`null`有时会指向一个对象类型，但是这个误解源自于语言的一个bug，即`typeof null`会错误地（并且令人疑惑地）返回字符串`"object"`。实际上，`null`本身就是一个基本类型。

**有一个常见的错误是『JavaScript的所有东西都是对象』。这明显不是真的。**

相反，*有*一些特殊的对象子类型，我们可以当作是*复杂的基本类型*。

`function`是对象的子类型（技术上，是“可调用对象”）。JS中的函数被称为"一等公民"，因为它们本质上只是普通对象（具有可调用行为语义），因此它们可以像任何其他普通对象一样处理。

数组也是一种对象形式，具有额外的行为。数组中的内容组织比一般对象更有条理。

### 内置对象

有一些其他的对象子类型，它们通常被当做内置对象引用。对于它们中的一些来说，它们的名字似乎暗示着和对应的简单基本类型有关，但实际上，它们的关系更加复杂，我们回随后介绍。

* `String`
* `Number`
* `Boolean`
* `Object`
* `Function`
* `Array`
* `Date`
* `RegExp`
* `Error`

如果你依照和其他语言的相似性来看的话，比如 Java 语言的`String`类，这些内建类型有着实际类型的外观，甚至是类（class）的外观。

但是在JS中，它们实际上只是内置的函数。每一个这些内置的函数都可以当做构造函数使用（即，使用`new`操作符调用函数 -- 见第二章），这样的结果就是一个新*构造*的相应的子类型对象。例如：

```js
var strPrimitive = "I am a string";
typeof strPrimitive;							// "string"
strPrimitive instanceof String;					// false

var strObject = new String( "I am a string" );
typeof strObject; 								// "object"
strObject instanceof String;					// true

// inspect the object sub-type
Object.prototype.toString.call( strObject );	// [object String]
```

我们将在后面的章节中详细讨论`Object.prototype.toString...`是如何工作的，但简单来说，我们可以通过借用基本的默认`toString()`方法检查内部子类型，你可以看到它显示`strObject`是一个实际上由`String`构造函数创建的对象。

原始值`"I am a string"`不是一个对象，它是一个原始字面量和不可变的值。要对其执行操作，例如检查其长度，访问其各个字符内容等，需要一个`String`对象。

幸运的是，语言会在必要时自动将`"string"`基本类型强制转换为`String`对象，这意味着你几乎不需要显式创建对象形式。JS社区主流都**强烈倾向**于在任何可能的情况下使用字面量作为值，而不是构造对象形式。

考虑：

```js
var strPrimitive = "I am a string";

console.log( strPrimitive.length );			// 13

console.log( strPrimitive.charAt( 3 ) );	// "m"
```

在这两种情况下，我们在字符串字面量上调用属性或方法，引擎会自动将其强制转换为`String`对象，以便属性/方法访问起作用。

当使用诸如`42.359.toFixed(2)`之类的方法时，在数字字面量`42`和`new Number(42)`对象包装器之间发生同样的强制转换。对于来自`“boolean”`基本类型和`Boolean`对象也是如此。

`null`和`undefined`没有对象包装器形式，只有它们的原始值。 相比之下，`Date`值*只能*用它们的构造对象形式创建，因为它们没有对应的字面量。

无论是使用字面量形式还是构造形式，`Object`，`Array`，`function`和`RegExp`（正则表达式）都是对象。 在某些情况下，构造形式确实提供了比字面量形式更多的创建选项。由于对象是可以以任一方式创建的，因此更简单的字面量形式基本上通常是优先考虑的。**如果你需要额外选项，请仅使用构造形式。**

`Error`对象很少在代码中显式创建，但通常在抛出异常时自动创建。它们可以使用构造的形式`new Error(..)`创建，但这通常是不必要的。

## 内容

如前所述，对象的内容由存储在特定名称*位置*的值（任何类型）组成，我们称之为属性。

重要的要注意的事，虽然我们当说“内容”时，意味着这些值*实际上*存储在对象内部，但这只是一个表象。引擎以依赖于实现的方式存储值，并且很可能不会将它们存储*在*某个对象容器中。存储在容器中的*是*这些属性名称，它们作为存储值的指针（技术上，*引用*）存在。

考虑：

```js
var myObject = {
	a: 2
};

myObject.a;		// 2

myObject["a"];	// 2
```

要访问`myObject`*位于*`a`的值，我们需要使用`.`操作符或者`[ ]`操作符。`.a`语法通常表示『属性』访问，而`["a"]`语法通常表示『键』访问。实际上，它们都访问了同一个*位置*，然后取出来同样的值，`2`，所以它们可以互换使用。从现在起，我们将会使用最常见的语法，『属性访问』。

两种语法之间的主要区别在于`.`运算符需要一个兼容`标识符`的属性名称，而`[“..”]`语法基本上可以采用任何兼容UTF-8/unicode的字符串作为属性名称。例如，要引用名称“Super-Fun!”的属性，你必须使用`[“Super-Fun!”]`访问语法，因为`Super-Fun!`不是有效的`标识符`属性名称。

同时，由于`[".."]`语法使用一个字符串**值**来定位，这意味着程序可以程序化地构造这个字符串的值，例如：

```js
var wantA = true;
var myObject = {
	a: 2
};

var idx;

if (wantA) {
	idx = "a";
}

// later

console.log( myObject[idx] ); // 2
```

在对象中，属性名称**通常**都是字符串。如果除了使用`string`（基本类型）之外的任何其他值作为属性，它将首先转换为字符串。这甚至包括通常用作数组索引的数字，因此请注意不要混淆对象和数组之间的数字使用。

```js
var myObject = { };

myObject[true] = "foo";
myObject[3] = "bar";
myObject[myObject] = "baz";

myObject["true"];				// "foo"
myObject["3"];					// "bar"
myObject["[object Object]"];	// "baz"
```

### 计算属性名称

如果你需要使用计算表达式值*作为*键名，比如`myObject[prefix + name]`，我们刚才描述的`myObject[..]`属性访问语法就很有用。但是在使用对象字面量语法声明对象时，这是不行的。

ES6添加了*计算属性名*，你可以在对象字面量声明的键名位置指定一个由`[ ]`对包裹的表达式：

```js
var prefix = "foo";

var myObject = {
	[prefix + "bar"]: "hello",
	[prefix + "baz"]: "world"
};

myObject["foobar"]; // hello
myObject["foobaz"]; // world
```

*计算属性名称*的最常见用法可能是ES6`Symbol`，我们在本书中不会详细介绍它。简而言之，它们是一种新的原始数据类型，它具有不透明的不可猜测的值（技术上是一个`string`值）。强烈建议你不要使用`Symbol`的*实际值*（理论上在不同JS引擎之间可能不同），因此你只能使用`Symbol`的名称，如`Symbol.Something`（只是一个伪装的名字！）：

```js
var myObject = {
	[Symbol.Something]: "hello world"
};
```

### 属性 vs. 方法

在谈论对象的属性访问时，如果被访问的值恰好是一个函数，一些开发人员喜欢区别对待。因为很容易将函数视为*属于*这个对象，而在其他语言中，属于对象的函数（又称“类”）被称为“方法”，所以相对于『属性访问』，听到『方法访问』并不罕见。

有趣的是，**规范中有同样的区别**。

从技术上讲，函数永远不会“属于”对象，所以说恰好碰巧在对象引用上访问的函数自动是一个“方法”，似乎有些牵强附会。

有些函数*确实*在它们内部有`this`引用，*有时*这些`this`引用在调用点使用了对象引用。但是这种用法实际上不会使该函数成为一个“方法”而与任何其他函数不同，因为`this`在运行时动态绑定的，取决于调用点，因此它最多与对象有间接关系。

每次访问对象上的属性时，都是**属性访问**，无论你得到的值是什么类型。如果你*碰巧*从该属性访问中获取一个函数，那么它并不是一个神奇的“方法”。关于来自属性访问的函数，没有什么特别的（除了前面解释的可能的隐式`this`绑定之外）。

例如：

```js
function foo() {
	console.log( "foo" );
}

var someFoo = foo;	// variable reference to `foo`

var myObject = {
	someFoo: foo
};

foo;				// function foo(){..}

someFoo;			// function foo(){..}

myObject.someFoo;	// function foo(){..}
```
`someFoo`和`myObject.someFoo`只是对同一个函数的两个独立引用，并且都没有暗示该函数的特殊性或任何其他对象“拥有”该函数。如果上面的`foo()`定义为在其内部使用了`this`引用，那么`myObject.someFoo` *隐式绑定*将是两个引用之间**唯一**可观察的差异。这两种引用方式都没有理由被称为“方法”。

**也许有人可能会争辩说*函数*变成了一个方法*，不是在定义时，但是在运行期间调用时，取决于它在调用点上的调用方式（是否带有对象引用上下文 - 有关详细信息，请参阅第2章）。即使这种解释也有点牵强。

最安全的结论可能是“函数”和“方法”在JavaScript中是可互换的。

**注意：** ES6添加了一个`super`引用，通常与`class`一起使用（参见附录A）。`super`的行为方式（静态绑定而不是后期绑定为`this`）进一步强调了这样一种观点，即在某处`super`绑定的函数更像是“方法”而不是“函数”。但同样，这些只是微妙的语义（和机械）细微差别。

即使你将一个函数表达式声明为对象字面量的一部分，该函数也不会神奇地*属于*对象 - 仍然只是对同一个函数对象的多个引用：

```js
var myObject = {
	foo: function foo() {
		console.log( "foo" );
	}
};

var someFoo = myObject.foo;

someFoo;		// function foo(){..}

myObject.foo;	// function foo(){..}
```

**注意：** 在第六章，我们会介绍ES6中对象字面量的`foo: function foo(){ .. }`定义语法的缩写形式。

### 数组

数组也使用`[ ]`访问形式，但如上所述，它们存储值的方式和位置（尽管对存储值的*类型*没有限制）更具结构化。数组使用*数字索引*，这意味着值存储在非负整数的位置，通常称为*索引*，例如`0`和`42`。

```js
var myArray = [ "foo", 42, "bar" ];

myArray.length;		// 3

myArray[0];			// "foo"

myArray[2];			// "bar"
```

数组*是*对象，因此即使每个索引都是正整数，你*也*可以在数组中添加属性：

```js
var myArray = [ "foo", 42, "bar" ];

myArray.baz = "baz";

myArray.length;	// 3

myArray.baz;	// "baz"
```

注意添加命名的属性（无论使用`.`或者`[ ]`操作符语法）不会改变数组呈现的`length`。

你*可以*使用数组作为普通键/值对象，并且永远不会添加任何数字索引，但这是一个坏主意，因为数组额外具有用于特定预期用途的行为和优化。使用对象存储键/值对，使用数组存储数值索引处的值。

**注意：** 如果你尝试向数组添加属性，但属性名称*看起来*像一个数字，它将最终成为数字索引（从而修改数组内容）：

```js
var myArray = [ "foo", 42, "bar" ];

myArray["3"] = "baz";

myArray.length;	// 4

myArray[3];		// "baz"
```

### 复制对象

当开发人员刚开始使用JavaScript语言时，最常需要的功能之一是如何复制对象。看起来应该有一个内置的`copy()`方法，对吧？事实证明它比这更复杂，因为默认情况下，复制算法应该是什么并不完全清晰。

例如，考虑这个对象：

```js
function anotherFunction() { /*..*/ }

var anotherObject = {
	c: true
};

var anotherArray = [];

var myObject = {
	a: 2,
	b: anotherObject,	// reference, not a copy!
	c: anotherArray,	// another reference!
	d: anotherFunction
};

anotherArray.push( anotherObject, myObject );
```

到底什么应该是`myObject`的*拷贝*的表示？

首先，我们应该回答它是否应该是*浅*或*深*拷贝。一个*浅拷贝*最终会在新对象上`a`的值为`2`，但是`b`，`c`和`d`属性只是引用，并且引用在原始对象中相同的位置。一个*深拷贝*不仅会复制`myObject`，而且会复制`anotherObject`和`anotherArray`。但是这样我们遇到的问题是`anotherArray`引用了`anotherObject`和`myObject`，所以*那些*也应该复制而不只是保留引用。这样由于循环引用，我们就会有一个无限循环复制问题。

我们应该检测循环引用，并且只是打破循环遍历（使深层元素不完全复制）吗？我们应该抛出错误吗？还是介于两者之间？

而且，一个函数的“复制”意味着什么并不是很清楚？有一些hack行为喜欢使用函数源码的`toString()`序列化（在引擎各个实现中各不相同，甚至不可靠，具体取决于被检查的函数类型）。

那么我们如何解决所有这些棘手的问题呢？各种JS框架各自做出了自己的解释与决定。但是哪些（如果有的话）是JS可以采用当做标准？在很长一段时间，并没有明确的答案。

一个子集解决方案对于JSON安全的对象（即，可以序列化为JSON字符串，然后重新解析为具有相同结构和值的对象）可以轻松地*复制*：

```js
var newObj = JSON.parse( JSON.stringify( someObj ) );
```

当然，这需要你确保对象是JSON安全的。在某些情况下，这是可以试试的。对于其他情况来说，这还不够。

同时，浅拷贝是可以理解的，并且问题少得多，所以ES6现在为此任务定义了`Object.assign(..)`。 `Object.assign(..)`将*目标*对象作为其第一个参数，并将一个或多个*源*对象作为其后续参数。它遍历*源*对象上的所有*可枚举*的（见下文），*自身拥有的键*（**直接体现**），并将它们（仅通过`=`赋值）复制到*目标*对象上。 最终返回*目标*对象，如下所示：

```js
var newObj = Object.assign( {}, myObject );

newObj.a;						// 2
newObj.b === anotherObject;		// true
newObj.c === anotherArray;		// true
newObj.d === anotherFunction;	// true
```

**注意：** 在下一节中，我们介绍“属性描述符”（属性特征）并显示`Object.defineProperty(..)`的使用。然而，`Object.assign(..)`的复制纯粹通过`=`赋值，因此源对象上属性的任何特殊特征（如`writable`）**都不会保留**在目标上。

### 属性描述符

在ES5之前，JavaScript语言没有为代码提供直接的方法来检查或描述属性特征之间的区别，例如属性是否为只读。

但在ES5，所有的属性可以用**属性描述符**描述。

考虑这段代码：

```js
var myObject = {
	a: 2
};

Object.getOwnPropertyDescriptor( myObject, "a" );
// {
//    value: 2,
//    writable: true,
//    enumerable: true,
//    configurable: true
// }
```

正如你所看到的，属性描述符（称为“数据描述符”，因为它仅用于保存数据值）对于我们的普通对象属性`a`不仅仅只有它的`值2`。它还包括3个其他特征：`writable`，`enumerable`和`configure`。

虽然我们可以看到，在创建普通属性时属性描述符具有默认值，但我们可以使用`Object.defineProperty(..)`添加新属性，或按照所需修改现有属性（如果它是`configurable`的！）。

例如：

```js
var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: true,
	configurable: true,
	enumerable: true
} );

myObject.a; // 2
```

使用`defineProperty(..)`，我们以手动显式方式将简单的普通属性`a`添加到`myObject`。但是，除非你想要从其正常行为中修改其中一个描述符特征，否则通常不会使用此手动方法。

#### 可写性（Writable）

控制你是否可以修改属性值能力的是`可写性`。

考虑：

```js
var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: false, // not writable!
	configurable: true,
	enumerable: true
} );

myObject.a = 3;

myObject.a; // 2
```

你可以看到，我们对于`值`得修改默默地失败了。如果我们是在`严格模式`下尝试，我们会得到一个错误：

```js
"use strict";

var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: false, // not writable!
	configurable: true,
	enumerable: true
} );

myObject.a = 3; // TypeError
```

这个`TypeError`告诉我们不能修改一个非可写属性。

**注意：** 我们稍后将会讨论getters/setters，但是简单地讲，你可以看到`writable:false`意味着一个值不能改变，这从某种意义上等同于你定义了一个没有操作的setter。实际上，你的没有操作的setter需要在调用时抛出`TypeError`，这样才等价于`writable:false`。

#### 可配置性（Configurable）

只要属性当前是可配置的，我们就可以使用相同的`defineProperty(..)`实用程序修改其描述符定义。

```js
var myObject = {
	a: 2
};

myObject.a = 3;
myObject.a;					// 3

Object.defineProperty( myObject, "a", {
	value: 4,
	writable: true,
	configurable: false,	// not configurable!
	enumerable: true
} );

myObject.a;					// 4
myObject.a = 5;
myObject.a;					// 5

Object.defineProperty( myObject, "a", {
	value: 6,
	writable: true,
	configurable: true,
	enumerable: true
} ); // TypeError
```

如果你尝试将描述符定义更改不可配置属性，而不论是否在`严格模式`下，则最后的`defineProperty(..)`调用会导致TypeError。小心：正如你所看到的，将`configurable`改为`false`是一种**单向动作，无法撤消！**

**注意：** 有一个细节要注意：即使属性已经是`configurable:false`，`writable`也是可以从`true`更改为`false`而不会出错，但如果已经为`false`，则不能回到`true`。

另一个`configurable:false`阻止的是使用`delete`运算符来删除现有属性。

```js
var myObject = {
	a: 2
};

myObject.a;				// 2
delete myObject.a;
myObject.a;				// undefined

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: true,
	configurable: false,
	enumerable: true
} );

myObject.a;				// 2
delete myObject.a;
myObject.a;				// 2
```

你可以看到，最后的`delete`调用失败了（默默地），因为我们将`a`属性改成了不可配置。

`delete`仅用于直接从相关对象中删除对象属性（可以删除的）。如果一个对象属性是某个对象/函数的最后的*引用*，你`delete`属性就是删除了引用，那么现在就可以对未引用的对象/函数进行垃圾回收。但是，像其他语言（如C/C ++）那样将`delete`视为释放已分配内存的工具并**不**正确。`delete`只是一个对象属性的删除操作 - 仅此而已。

#### 可枚举（Enumerable）

我们将在这里提到的最后一个描述符特征（还有另外两个，我们随后在讨论getter/setter时介绍）是`可枚举`。

这个名称的含义可能很明显，但是这个特性控制了属性是否会出现在对象属性枚举中，例如`for..in`循环。设置为`false`就不会出现在这些枚举中，即使它仍然完全可访问。设置为`true`则会出现。

所有普通的用户定义属性都默认为`可枚举`，因为这通常是你想要的。但是如果你想要隐藏枚举的特殊属性，请将其设置为`enumerable:false`。

我们稍后将更详细地演示可枚举性，因此请保留关于此主题的精神书签。

### 不变性（Immutability）

有时需要创建无法改变的属性或对象（无论是无意还是有意）。ES5增加了多种处理支持，它们之间有细微的差别。

重要的是要注意，**所有**的这些方法只会产生浅不变性。也就是说，它们仅影响对象及其直接属性特征。如果一个对象具有对另一个对象（数组，对象，函数等）的引用，则该对象的*内容*不会受到影响，并且仍然是可变的。

```js
myImmutableObject.foo; // [1,2,3]
myImmutableObject.foo.push( 4 );
myImmutableObject.foo; // [1,2,3,4]
```

我们假设在这个片段中，`myImmutableObject`已经被创建并保护为不可变的。但是，要保护`myImmutableObject.foo`（它自己的对象 - 数组）的内容，你还需要使用以下一个或多个工具使`foo`成为不可变的。

**注意：** 在JS程序中创建深不可变对象并不常见。特殊情况下当然可以要求这样做，但作为一般设计模式，如果你发现自己想要*seal*或*freeze*所有对象，你可能需要退后一步并重新考虑你的程序设计，以使当对象值可能潜在变化时更加健壮。

#### 对象常量（Object Constant）

联合使用`writable:false`和`configurable:false`，你可以创建一个*常量*（不能被改变，重新定义或者删除）作为一个对象的属性，例如：

```js
var myObject = {};

Object.defineProperty( myObject, "FAVORITE_NUMBER", {
	value: 42,
	writable: false,
	configurable: false
} );
```

#### 阻止扩展（Prevent Extensions）

如果你想阻止在一个对象上新增属性，但是可以对对象属性做其他操作，调用`Object.preventExtensions(..)`：

```js
var myObject = {
	a: 2
};

Object.preventExtensions( myObject );

myObject.b = 3;
myObject.b; // undefined
```

在`非严格模式`下,`b`的创建默默地失败。在`严格模式`下，它会抛出`TypeError`。

#### 密封（Seal）

`Object.seal(..)`创建一个“密封”对象，这意味着它在一个现有的对象上调用`Object.preventExtensions(..)`，并且也将它的所有现有属性标记为`configurable:false`。

因此，你不仅不能再添加任何属性，而且还无法重新配置或删除任何现有属性（尽管你仍然*可以*修改其值）。

#### 冻结（Freeze）

`Object.freeze(..)`创建一个冻结的对象，这意味着它在一个现有的对象上调用`Object.seal(..)`，并且也将所有“数据访问器”属性标记为`writable:false`，这样它们的值就无法改变。

这种方法是对象本身可以达到的最高级别的不变性，因为它可以防止对对象或其任何直接属性的任何更改（但是，如上所述，任何引用的其他对象的内容都不受影响）。

你可以通过在对象上调用`Object.freeze(..)`，然后递归迭代它引用的所有对象（到目前为止本来不受影响），然后对引用调用`Object.freeze(..)`，来“深度冻结”一个对象。但要小心，因为这可能会影响你不打算影响的其他（共享）对象。

### `[[Get]]`

关于属性访问的行为，有一个微妙但重要的细节。

考虑：

```js
var myObject = {
	a: 2
};

myObject.a; // 2
```

`myObject.a`是一次属性访问，但是它并不*仅仅只是*查看`myObject`中名称为`a`的属性。

根据规范，上面这段代码实际上在`myObject`上执行了一个`[[Get]]`操作（类似一个函数调用：`[[Get]]()`）。一个对象上内置默认的`[[Get]]`操作*首先*检查对象上所需名称的属性，如果找到了，它会返回相应的值。

但是，如果`[[Get]]`算法*没有*找到所请求名称的属性，它还定义了其他重要的行为。我们将在第5章中讨论*接下来*（遍历`[[Prototype]]`链，如果有的话）会发生什么。

但是`[[Get]]`操作的一个重要结果是，如果它不能通过任何方式为请求的属性找到值，它将返回`undefined`。

```js
var myObject = {
	a: 2
};

myObject.b; // undefined
```

此行为与通过标识符名称引用*变量*时不同。如果该引用在适用的词法作用域查找中无法解析变量，则结果不会像对象属性那样是`undefined`，而是抛出`ReferenceError`错误。

```js
var myObject = {
	a: undefined
};

myObject.a; // undefined

myObject.b; // undefined
```

从*值*的角度来看，这两个引用之间没有区别 -- 它们结果都是`undefined`。然而，`[[Get]]`操作的内部实现，虽然微妙但一目了然，对于引用`myObject.b`而言可能比引用`myObject.a`执行了更多的“操作”。

只检查值结果，你无法区分是否是属性存在并保持显式值`undefined`，或者属性*不*存在，`undefined`是`[[Get]]`获取显式值失败后的默认返回值。但是，我们很快就会看到*如何*区分这两种情况。

### `[[Put]]`

由于有一个内部定义的`[[Get]]`操作用于从属性获取值，因此显然还有一个默认的`[[Put]]`操作。

可能很容易认为对对象的属性赋值，只是调用`[[Put]]`来设置或创建对象上的属性。但实际情况比这更微妙。

当调用`[[Put]]`时，它的行为方式因许多因素而异，包括（影响最大）属性是否已存在于对象上。

如果属性存在，`[[Put]]`算法将粗略检查：

1.属性是否为访问器描述符（请参阅下面的“Getters & Setters”部分）？**如果是，调用setter，如果有的话。**
2.属性是否为`writable`为`false`的数据描述符？**如果是这样，在`非严格模式`中静默失败，或在`严格模式`中抛出`TypeError`。**
3.否则，如正常所为将值设置给现有属性。

如果该对象上尚未出现该属性，则`[[Put]]`操作更加微妙和复杂。我们将在第5章当我们讨论`[[Prototype]]`时重新讨论这个场景，那时会更加清晰。

### Getters & Setters

对象的默认`[[Put]]`和`[[Get]]`操作分别完全控制如何将值设置为现有属性或新属性，以及从现有属性中取值。

**注意：** 使用语言的未来/高级功能，可以覆盖整个对象（不仅仅是每个属性）的默认`[[Get]]`或`[[Put]]`操作。这超出了我们在本书中讨论的范围，但稍后将在“你不了解的JS”系列中介绍。

ES5引入了一种方法来覆盖部分这些默认操作，不是在对象级别，而是在每个属性级别通过使用getter和setter来实现。Getter是调用隐藏函数来获取值的属性。Setter是调用隐藏函数来设置值的属性。

当你将属性定义为具有getter或setter或两者都有时，其定义就成为“访问器描述符”（而不是“数据描述符”）。对于访问器描述符，描述符的`value`和`writable`属性将没有实际意义并被忽略，而JS会使用属性的`set`和`get`属性（以及`configurable`和`enumerable`） 。

考虑：

```js
var myObject = {
	// define a getter for `a`
	get a() {
		return 2;
	}
};

Object.defineProperty(
	myObject,	// target
	"b",		// property name
	{			// descriptor
		// define a getter for `b`
		get: function(){ return this.a * 2 },

		// make sure `b` shows up as an object property
		enumerable: true
	}
);

myObject.a; // 2

myObject.b; // 4
```

通过使用对象字面量语法`get a() {..}`或通过`defineProperty(..)`的显式定义，在这两种情况下我们都在对象上创建了一个实际上不包含值的属性，但是对其访问会自动导致对getter隐藏函数的调用，其返回的值就是属性访问的结果。

```js
var myObject = {
	// define a getter for `a`
	get a() {
		return 2;
	}
};

myObject.a = 3;

myObject.a; // 2
```

因为我们只为`a`定义了一个getter，如果我们稍后尝试设置`a`的值，set操作不会抛出错误，而是会默默地忽略赋值。即使有一个有效的setter，我们的自定义getter也被硬编码为仅返回`2`，因此set操作将没有实际意义。

为了使这个场景更加合理，还应该使用setter来定义属性，setter会覆盖响应属性默认的`[[Put]]`操作（又称，赋值），就像你期望的那样。你几乎肯定会始终声明getter和setter（只有一个经常导致意外/令人惊讶的行为）：

```js
var myObject = {
	// define a getter for `a`
	get a() {
		return this._a_;
	},

	// define a setter for `a`
	set a(val) {
		this._a_ = val * 2;
	}
};

myObject.a = 2;

myObject.a; // 4
```

**注意：** 在这个例子中，我们实际上将赋值（`[[Put]]`操作）的指定值`2`存储到另一个变量`_a_`中。对于这个例子，`_a_`是按惯例命名的，并且暗示它的行为没什么特别的 -- 它是一个像其他任何一样的普通属性。

### 存在（Existence）

我们之前说过，`myObject.a`这样的属性访问结果可能是`undefined`，可能是由于显式地存储了`undefined`，也可能是`a`属性不存在。那么，如果两种情况下得到的值相同，我们如何区分它们呢？

我们可以询问一个对象是否有某个属性，*而不用*获取该属性的值：

```js
var myObject = {
	a: 2
};

("a" in myObject);				// true
("b" in myObject);				// false

myObject.hasOwnProperty( "a" );	// true
myObject.hasOwnProperty( "b" );	// false
```

`in`运算符将检查属性是否*在*对象中，或者它是否存在于`[[Prototype]]`链对象遍历的任何更高层次（参见第5章）。相比之下，`hasOwnProperty(..)`*只*检查`myObject`本身是否具有该属性，并*不*查询`[[Prototype]]`链。当我们在第5章中详细探讨`[[Prototype]]`时，我们将回过头来看这两个操作之间的重要区别。

通过委托给`Object.prototype`，所有普通对象可以访问`hasOwnProperty(..)`（参见第5章）。但是可以创建一个链接不到`Object.prototype`的对象（通过`Object.create(null)` - 参见第5章）。在这种情况下，像`myObject.hasOwnProperty(..)`这样的方法调用会失败。

在那种情况下，执行这种检查更健壮的方法是，`Object.prototype.hasOwnProperty.call(myObject,"a")`，它借用了基本的`hasOwnProperty(..)`方法并使用*显式`this`绑定*（参见第2章），将其应用于我们的`myObject`。

**注意：** `in`运算符的表面上看是它将检查容器内是否存在*值*，但它实际上检查是否存在属性名称。对于数组而言，这种差异很重要，因为尝试像`4 in [2, 4, 6]`这样的检查的诱惑力很强，但这不会符合预期。

#### 枚举（Enumeration）

之前，在我们介绍`enumerable`属性描述符特性时，简单解释了『枚举』的思想。让我们重新审视一下，并更详细地研究它。

```js
var myObject = { };

Object.defineProperty(
	myObject,
	"a",
	// make `a` enumerable, as normal
	{ enumerable: true, value: 2 }
);

Object.defineProperty(
	myObject,
	"b",
	// make `b` NON-enumerable
	{ enumerable: false, value: 3 }
);

myObject.b; // 3
("b" in myObject); // true
myObject.hasOwnProperty( "b" ); // true

// .......

for (var k in myObject) {
	console.log( k, myObject[k] );
}
// "a" 2
```

你会注意到`myObject.b`实际上**存在**且具有可访问的值，但它不会出现在`for..in`循环中（但令人惊讶的是，由`in`运算符检查显示存在）。这是因为“可枚举”本质上意味着“如果对象的属性被迭代时将包含”。

**注意：** 应用于数组的`for..in`循环可能会产生一些意想不到的结果，因为数组的枚举不仅包括所有数字索引，还包括任何可枚举属性。*只*在对象上使用`for..in`循环是个好主意，对于存储在数组中的值使用传统的`for`循环和数字索引迭代。

可以区分可枚举和不可枚举属性的另一种方法：

```js
var myObject = { };

Object.defineProperty(
	myObject,
	"a",
	// make `a` enumerable, as normal
	{ enumerable: true, value: 2 }
);

Object.defineProperty(
	myObject,
	"b",
	// make `b` non-enumerable
	{ enumerable: false, value: 3 }
);

myObject.propertyIsEnumerable( "a" ); // true
myObject.propertyIsEnumerable( "b" ); // false

Object.keys( myObject ); // ["a"]
Object.getOwnPropertyNames( myObject ); // ["a", "b"]
```

`propertyIsEnumerable(..)`测试给定的属性名是否在对象上*直接*存在并且也是`enumerable:true`。

`Object.keys(..)`返回所有可枚举属性的数组，而`Object.getOwnPropertyNames(..)`返回*所有*属性的数组，不论是否可枚举。

而`in`与`hasOwnProperty(..)`的区别在于它们是否参考`[[Prototype]]`链，`Object.keys(..)`和`Object.getOwnPropertyNames(..)` *仅*检查指定的直接对象。

（目前）没有内置的方法来获取**所有属性的列表**这相当于`in`操作符测试将参考的内容（遍历整个`[[Prototype]]`链上的所有属性，在第5章中解释）。你可以实现通过递归遍历对象的`[[Prototype]]`链来近似这样的实用程序，并且对于每个层次，使用`Object.keys(..)`获取列表 - 只有可枚举的属性。

## 迭代

`for..in`循环遍历对象上的可枚举属性列表（包括其`[[Prototype]]`链）。但是，如果想要对值进行迭代，该怎么办？

对于数字索引的数组，迭代值通常使用标准的`for`循环来完成，如：

```js
var myArray = [1, 2, 3];

for (var i = 0; i < myArray.length; i++) {
	console.log( myArray[i] );
}
// 1 2 3
```

但是，这不会迭代值，而是遍历索引，然后使用索引来引用值，正如`myArray[i]`所示。

ES5还为数组添加了几个迭代工具，包括`forEach(..)`，`every(..)`和`some(..)`。这些工具中的每一个都接受一个函数回调以应用于数组中的每个元素，区别仅在于它们分别如何响应回调中的返回值。

`forEach(..)`将迭代数组中的所有值，并忽略任何回调返回值。 `every(..)`一直持续到结束*或*回调返回一个`false`（或“falsy”）值，而`some(..)`一直持续到结束*或*回调返回一个`true`（或“truthy”）值。

`every(..)`和`some(..)`中的这些特殊返回值有点像普通`for`循环中的`break`语句，因为它们可以在到达结束之前就提前停止迭代。

如果你使用`for..in`循环迭代一个对象，你也只是间接得到这些值，因为它实际上只是迭代对象的可枚举属性，然后让你手动访问属性以获得值。

**注意：** 与以数字有序的方式迭代数组的索引（`for`循环或其他迭代器）相比，对象属性的迭代顺序**不能保证**，并且可能因不同的JS引擎而异。对于需要在环境中保持一致的事物，**不要依赖**观察到的顺序，因为任何观察到的一致性都是不可靠的。

但是如果你想直接迭代值而不是数组索引（或对象属性）呢？有用的是，ES6添加了一个`for..of`循环语法，用于迭代数组（和对象，如果对象定义了自己的自定义迭代器）：

```js
var myArray = [ 1, 2, 3 ];

for (var v of myArray) {
	console.log( v );
}
// 1
// 2
// 3
```

`for..of`循环要求被迭代的*事物*拥有一个迭代器对象（来自规范中称为`@@iterator`的默认内部函数），然后循环从调用迭代器对象的`next()`方法得到返回值，迭代其返回值，每次循环迭代一次。

数组有一个内置的`@@iterator`，所以`for..of`很容易对它们起作用，如上所示。让我们使用内置的`@@ iterator`手动迭代数组，看看它是如何工作的：

```js
var myArray = [ 1, 2, 3 ];
var it = myArray[Symbol.iterator]();

it.next(); // { value:1, done:false }
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { done:true }
```

**注意：** 我们使用ES6`Symbol`：`Symbol.iterator`获取对象的`@@iterator` *内部属性*。我们在本章前面简要提到了`Symbol`语义（参见“计算属性名称”），因此同样的语义也适用于此。你应该总是通过`Symbol`名称引用而不是它可能包含的特殊值来引用这些特殊属性。另外，尽管名称有所暗示，`@@iterator`**不是iterator对象**本身，而是**返回迭代器对象的函数** - 一个微妙但重要的细节！

正如上面的代码片段所示，迭代器的`next()`调用的返回值是`{value: ..，done: ..}`形式的对象，其中`value`是当前的迭代值，并且`done`是一个`boolean`，表示是否还有更多迭代。

请注意，返回值`3`时带有`done:false`，乍一看似乎很奇怪。你必须第四次调用`next()`（前一个代码片段中的`for..of`循环自动执行）才能得到`done:true`知道你真正完成了迭代。这个奇怪行为的原因超出了我们在这里讨论的范围，但它来自ES6生成器函数的语义。

数组在`for..of`循环中可以自动迭代，而常规对象**没有内置的`@@iterator`**。这种故意缺失的原因比我们在这里要讨论的更为复杂，但总的来说，最好不要包含一些可能对未来类型的对象造成麻烦的实现。

你*可以*为想要迭代的任何对象定义你自己默认的`@@iterator`。例如：

```js
var myObject = {
	a: 2,
	b: 3
};

Object.defineProperty( myObject, Symbol.iterator, {
	enumerable: false,
	writable: false,
	configurable: true,
	value: function() {
		var o = this;
		var idx = 0;
		var ks = Object.keys( o );
		return {
			next: function() {
				return {
					value: o[ks[idx++]],
					done: (idx > ks.length)
				};
			}
		};
	}
} );

// iterate `myObject` manually
var it = myObject[Symbol.iterator]();
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { value:undefined, done:true }

// iterate `myObject` with `for..of`
for (var v of myObject) {
	console.log( v );
}
// 2
// 3
```

**注意：** 我们使用`Object.defineProperty(..)`来定义我们自定义的`@@iterator`（主要是因为我们可以使它不可枚举），但使用`Symbol`作为*计算属性名称*（在本章前面已经介绍过），我们可以直接声明它，比如`var myObject = { a:2, b:3, [Symbol.iterator]: function(){ /* .. */ } }`。

每次`for..of`循环在`myObject`的迭代器对象上调用`next()`时，内部指针将前进并返回对象属性列表中的下一个值（请参阅前面关于迭代对象属性/值顺序的注释）。

我们刚刚演示的迭代是一个简单的逐值迭代，但你可以根据需要为自定义数据结构定义任意复杂的迭代。自定义迭代器结合ES6的`for..of`循环是一个强大的新语法工具，用于操作用户自定义的对象。

例如，`Pixel`对象列表（带有`x`和`y`坐标值）可以决定根据与`(0,0)`原点的线性距离对其迭代进行排序，或者过滤掉“太远”的点等等。只要你的迭代器从`next()`调用返回预期的`{value: ..}`返回值，并在迭代完成后返回`{done: true}`，ES6的`for..of`就可以迭代它。

实际上，你甚至可以生成“无限”迭代器，它们永远不会“完成”并始终返回一个新值（例如随机数，递增值，唯一标识符等），尽管你可能不会在无边界的`for..of`循环使用这样的迭代器，因为它永远不会结束并且会挂起你的程序。

```js
var randoms = {
	[Symbol.iterator]: function() {
		return {
			next: function() {
				return { value: Math.random() };
			}
		};
	}
};

var randoms_pool = [];
for (var n of randoms) {
	randoms_pool.push( n );

	// don't proceed unbounded!
	if (randoms_pool.length === 100) break;
}
```

这个迭代器将“永远”生成随机数，所以我们小心地只提取100个值，这样我们的程序就不会被挂起。

## 回顾 (TL;DR)

JS中的对象既有字面量形式（如`var a = {..}`）又有构造形式（如`var a = new Array(..)`）。文字形式几乎总是首选，但在某些情况下，构造的形式提供了更多的创建参数选项。

许多人错误地声称“JavaScript中的所有东西都是对象”，但这是不正确的。对象是6（或7，取决于你的观点）原始类型之一。对象具有子类型，包括`function`，也可以是有专用行为的，如`[object Array]`作为表示数组对象子类型的内部标签。

对象是键/值对的集合。值可以使用`.propName`或`[“propName”]`语法通过属性进行访问。每当访问一个属性时，引擎实际上会调用内部默认的`[[Get]]`操作（和`[[Put]]`来设置值），它不仅直接在对象上查找属性，而且在如果没有找到时，遍历`[[Prototype]]`链（见第5章）。

属性具有某些特性，可以通过属性描述符来控制，例如`writable`和`configurable`。此外，对象可以使用`Object.preventExtensions(..)`，`Object.seal(..)`和`Object.freeze(..)`将其（以及它们的属性）可变性控制到不同的级别。

属性不必包含值 - 它们也可以是“访问器属性”，具有getter/setter。它们也可以是是否*可枚举*，控制它们在`for..in`循环迭代中是否出现。

你还可以使用ES6`for..of`语法迭代数据结构（数组，对象等）中的**值**，该语法查找内置或自定义`@@iterator`对象。每次使用`next()`方法，一次一个地迭代数据值。
