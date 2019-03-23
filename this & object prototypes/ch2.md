# 你不知道的JS：*this* & 对象原型
# 第二章：`this`这样就解释通了

在第1章中，我们抛弃了关于`this`的各种误解，并且学会了`this`是为每个函数调用做的绑定，完全基于它的**调用点**（如何调用函数）。

## 调用点

要理解`this`，我们必须理解调用点：代码中函数调用的位置（**不是定义的位置**）。我们必须审视调用点来回答这个问题：*这个*`this`指向啥？

找到调用点大概就是：『到一个函数被调用的地方』，但是这并不总是十分容易，因为某些编码模式可能会掩盖*真正的*调用点。

重要的是考虑**调用栈**(已被调用的函数堆栈，以便让我们获得执行的当前时刻）。我们关心的调用点是*在*当前执行函数之前。

让我们模拟调用栈和调用点：

```js
function baz() {
    // call-stack is: `baz`
    // so, our call-site is in the global scope

    console.log( "baz" );
    bar(); // <-- call-site for `bar`
}

function bar() {
    // call-stack is: `baz` -> `bar`
    // so, our call-site is in `baz`

    console.log( "bar" );
    foo(); // <-- call-site for `foo`
}

function foo() {
    // call-stack is: `baz` -> `bar` -> `foo`
    // so, our call-site is in `bar`

    console.log( "foo" );
}

baz(); // <-- call-site for `baz`
```

在分析代码来找到实际调用点（通过调用栈）时要当心，因为这是唯一影响`this`绑定的事情。

**注意：** 你可以通过按顺序查看函数调用链来在脑海中可视化调用堆栈，就像我们对上面代码段注释中所做的那样。但这是艰苦的，容易出错。查看调用堆栈的另一种方法是在浏览器中使用调试器工具。大多数现代桌面浏览器都有内置的开发人员工具，其中包括一个JS调试器。在上面的代码片段中，你可以使用工具在`foo()`函数的第一行的设置断点，或者只是在第一行插入`debugger;`语句。当你运行该页面时，调试器将在此位置暂停，并将显示已调用的函数列表，这些函数将成为您的调用堆栈。 因此，如果你正在尝试确定`this`绑定，请使用开发人员工具获取调用堆栈，然后从栈顶找到第二项，这将显示真实的调用点。

## 只是一些规则

我们将注意力转向调用点*如何*在函数执行时决定`this`的指向。

你必须检查调用点并确定适用4个规则中的哪一个。我们将首先独立解释这四个规则，然后我们将说明在*可以*适用多个规则时，它们的优先顺序是怎样。

### 默认绑定

我们将研究的第一个规则来自函数调用的最常见情况：独立函数调用。当没有其他规则适用时，可以将*这个*`this`规则将视为默认的catch-all规则。

考虑这段代码：

```js
function foo() {
	console.log( this.a );
}

var a = 2;

foo(); // 2
```

第一件需要知道的是，如果你没有意识到的话，在全局作用域中定义的变量，比如`var a = 2`，会是全局对象中相同名称属性的同义词。它们彼此不是备份，而就*是*对方。就像一个硬币的正反面。

其次，我们看到当`foo()`被调用时，`this.a`被解释为全局变量`a`。为什么呢？因为这种情况下，`this`的*默认绑定*在函数调用时被使用，因此`this`指向了全局变量。

我们怎么知道*默认绑定*规则在这里使用？我们通过检查调用点来看`foo()`如何被调用，`foo()`通过一个普通的装饰的引用来调用。我们将演示的其他规则都不适用于此，因此*默认绑定*适用。

在`严格模式`下，全局变量不会在*默认绑定*中使用，因此`this`被设置为`undefined`。

```js
function foo() {
	"use strict";

	console.log( this.a );
}

var a = 2;

foo(); // TypeError: `this` is `undefined`
```

一个细微但是重要的细节是：即使整个`this`绑定规则完全基于调用点，全局对象适用于*默认绑定***仅仅**在`foo()`的**内容**处于*非*`严格模式`运行时。而处于`严格模式`状态的`foo()`调用点是无关的。

```js
function foo() {
	console.log( this.a );
}

var a = 2;

(function(){
	"use strict";

	foo(); // 2
})();
```

**注意：** 有意地在你的代码中混合`严格模式`与非`严格模式`一起使用通常是不可取的。你的整个程序要么是**严格**要么是**非严格**。然而，有时你的代码中包含的第三方库与你的代码是否**严格**有出入，因此要特别小心这些适用性细节。

### 隐式绑定

另外一条规则是：调用点具有的上下文对象，也称为拥有或包含对象，但*这些*替代术语可能会略有误导。

考虑：

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

obj.foo(); // 2
```

首先，注意`foo()`被定义了，并且作为属性被添加到`obj`。无论`foo()`是否在初始时就定义在`obj`*上*，还是作为引用后续添加（上述代码展示），**函数**都确实被`obj`对象『拥有』或『包含』。

然而，调用点*使用*`obj`上下文来**引用**到函数，因此你*可以*说`obj`对象在函数被调用时『拥有』或『包含』这个**函数引用**。

无论你选择何种方式调用这种模式，在调用`foo()`时，都会有一个对象引用`obj`。当有一个函数引用的上下文对象时，*隐式绑定*规则表明*它*就是那个应该用于函数调用`this`绑定的对象。

由于`obj`就是`foo()`调用的`this`，`this.a`相当于`obj.a`。

只有对象属性引用链的顶级/最后一级对调用点有作用。例如：

```js
function foo() {
	console.log( this.a );
}

var obj2 = {
	a: 42,
	foo: foo
};

var obj1 = {
	a: 2,
	obj2: obj2
};

obj1.obj2.foo(); // 42
```

#### 隐式丢失

一个常见的意外情况是，当*隐式绑定*函数失去该绑定时的`this`绑定，这通常意味着它回退到*默认绑定*，指向全局对象或`undefined`，具体取决于是否在`严格模式`。

考虑：

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

var bar = obj.foo; // function reference/alias!

var a = "oops, global"; // `a` also property on global object

bar(); // "oops, global"
```

即使`bar`就是`obj.foo`的引用，实际上，他确实就是`foo`自身的另一个引用。然而，调用点产生了影响，调用点是`bar()`，这是一个普通的没有修饰的调用，因此适用于*默认绑定*。

一种更细微，更普遍，更不符合预期的方式是在我们使用回调函数时：

```js
function foo() {
	console.log( this.a );
}

function doFoo(fn) {
	// `fn` is just another reference to `foo`

	fn(); // <-- call-site!
}

var obj = {
	a: 2,
	foo: foo
};

var a = "oops, global"; // `a` also property on global object

doFoo( obj.foo ); // "oops, global"
```

参数传递只是一个隐式赋值，因为我们传递一个函数，它是一个隐式引用赋值，所以最终结果与前一个片段相同。

如果你将回调函数传递给内置函数，而不是你自己的函数，那会怎样？没有区别，同样的结果。

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

var a = "oops, global"; // `a` also property on global object

setTimeout( obj.foo, 100 ); // "oops, global"
```

作为JavaScript环境中的内置提供的`setTimeout()`，考虑一下它的粗略的理论上的伪实现：

```js
function setTimeout(fn,delay) {
	// wait (somehow) for `delay` milliseconds
	fn(); // <-- call-site!
}
```

正如我们刚刚看到的那样，我们的回调函数*丢失*它们的`this`绑定是很常见的。但另一种`this`让我们感到惊讶的方式是当我们通过回调函数故意改变调用的`this`时。流行的JavaScript库中的事件处理程序非常喜欢强制你的回调有一个`this`，它指向例如触发事件的DOM元素。虽然这可能有时是有用的，但有时它可能彻头彻尾地气人。不幸的是，这些工具很少提供选择给你。

无论哪种方式，`this`都会意外更改，你实际上无法控制回调函数引用的执行方式，因此你（还）无法控制调用点以提供预期的绑定。我们很快就会看到一种通过*修复*`this`“解决”这个问题的方法。

### 显式绑定

正如我们刚刚看到的那样，使用*隐式绑定*，我们不得不改变有问题的对象，以包含对函数本身的引用，并使用此属性函数引用间接（隐式）将`this`绑定到对象。

但是，如果你想强制一个函数调用使用一个特定的对象进行`this`绑定，而不在对象上放置一个属性函数引用呢？

语言中的“所有”函数都有一些可用的实用程序（通过它们的`[[Prototype]]` - 后面会有更多内容），这对于这个任务很有用。具体来说，函数有`call(..)`和`apply(..)`方法。从技术上讲，JavaScript主机环境有时提供足够特殊（说的好听！）的函数，它们没有这些方法。但那些很少。提供的绝大多数函数，当然还有你创建的所有函数，都可以访问`call(..)`和`apply(..)`。

这些实用程序如何工作？它们的第一个参数，使用一个用于`this`的对象，然后用指定的`this`调用该函数。既然你直接说明了你想要的`this`，我们称它为*显式绑定*。

考虑：

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

使用`foo.call(..)`*显式绑定*来调用`foo`，可以让我们将它的`this`强制设为`obj`。

如果你提供了一个简单的基本类型（比如`string`, `boolean`, 或者`number`）作为`this`绑定，基本类型会被它们的对象形式包裹（分别为`new String(..)`, `new Boolean(..)`, 或者`new Number(..)`）。这通常称为『装盒（boxing）』。

**注意：** 关于`this`绑定，`call(..)`和`apply(..)`是相同的。它们*确实*的不同行为与它们的附加参数相关，但这不是我们目前关心的事情。

不幸的是，单独的*显式绑定*仍然不能解决任何前面提到问题，一个函数“丢失”它的预期`this`绑定，或者只是被框架覆盖了。

#### 强绑定

但围绕*显式绑定*的变化模式实际上可以解决问题。考虑：

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2
};

var bar = function() {
	foo.call( obj );
};

bar(); // 2
setTimeout( bar, 100 ); // 2

// `bar` hard binds `foo`'s `this` to `obj`
// so that it cannot be overriden
bar.call( window ); // 2
```

让我们来看看这种变化是如何工作的。 我们创建一个函数`bar()`，它在内部手动调用`foo.call(obj)`，从而强制使用`obj`绑定`this`调用`foo`。无论你以后如何调用函数`bar`，它总是用`obj`手动调用`foo`。这种绑定既明确又强大，所以我们称之为*强绑定*。

使用*强绑定*包装函数的最典型方法是，透传所有参数，并返回收到的任何返回值：

```js
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

var obj = {
	a: 2
};

var bar = function() {
	return foo.apply( obj, arguments );
};

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

此模式的另一种方法是创建一个可重用的工具：

```js
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

// simple `bind` helper
function bind(fn, obj) {
	return function() {
		return fn.apply( obj, arguments );
	};
}

var obj = {
	a: 2
};

var bar = bind( foo, obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

由于*硬绑定*是如此常见的模式，在ES5中作为内置工具被提供：`Function.prototype.bind`，它可以这样使用『』

```js
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

var obj = {
	a: 2
};

var bar = foo.bind( obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

`bind(..)`返回一个新的硬编码的函数，用你指定的`this`上下文设置调用原始函数。

**注意：** 在ES6中，`bind(..)`生成的硬绑定函数有一个`.name`属性，它源自原始*目标函数*。 例如：`bar = foo.bind(..)`应该有一个`bar.name`值为`"bound foo"`，这是应该在堆栈跟踪中显示的函数调用名。

#### API调用"上下文"

在JavaScript语言和株距环境中，很多库函数，还有很多新的内置函数提供一个可选的参数，通常成为『上下文』，这是一个解决方案，你不必使用`bind(..)`来确保你的回调函数使用特定的`this`。

例如：

```js
function foo(el) {
	console.log( el, this.id );
}

var obj = {
	id: "awesome"
};

// use `obj` as `this` for `foo(..)` calls
[1, 2, 3].forEach( foo, obj ); // 1 awesome  2 awesome  3 awesome
```

在内部实现中，这些函数几乎肯定使用了*显式绑定*通过`call(..)`或`apply(..)`，来省去了你的麻烦。

### `new`绑定

第四条同时也是最后一个`this`绑定规则，要求我们重新思考一个JavaScript常见的关于函数和对象的误解。

在传统的面向对象的语言中，『构造函数』是类上的特殊方法，当类被`new`操作符初始化时，类的构造函数被调用。通常像这样：

```js
something = new MyClass(..);
```

JavaScript有一个`new`运算符，使用它的代码模式看起来与我们在那些面向类的语言中看到的基本相同; 大多数开发人员都认为JavaScript的机制正在做类似的事情。但是，在JS中使用`new`用法隐含的与面向类的功能确实*没有关系*。

首先，让我们重新定义JavaScript中的“构造函数”。在JS中，构造函数*只是函数*，恰好使用`new`运算符在前面时调用。它们不附着于类，也不是实例化类。它们甚至不是特殊类型的函数。它们只是常规函数，实际上是在调用中使用`new`来劫持。

例如，`Number(..)`函数作为一个构造函数，引用自ES5.1规范：

> 15.7.2 Number构造函数
>
> 当Number作为new表达式的一部分被调用时，它是一个构造函数：它初始化新创建的对象。

因此，几乎任何函数，包括内置对象函数，如`Number(..)`（参见第3章）都可以在它前面用`new`调用，这使得该函数称为一个*构造函数调用*。这是一个重要但微妙的区别：实际上没有“构造函数”这样的东西，而是函数*的*函数构造调用。

当一个函数在前面使用`new`时，除了称为构造函数调用，以下这几件事儿也会自动进行：

1. 一个全新的对象会被凭空创建（也称构造）
2. *这个新构建的对象会被接入原形链（[[Prototype]]-linked）*
3. 这个新构建的对象会被设置为此次函数调用的`this`绑定
4. 除非函数返回它自己的**对象**，这个`new`调用的函数会*自动*返回新构造的对象

第1，3，4步与我们当前讨论的有关。我们当前将跳过第2步，到第五章时再回来看。

考虑这段代码：

```js
function foo(a) {
	this.a = a;
}

var bar = new foo( 2 );
console.log( bar.a ); // 2
```

在`foo(..)`使用`new`来调用，我们构造了一个新的对象，并在`foo(..)`调用中将新对象设置为`this`。**因此`new`是最后一种在函数调用时绑定`this`的方式**。我们称之为*new绑定*。

## 一切都井然有序

所以，现在我们揭开了函数调用绑定`this`的四条规则。你所要做的*全部*工作就是找到调用点，检查适用于哪条规则。但是，如果调用点如果适合多条规则呢？那这些规则一定有优先级顺序，我们接下来就演示这些规则的应用顺序。

需要清楚的时*默认绑定*的优先级顺序是四条规则中最低的。因此我们将其搁置一边。

*隐式绑定*和*显式绑定*哪一个更优先？我们来测试一下：

```js
function foo() {
	console.log( this.a );
}

var obj1 = {
	a: 2,
	foo: foo
};

var obj2 = {
	a: 3,
	foo: foo
};

obj1.foo(); // 2
obj2.foo(); // 3

obj1.foo.call( obj2 ); // 3
obj2.foo.call( obj1 ); // 2
```

因此*显式绑定*优先于*隐式绑定*，这意味着你应该**首先**检查*显式绑定*。

现在，我们需要看*new绑定*的优先级。

```js
function foo(something) {
	this.a = something;
}

var obj1 = {
	foo: foo
};

var obj2 = {};

obj1.foo( 2 );
console.log( obj1.a ); // 2

obj1.foo.call( obj2, 3 );
console.log( obj2.a ); // 3

var bar = new obj1.foo( 4 );
console.log( obj1.a ); // 2
console.log( bar.a ); // 4
```

好的，*new绑定*优先于*隐式绑定*。但是你认为*new绑定*是否优先于*显式绑定*呢？

**注意：** `new`与`call`/`apply`不能同时使用，因此将*new绑定*与*显式绑定*直接测试`new foo.call(obj1)`是行不通的。但是我们依然可以使用*硬绑定*来测试这两条规则的优先级。

我们在代码清单中探讨之前，回想一下*hard binding*如何工作，即`Function.prototype.bind(..)`创建一个新的包装函数，它被硬编码以忽略它自己的`this `绑定（无论它是什么），而是使用我们手动提供的。

由于这个原因，一个显然的假设就是*硬绑定*（*显式绑定*的一种形式）有比*new绑定*更高的优先级，因此不能被`new`覆盖。

我们检查下：

```js
function foo(something) {
	this.a = something;
}

var obj1 = {};

var bar = foo.bind( obj1 );
bar( 2 );
console.log( obj1.a ); // 2

var baz = new bar( 3 );
console.log( obj1.a ); // 2
console.log( baz.a ); // 3
```

哇！`bar`硬绑定到了`obj1`，但是`new bar(3)`并*没有*如我们预期将`obj1.a`改变为`3`。相反*硬绑定*（到`obj1`）的`bar(..)`***可以***被`new`覆盖。由于`new`的应用，我们得到了新创建的对象，命名为`baz`，同时`baz.a`的值为`3`。

如果你回头看我们的『假』绑定工具，就会比较奇怪：

```js
function bind(fn, obj) {
	return function() {
		fn.apply( obj, arguments );
	};
}
```

如果你深究这个工具如何运行，就会发现它并没有处理`new`操作符调用时覆盖硬绑定到`obj`的操作。

但是ES5内置的`Function.prototype.bind(..)`相比就非常复杂。这是来自MDN页面的`bind(..)`的polyfill（稍微格式化后）：

```js
if (!Function.prototype.bind) {
	Function.prototype.bind = function(oThis) {
		if (typeof this !== "function") {
			// closest thing possible to the ECMAScript 5
			// internal IsCallable function
			throw new TypeError( "Function.prototype.bind - what " +
				"is trying to be bound is not callable"
			);
		}

		var aArgs = Array.prototype.slice.call( arguments, 1 ),
			fToBind = this,
			fNOP = function(){},
			fBound = function(){
				return fToBind.apply(
					(
						this instanceof fNOP &&
						oThis ? this : oThis
					),
					aArgs.concat( Array.prototype.slice.call( arguments ) )
				);
			}
		;

		fNOP.prototype = this.prototype;
		fBound.prototype = new fNOP();

		return fBound;
	};
}
```

**注意：** 上面显示的`bind(..)`polyfill与ES5中内置的`bind(..)`的不同之处在于与`new`一起使用的硬绑定函数（为什么那是有用的见下文）。因为polyfill不能像内置实用程序那样创建没有`.prototype`的函数，所以有一些细微的间接方式来近似相同的行为。如果你计划使用具有硬绑定功能的`new`并依赖此polyfill，请小心一点。

这些部分可以让`new`来覆盖：

```js
this instanceof fNOP &&
oThis ? this : oThis

// ... and:

fNOP.prototype = this.prototype;
fBound.prototype = new fNOP();
```

我们实际上不会深入解释这个技巧是如何工作的（它很复杂并超出了我们的范围），但该实用程序实际上取决于是否使用`new`调用了硬绑定函数（导致新构造的对象成为它的`this`），如果是这样的话，它使用*那个*新创建的`this`而不是之前指定的*硬绑定*指定的`this`。

为什么`new`可以覆盖*硬绑定*是有用的？

主要的原因就是这种行为可以创建这样一个函数（可以用来通过`new`构件对象），它可以忽略`this`*硬绑定*但可以预制一部分或者全部函数参数。`bind(..)`的一个基本能力就是任何在第一个`this`绑定参数之后传入的参数会当做边准的参数传入所依赖的那个函数（技术上称为『部分应用(partial application)』）,它是『柯里化(currying)』的一个子集。

例如：

```js
function foo(p1,p2) {
	this.val = p1 + p2;
}

// using `null` here because we don't care about
// the `this` hard-binding in this scenario, and
// it will be overridden by the `new` call anyway!
var bar = foo.bind( null, "p1" );

var baz = new bar( "p2" );

baz.val; // p1p2
```

### 判断`this`

现在，我们可以总结判断从一个函数调用的调用点来判断`this`的规则了，即它们的优先级。以这样的顺序问问题，直到第一个满足情况的规则。

1. 这个函数是否被`new`调用（**new绑定**）？如果是这样，`this `就是新创建的对象。

    `var bar = new foo()`

2. 这个函数是否使用`call`或者`apply`调用（*显式绑定*），或者隐藏在一个`bind`*硬绑定*之后？如果是这样，`this`就是显式指定的那个对象。

    `var bar = foo.call( obj2 )`

3. 这函数是否使用一个上下文来调用（**隐式绑定**），即通过一个拥有或者包含对象？如果是这样，`this`就是*那个*上下文对象。

    `var bar = obj1.foo()`

4. 否则就是默认的`this`（**默认绑定**）。如果在`严格模式`，就是`undefined`，否则就是`global`对象。

    `var bar = foo()`

就是这样。这就是要理解正常函数调用的`this`绑定的*全部内容*。嗯......差不多。

## 绑定异常

像往常一样，“规则”有一些*例外*。

在某些情况下，`this`绑定行为可能会令人惊讶，你打算使用不同的对象绑定，但最终会遇到来自*默认绑定*规则的绑定行为（请参阅上一页）。

### 忽略`this`

如果你将`null`或者`undefined`作为`this`绑定的参数传入`call`, `apply`, 或者`bind`时，这些值实际上会被忽略，而*默认绑定*规则会应用于此次调用。

```js
function foo() {
	console.log( this.a );
}

var a = 2;

foo.call( null ); // 2
```

你为什么会有意地将类似`null`的东西传给`this`绑定？

一个非常常用的用法是在`apply(..)`中扩展值的数组作为一个函数调用的参数。类似的，`bind(..)`可以柯里化参数（预置参数），这是非常有用的。

```js
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}

// spreading out array as parameters
foo.apply( null, [2, 3] ); // a:2, b:3

// currying with `bind(..)`
var bar = foo.bind( null, 2 );
bar( 3 ); // a:2, b:3
```

这些工具都需要一个`this`绑定作为第一个参数。如果函数实际上不关心`this`，那么你需要一个站位值，这时候正如上述代码所示的`null`可能是一个合理的选择。

**注意：** 我们在本书中没有介绍它，但是ES6有`...`扩展运算符，它可以让你在语法上“展开”一个数组作为参数，而不需要`apply(..)`，例如`foo(...[1,2])`，相当于`foo(1,2)` - 如果没有必要的话在语法上避免了`this`绑定。不幸的是，没有ES6语法可以替代柯里化，所以`bind(..)`调用的`this`参数仍然需要注意。

但是，当你不关心`this`绑定时，总是使用`null`会有一点隐藏的“危险”。如果你曾在函数调用（例如，你不能控制的第三方库函数）中使用它，并且该函数*确实*使用了`this`引用，*默认绑定*规则意味着它可能会无意中引用（或者更糟，改变！）`global`对象（浏览器中的`window`）。

显然，这样的陷阱会导致各种*非常难以*诊断/追踪的bug。

#### 更安全的`this`

也许可能“更安全”的做法是为`this`传递一个专门设置的对象，这保证在程序中不会产生引起副作用问题的对象。从网络（和军队）借用术语，我们可以创建一个“DMZ”（非军事区）对象 - 没有什么比完全空的，无委托（见第5章和第6章）对象更特殊。

如果我们总是传递一个忽略`this`绑定的DMZ对象，我们就没有什么好担心了，我们确定`this`的隐藏/意外用法将仅限于在空对象上，这使我们的程序将全局对象与副作用隔离开来。

由于这个对象是完全空的，我个人喜欢给它命名为`ø`（空集的小写数学符号）。在许多键盘上（如Mac上的US-layout），这个符号可以用`⌥`+`o`（选项+`o`）轻松输入。某些系统还允许你为特定符号设置热键。如果您不喜欢“ø”符号，或者您的键盘不能轻易打出来，你当然可以另起一个名字。

无论你怎么称呼它，将它设置为**完全空**的最简单方法是`Object.create(null)`（见第5章）。`Object.create(null)`类似于`{ }`，但没有委托给`Object.prototype`，所以它比`{ }`更“空”。

```js
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}

