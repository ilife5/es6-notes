# ECMAScript 6

## Module Systems for current Javascript

+ CommonJs Modules，在Node中使用的规范

	特点是：语法简洁，为了同步引用而设计，目前主要用在server端

+ Asynchronous Module Definition(AMD)：最流行的实现为[RequireJS](http://requirejs.org/)。

	特点为：略复杂的语法，为了异步引用设计，主要用于浏览器端
	
更多关于CommonJs和AM规范的内容可以参考[Writing Modular JavaScript With AMD, CommonJS & ES Harmony](http://addyosmani.com/writing-modular-js/)

## ECMAScript 6 modules

由于现存的javascript模块存在两种方案，浏览器端使用的AMD以及服务器端使用的CommonJs，ES6模块要做的事情就是兼容两种使用习惯：

+ 同CommonJs一致，语法简洁。提供单入口导出（exports）以及循环依赖检测。

+ 同AMD一致，支持异步的模块加载以及提供模块加载的配置项

ES6 Modules要比起CommonJs以及AMD的优势有：

+ 语法更加简洁

+ 代码结构可以被静态分析（静态检测，优化等等）

+ 循环依赖的检测要优于CommonJS的

ES6模块标准包含如下两部分：

+ 声明式语法（引入，导出）

+ 编程式的加载API：配置模块如何加载以及加载条件

## ES6模块语法概览

导出形式有两种：命名导出（以对象形式导出多个项）以及默认导出（一个模块导出一个项）

### 命名导出（Named exports）

通过在声明前添加<code>export</code>前缀，一个模块可以导出多个项。这些导出项通过它们的命名作区分，被称作命名导出。

```
//----- lib.js ------

export const sqrt = Math.sqrt;

export function square(x) {
	return x * x;
}
 
export function diag(x, y) {
	return sqrt(square(x) + square(y));
}


//------ main.js ------

import { square, diag } from 'lib';
console.log(square(11));	//121
console.log(diag(4, 3));	//5

```

还有很多的方法来指定命名导出，有一种方法是最方便的：在编写代码的时候不要考虑哪个项要被导出，在完成之后把想要的导出项通过<code>export</code>标识。

如果在引入模块的时候要引入所有的命名导出，通过属性符号：

```
//------ main.js ------

import * as lib from 'lib';
console.log(lib.square(11));	//121
console.log(lib.diag(4, 3));	//5
```

在CommonJS中同样的代码：

```
//------ lib.js ------

var sqrt = Math.sqrt;
function square(x) {
	return x * x;
}

function diag(x, y) {
	return sqrt( square(x) + square(y) );
}

module.exports = {
	sqrt: sqrt,
	square: square,
	diag: diag
};

//----- main.js ------

var square = require('lib').square;
var diag = require('lib').diag;
conosle.log(square(11));	//121
console.log(diag(4, 3));	//5
```

### 默认导出

在Node.js社区中，只需要导出单项的模块非常的流行。同样的情况也存在于前端中，比如构造函数或者是类的声明，一个模块对应一个模型。一个Es6的模块可以通过default标识导出。默认导出容易被引用。

下面是一个单功能的Es6模块：

```
//------ myFunc.js ------

export default function() {};

//------ main1.js ------

import myFunc from 'myFunc';
myFunc();
```

一个类模块的导出

```
//------ MyClass.js ------

export default class {};

//------ main2.js ------
import MyClass from 'MyClass';
let inst = new MyClass();
```

注意：一般来说，使用了<code>default export</code>声明的表达式通畅没有名字。在引用的时候可以通过模块的名字来标识引用。

### 一个模块中同时存在export和default export

以下的形式在Javascript中惊人的相似。一个库包含一个函数，所有额外的服务通过这个函数的属性添加。比如jQuery和Underscore.js。下面是Underscore在CommonJs模块下的一个缩影：

```
//------ underscore.js ------

var _ = function(obj) {
	...
};

var each = _.each = _.forEach = function(obj, iterator, context) {
	...
};

module.exports = _;

//------ mian.js ------

var _ = require('underscore');
var each = _.each;
...

```

在ES6的模块中，_是默认的导出项，each和forEach是命名的导出项。在ES6的模块中，可以同时拥有<code>export</code>和<code>default export</code>。上述的模块在ES6的模块规范下，可写作：

```
//------ underscore.js ------

export default function(obj) {
	...
}

export function each(obj, iterator, context) {
	...
}

export {
	each as forEach
}

// ------ main.js ------
import _, { each } from 'underscore';
```

需要注意的是，CommonJS和ES6版本的模块写法大致是相似的。比起前者的嵌套关系，后者有着更平滑的结构。你可以根据自己的喜好选择其中一种，但是平滑的结构更容易进行静态分析。当需要命名空间时，前者在使用上更接近使用者的期望，而在ES6中也是可以完成的。

**default export**也可以被用来做命名空间。

实际上**default export**是命名为default的命名导出项，下面的两段语句是相等的：

```
import {default as foo} from 'lib';
import foo from 'lib';
```

同样的，下面两个模块的默认导出项是一样的：

```
//------ module1.js ------

export default 123;

//------ module2.js ------

const D = 123;
export { D as default };
```

回到根本的问题上来，我们为什么需要命名的导出项？

有了默认的导出项，为什么我们还要使用命名的导出项呢？答案是我们不可能通过对象生成静态的结构，进而丢失所有相关的优点。

## 设计目标

了解ES6的设计目标如何影响到设计方案对搞清楚ES6模块的实现是有帮助的。最主要的目标是：

+ 推荐使用默认导出项

+ 静态的模块结构

+ 同时支持同步以及异步的加载

+ 支持循环依赖检测

下面的章节将对以上的目标具体解释。

### 推荐使用默认导出项

ES6模块的语法建议“默认的导出项就是这个模块”，这个说法咋听上去有点奇怪。但是它解释了一个主要的设计目标就是使得创建默认的导出项越方便越好。正如**David Herman**所说：

> ES6推荐单独/默认的导出风格，并且对引入默认项提供了语法糖。引入命名的导出项会丧失一些简洁性。

### 静态的模块结构

在当前的javascript模块系统中，需要按顺序执行代码才能找到引入和导出的内容。这也是ES6要打破原有系统的原因：在使用ES6构建模块系统的过程中，可以按照语法得到静态的模块结构。我们来看一下这个过程以及我们从中获取到的好处。

如果一个模块的结构是静态的，这就意味着可以在编译时决定引入和导出，这个过程只需要依据源码进行分析而不用执行它。下面的两个例子举证了为什么CommonJS模块没有办法办到。第一个例子中，必须运行代码才能知道引入的是什么。

```
var mylib;

if(Math.random()) {
	mylib= require('foo');
} else {
	mylib = require('bar');
}
```

第二个例子中必须通过运行代码才能知道要导出什么

```
if(Math.random()) {
	exports.baz = ...;
}
```

ES6虽然会丧失一部分灵活性，但是会让其变的静态。作为结果，将会获得很多收益。

#### 收益一：更快的查找

在CommonJS中引入一个库，可能会得到一个Object

```
var lib = require('lib');
lib.someFunc();	//property lookup
```

因此访问一个命名导出项通过lib.someFunc意味着必须进行属性查找，因为是动态查找，相对来说是很慢的过程。

相对应的，如果在ES6中引入了一个库，可以静态的知道它的内容以及可以优化查找。

```
import * as lib from 'lib';
lib.someFunc();
```

参考[static-module-resolution](http://calculist.org/blog/2012/06/29/static-module-resolution/)

#### 收益二：变量检查

通过模块静态结构，可以知道模块里面所有的变量是否生效。

+ 全局变量：随着发展，完全的全局变量只会出现在适当的场合。剩下的都会是模块（包括标准库和浏览器中提供的方法）。也就是说，可以通过静态检测知道所有的全局变量。

+ 模块载入：通过静态检测，也可以知道哪些模块需要载入

+ 模块作用域下的变量：可以通过静态检测模块内部得到

这在检测一个给定的标识符是否拼写正确有着惊人的作用。这种类型的检测对于当前的检测工具，如JSLint以及JSHint，是未来流行的一个趋势。在ES6中，通过静态检测得到的结果，可以被Javascript引擎进行优化。

另外，对引入文件的访问也是可以被检测到的。如<code>lib.foo</code>。

#### 收益三：对宏的支持

宏已经走在了JavaScript未来的路上。如果一个JavaScript引擎支持宏，我们就可以通过库添加新的语法。[Sweet.js](http://sweetjs.org/)是一个试验性的项目。以下的例子来自于Sweet.js的官网：类的宏。

```
//Define the macro

macro class {
	rule {
		$className {
			constructor $cparams $cbody
			$($mname $mparams $mbody) ...
		}
	} => {
		function $className $cparams $cbody
		$($className.prototype.$mname
			= function $mname $mparams $mbody; ) ...
	}
}

//Use the macro
class Person {
	constructor(name) {
		this.name = name;
	}
	say(msg) {
		console.log(this.name + " says: " + msg);
	}
}

var bob = new Person("Bob");
bob.say("Macros are sweet");
```

处理宏时，JavaScript引擎需要在编译之前对代码进行预处理：如果在语法分析器产生的token流中存在匹配宏的token序列，这部分token将被宏生成的tokens替换。预处理阶段只能在静态的分析宏定义时产生作用。因此，要想引入宏，首先要有静态的结构。

#### 收益四：对静态类型做准备

静态类型检查同宏一样，因为需要静态类型的定义，受到静态结构的限制。如果模块有静态的结构，类型就可以被引入进来。

类型吸引人的地方在于允许通过方言编写更高效的代码。如(LLJS)(http://lljs.org/)，允许手动的控制内存的使用以及回收。当前LLJS编译的结果是[asm.js](http://www.2ality.com/2013/02/asm-js.html)。

#### 收益五：可以使用其它语言

就像上面所说的，要支持编译类型的语言，如宏或者静态类型，那么JavaScript的模块必须拥有静态类型的结构。

### 同时支持异步和同步的模块加载

无论引擎支持同步或者异步的加载方案，ES6模块需要脱离引擎来工作。在语法层面，ES6模块完美的支持同步加载，可以通过静态结构支持异步加载。由于可以通过静态结构决定要引入的模块，可以在不执行模块体的前提下加载模块。

### 支持循环依赖检查

如果两个模块A，B存在循环依赖，即A依赖B，B依赖A。如果可能的话，循环依赖应该被避免，存在上述情况时，AB存在紧耦合的关系，他们没有办法脱离其中一方单独使用。

**为什么需要支持循环依赖？**

从本质上来讲，循环依赖不是邪恶的。尤其是对对象来说，有时是需要这种依赖关系的。比如，在一些树形结构（比如DOM tree）中，父级节点存在对子节点的引用，子节点也需要保存的父级节点的引用。在一些库中，可以通过仔细的设计来避免循环依赖的问题。在大的系统中，出现循环依赖的几率还是很高的，尤其是在重构的过程中。所以循环依赖对这样的系统很适用，这样在重构的时候不会发生系统错误。

Node.js的文档中说明了循环依赖的重要性，Rob Sayre提供了如下的证据：

> Data point: 我在Firefox上实现了一个类似于ES6模块的系统，在发布了仅仅三周之后就被建议添加循环依赖的支持。
> 我工作在Alex Fritze推荐的系统上，它并不完美，语法也不是很漂亮。但是在上面工作了已经7年了，它确实有一些设计的正确的地方。

我们来看一下CommonJS和ES6对循环依赖的处理。

**CommonJS中的循环依赖**

在CommonJS中，如果模块B依赖模块A，此时A模块的模块体已经被执行了，返回A当前状态的导出对象（下面例子中的#1）。这样B可以引用导出的对象（#2）。当B执行完毕时，属性也被赋值完毕，从而B的导出项可以正常工作。

```
//------ a.js ------

var b = require('b');
exprots.foo = function() {...};

//------ b.js ------

var a = require('a');	//(1)
//Cant't use a.foo in module body,
//but it will be filled in latter
exports.bar = function() {
	a.foo();	//OK (2)
}

//------ main.js ------

var a = require('a');
```

作为一个通用的规则，记住在循环引用的情况下，是不能访问引用的模块体的。这是一个固有的情况，不会在ES6的模块中产生改变。

CommonJS提供的方法的局限为：

+ Node.js风格的单值导出将无法工作。在Node.js中，可以导出单值而不是对象，比如：
<code>module.exports = function () {...}</code>
如果模块A的内容如上所示，将没有办法使用模块B中导出的方法，因为模块B中的变量a指向A原始导出的对象。

+ 无法直接使用命名导出项。也就是说，模块B不能像如下的形式引入**a.foo**：
<code>var foo = require('a').foo;</code>
foo的值会是<code>undefined</code>。换句话说，除了在export中使用a对象引用foo函数，没有其他的办法。

CommonJS有一个独特的特性，可以在引入之前执行导出。这样就可以访问到引用模块的模块体内容了。无论如何，这都是一个比较有用的特性。

**ES6中的循环依赖**

为了消除上述两个局限，ES6模块导出的是绑定，而不是值。也就是说，在模块体内部声明的变量的引用还继续生效。如下所示：

```
//------ lib.js ------
export let counter = 0;
export function inc() {
	counter ++;
}

//------ main.js ------
import {inc, counter} from 'lib';
console.log(counter);	//0
inc();
console.log(counter);	//1
```

这里可以看到导出的是对模块中变量的引用，无论该值是否是基本变量还是函数。

因此，面对循环依赖时，不需要区分访问的是一个命名导出项还是通过它的模块：通过间接关系作为访问通道来保证在这些情况下都可以运行。

## 对引入和导出更多的介绍

### 引入

ES6提供如下的引入方式：

```
//默认的导出和命名导出
import theDefault, { named1, named2 } from 'src/myLib';
import theDefault from 'src/myLib';
import { named1, named2 } from 'src/myLib';

//重命名：将named1重命名为myNamed1
import { named1 as myNamed1, named2 } from 'src/myLib';

//将模块导入成一个对象
//(每一个属性是一个命名导出项)
import * as mylib from 'src/myLib';

//单纯的导入模块，不引入任何的属性
import 'src/myLib';
```

### 导出

在当前的模块中，有两种方式可以导出资源。可以通过<code>export</code>关键字进行标识要导出的项。

```
export var myVar1 = ...;
export let myVar2 = ...;
export const MY_CONST = ...;

export function myFunc() {
	...
}

export function* myGeneratorFunc() {
	...
}

export class MyClass {
	...
}
```

<code>default export</code>是一个表达式（包括函数表达式以及类表达式）。例如：

```
export default 123;
export default function(x) {
	return x;
};
export default x => x;
export default class {
	constructor(x, y) {
		this.x = x;
		this.y = y;
	}
};

```

另外，可以把所有需要导出的项放置在模块的末尾，通过对象导出，这属于复古的导出风格。

```
const MY_CONST = ...;
function myFunc() {
	...
}

export { MY_CONST, myFunc };
```

可以在导出时命名导出项

```
export { MY_CONST as THE_CONST, myFunc as theFunc };
```

需要注意的是不能够使用保留字（如<code>default</code><code>new</code>）作为变量的名字，但是可以作为导出项的名字（在ES5中甚至可以作为属性名）。如果想要直接引入使用了保留字的命名导出项，必须在引用的时候对其进行重命名。

### 间接导出

间接导出的意思是在一个模块中导出其他模块的导出项。可以通过如下方式把其他模块所有的导出项通过这个模块导出：

```
export * from 'src/other_module';
```

或者可以有更多的选择（通过重命名导出）：

```
export { foo, bar } from 'src/other_module';

//把其它模块的foo重命名为myFoo导出
export { foo as myFoo, bar } from 'src/other_module';
```

## eval()以及模块

eval()不支持模块的语法。eval函数通过Script语法规则解析参数，但是不支持模块的语法。如果要执行模块的代码，可以通过module加载的API。

## ES6模块加载API

模块除了声明式的语法之外，还有编程性的API。可以允许：

+ 编程式的处理模块和脚本

+ 配置模块加载

加载器能够识别模块的说明符号（比如import...from之后的ID字符串），加载模块等等。构造函数是<code>Reflect.loader</code>。每个平台在全局变量<code>System</code>上都有一个自定义的实例，可以对模块加载有自己的风格。

### 引入模块以及加载脚本

可以通过基于[ES6 Promise](http://www.html5rocks.com/en/tutorials/es6/promises/)编程的引入一个模块。

```
System.import('some_module').then(some_module => {
	//使用 some_module
})
.catch(error => {
	...
})
```

<code>System.import()</code>允许：

+ 使用<code>script</code>元素中的模块（当模块的语法不被支持的时候，更多细节参考下面的内容）

+ 通过条件加载模块

<code>System.import()</code>引入一个单一的模块，可以通过<code>Promise.all()</code>引入多个模块：
 
```
Promise.all(
['module1', 'module2', 'module3']).map(x => System.import(x))
.then(([module1, module2, module3]) => {
	//使用模块1，模块2，模块3
})
```

更多加载的方法有：

+ <code>System.module(source, options?)</code>将source中的代码执行并生成一个模块（在promise形式中执行是异步的。

+ <code>System.set(name, module)</code>用来注册一个模块（例如，已经通过<code>System.module()</code>创建的模块）。

+ <code>System.define(name, source ,options?)</code>通过source中的代码生成一个模块并且通过名字注册

### 配置模块的加载

模块加载的API在配置中提供了丰富的钩子。即使在过程中也能够生效。最初的在浏览器中使用的加载器已经被实现以及通过测试。目标是找到使用模块加载配置的最佳实践。

加载器的API允许自定义的加载过程，例如：

1. 引入之前进行检查，比如使用JSLint或者JSHint。
2. 在引入时进行处理，比如使用了CoffeeScript或者TypeScript代码。
3. 使用遗留的模块，比如AMD，Node.js模块。

## 更多信息

以下内容是关于两个关于ES6模块提出的重要问题：当前如何使用以及如何在HTML中引入。

+ "[当前如何使用ES6模块](http://www.2ality.com/2014/08/es6-today.html)"给出了ES6的使用概览以及如何翻译成ES5的语法。如果看过之后感兴趣的话，可以阅读[下一个章节](http://www.2ality.com/2014/08/es6-today.html#using_ecmascript_6_today)。一个吸引人的解决办法是[ES6模块转化](https://github.com/esnext/es6-module-transpiler)，在ES5的语法中增加了ES6的模块语法并能够把ES6的模块编译成AMD或者是CommonJS模式的代码。

＋ "在HTML中使用ES6模块"：在<code>script</code>标签中的代码不支持ES6模块的语法，因为元素的同步属性与模块的异步加载不兼容。反而，应该使用新的<code>module</code>元素。在[在未来浏览器中的ES6模块](http://www.2ality.com/2013/11/es6-modules-browsers.html)解释了如何使用<code>module</code>标签。比起<code>script</code>有多种优点。可以通过在<code>script</code>标签上添加<code>type="module"</code>来模拟。

+ CommonJS vs. ES6：[Javascript 模块](http://jsmodules.io/)是对ES6模块的入门介绍，有兴趣的话可以继续看[第二节](http://jsmodules.io/cjs.html)，其中有CommonJS与ES6模块的对比图。

## ES6模块的优势

直观上来看，用ES6的模块构建系统有一点点令人厌烦。最后，我们将拥有一些好的模块系统。现在来看，ES6模块的特性，比如紧凑的语法和静态结构（可以用来优化，静态分析等等），不能通过增加JavaScript库来实现。但是，ES6是值得期待的，来打破现在的标准，同时存在的AMD和CMD标准。

拥有一个单一的内置的模块标准意味着：

+ 不会有[UMD](https://github.com/umdjs/umd)这样为了兼容各种规范而来的规范。

+ 新的浏览器API将会变成模块化的，而不是使用全局变量和浏览器提供的API

+ 不会有使用对象作为命名空间的情况：如Math，JSON在ES5中做的。在未来，这样的功能将会通过模块引入

## 参考

1. [Javascript术语表](http://www.2ality.com/2011/06/ecmascript.html)
2. [模块化静态解决方案](http://calculist.org/blog/2012/06/29/static-module-resolution/)
3. [模块：循环](http://nodejs.org/api/modules.html#modules_cycles)，参考Node.js API文档
4. [Imports](https://people.mozilla.org/~jorendorff/es6-draft.html#sec-imports) ES6标准
5. [Exports](https://people.mozilla.org/~jorendorff/es6-draft.html#sec-exports)


