// our DMZ empty object
var ø = Object.create( null );

// spreading out array as parameters
foo.apply( ø, [2, 3] ); // a:2, b:3

// currying with `bind(..)`
var bar = foo.bind( ø, 2 );
bar( 3 ); // a:2, b:3
```

不仅在功能上“更安全”，`ø`还有一种风格上的好处，因为它在语义上传达了“我希望`this`为空”，比`null`更清楚。但是，再次强调，你可以按照喜好命名DMZ对象。

### 间接

另外一件你需要注意的事你可能（有意无意地！）给函数创造一个『间接引用』，在这些场景中，当函数引用被调用，*默认绑定*规则会被应用。

一个*间接引用*最常见的情况就是一次赋值：

```js
function foo() {
	console.log( this.a );
}

var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };

o.foo(); // 3
(p.foo = o.foo)(); // 2
```

赋值表达式`p.foo = o.foo`的*结果值*是对最深层函数对象的引用。因此，有效的调用点只是`foo()`，而不是你想象中的`p.foo()`或`o.foo()`。根据上述规则，适用*默认绑定*规则。

提醒：无论你如何进行函数调用，只要符合*默认绑定*规则，调用函数的**内容**的是否处于`strict mode`状态决定`this`引用 - 而不是函数调用点 - 确定*默认绑定*值：如果是非`严格模式`，则为`global`对象;如果是`strict mode`，则为`undefined`。

### 软化绑定

我们之前见到了*硬绑定*是一种可以防止函数调用在不经意间落入*默认绑定*规则的手段，它可以强制指定一个`this`（除非你用`new`覆盖它！）。问题是*硬绑定*极大地降低了函数的灵活性，阻止了手动的`this`覆盖，无论使用*隐式绑定*还是之后的*显式绑定*都不行。

如果有一种方法可以为*默认绑定*提供不同的默认值（不是`global`或`undefined`），同时仍然可以通过*隐式绑定*或*显式绑定*手动将此函数绑定，那么这将是极好的。

我们可以构造一个所谓的*软绑定*实用程序来模拟我们想要的行为。

```js
if (!Function.prototype.softBind) {
	Function.prototype.softBind = function(obj) {
		var fn = this,
			curried = [].slice.call( arguments, 1 ),
			bound = function bound() {
				return fn.apply(
					(!this ||
						(typeof window !== "undefined" &&
							this === window) ||
						(typeof global !== "undefined" &&
							this === global)
					) ? obj : this,
					curried.concat.apply( curried, arguments )
				);
			};
		bound.prototype = Object.create( fn.prototype );
		return bound;
	};
}
```

除了我们的*软绑定*行为，`softBind(..)`工作方式类似于ES5内置的`bind(..)`。它将一个函数包裹，然后在调用时检查`this`是否是`global`或者`undefined`，如果是就是用一个预置的*默认值*(`obj`)。否则，`this`保持不变。它还提供可选的柯里化（参见前面的`bind(..)`讨论）。

我们来演示下它的用法：

```js
function foo() {
   console.log("name: " + this.name);
}

var obj = { name: "obj" },
    obj2 = { name: "obj2" },
    obj3 = { name: "obj3" };

var fooOBJ = foo.softBind( obj );

fooOBJ(); // name: obj

obj2.foo = foo.softBind(obj);
obj2.foo(); // name: obj2   <---- look!!!

fooOBJ.call( obj3 ); // name: obj3   <---- look!

setTimeout( obj2.foo, 10 ); // name: obj   <---- falls back to soft-binding
```

如上所示，软绑定版本的`foo()`函数可以手动将`this`绑定到`obj2`或者`obj3`，但是它会在*默认绑定*应用时回退到`obj`。

## 词法的`this`

我们刚刚了解了符合四条规则的常规函数。但是ES6介绍了一种不适用我们的规则的特殊函数：箭头函数。

箭头函数并不以`function`关键字标记，而是使用『胖箭头』操作符`=>`。箭头函数的`this`绑定在包裹（函数或者全局）作用域上，而不是由四条标准`this`规则决定。

我们来说明箭头函数词法作用域：

```js
function foo() {
	// return an arrow function
	return (a) => {
		// `this` here is lexically adopted from `foo()`
		console.log( this.a );
	};
}

var obj1 = {
	a: 2
};

var obj2 = {
	a: 3
};

var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, not 3!
```

在`foo()`中创建的箭头函数在调用时从词法上捕获了`foo()`的`this`。由于`foo()`的`this`绑定到`obj1`，`bar`（返回的箭头函数引用）也将`this`绑定到`obj1`。箭头函数的词法绑定不能被覆盖（即使使用`new`！）。

最常见的用例可能是使用回调，例如事件处理程序或计时器：

```js
function foo() {
	setTimeout(() => {
		// `this` here is lexically adopted from `foo()`
		console.log( this.a );
	},100);
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

虽然箭头函数提供了在函数上使用`bind(..)`来确保`this`的替代方法，这看起来很有吸引力，但重要的是要注意它们基本上禁用了传统的`this`机制，以支持更广泛理解的词法作用域。在ES6之前，我们已经有了相当普遍的模式，基本上与ES6箭头功能相同：

```js
function foo() {
	var self = this; // lexical capture of `this`
	setTimeout( function(){
		console.log( self.a );
	}, 100 );
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

虽然`self = this`和箭头函数似乎都是不想要使用`bind(..)`的好的“解决方案”，但它们本质上是逃避`this`而不是理解并拥抱它。

如果你发现自己编写了`this`风格的代码，但是大多数或者所有时间都是用词法的`self = this`或者箭头函数“技巧”来取代`this`机制，也许你应该：

1. 只使用词法作用域，忘记假装`this`式的代码。

2. 完全接受`this`风格的机制，包括在必要时使用`bind(..)`，并尽量避免使用`self = this`和剪头函数的“词法this”技巧。

一个程序可以有效地使用这两种代码（词法和`this`），但是在同一个函数中，实际上对于相同类型的查找，混合这两种机制通常会导致代码难以维护，并且也不太聪明。

## 回顾 (TL;DR)

确定执行函数的`this`绑定需要找到该函数的直接调用点。一旦要检查，可以将四个规则应用于调用点，遵从*这种*的优先顺序：

1. 是否以`new`调用？使用新构建的对象。
   
2. 是否以`call`或者`apply` (或者`bind`)调用？使用指定的对象。

3. 是否以一个上下文对象调用？使用上下文对象。

4. 默认：`严格模式`使用`undefined`，否则为全局对象。

注意意外/无意的调用*默认绑定*规则。如果你想“安全地”忽略`this`绑定，像`ø = Object.create(null)`这样的“DMZ”对象是一个很好的占位符，可以保护`全局`对象免受意外的副作用。

ES6箭头函数不是使用四个标准绑定规则，而是使用词法作用域来实现`this`绑定，这意味着它们从其封闭的函数调用中确定`this`绑定（无论它是什么）。它们本质上是ES6前编码中“self = this”的语法替代。
