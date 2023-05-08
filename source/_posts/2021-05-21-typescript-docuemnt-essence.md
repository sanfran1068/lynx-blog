layout: post
title: "TypeScript 文档精粹"
date: 2021-05-21 23:01:00
comments: true
categories: 
- Document
tags:
- TypeScript
- 基础知识
---

### Introduction to TypeScript

- TypeScript is a super set of JavaScript. 一个没有任何预发错误的 js 文件也可以被 ts 正常编译。
- TS 一般会支持到 JS 未来版本 stage3

<!-- more -->

### TypeScript Setup

- install NodeJS
- install TypeScript compiler (npm install -g typescript)

### TypeScript Type

Type 描述一个值拥有什么样的属性和方法，每一个值都有一个类型。Type的主要目的，一是为了 TypeScript 编译器分析代码错误，二是让开发者明确所处理的变量的特征和属性。

- Primitive Type: string, number, boolean, null, undefined, symbol
    - null和undefined: 一般不会用作类型标记，只能赋值给void类型和其自身类型的变量
- Object Types: functions, arrays, classes etc.
    - object vs. Object: object 代表所有非原始类型值，Object 描述 objects 所拥有的能力
    - {} 是一种特殊类型，空对象
    - tuple（元组）: 长度固定、元素确定且无重复的数组，可以有可选项带 ? 表示，通过[type1, type2...]进行标记
    - enum（枚举）: 一组常量，enum Month { Jan = 1, Feb, Mar ... }，定义枚举，默认有各自序号，也可以为每个自行定义
    - array（数组）: 可以通过 type[] 或 Array<type> 进行标记
    - any: 不进行类型检查直接通过编译阶段
    - void: 没有任何类型，一般用于没有返回值的函数
    - never: 用不存在的值的类型，总抛出异常或永无返回值的函数表达式。只能赋值给自身类型的变量
- 类型标记（type annotation: variable: [type]）
- 类型推断（type inference）
变量赋值、函数参数设置默认值、函数返回值类型确定时会自动推断其类型。TS 能够推断出当下变量最合适的一个类型。
- 类型断言：通过“尖括号 <> ”和 as 语法
当变量只声明未赋值时、变量类型无法推断、函数返回值类型不明确等情况是需要类型标记的。

### 接口

- 通过 ? 表示可选属性
- 通过添加 readonly 表示只读属性，只能在对象刚刚创建时赋值
- 如果有未知的需要扩展的属性，可以添加 [propName: string]: any 在接口中
- 方法的接口：（param1: type1, param2: type2): returnType
- 索引（类数组）类型 [index: number]: string，可以通过下标返回对应的值
- 类接口：强制一个类符合某种契约，可以通过 implements 对接口进行实现和拓展
- 接口继承：通过 extends 对接口进行集成，在原有接口上添加更多属性定义
- 混合类型：对象可以同时作为函数和对象使用

### 类

- 类可以通过 extends 进行继承。派生类可以没有构造函数，若写构造函数必须调用 super()。在构造函数里访问 this 的属性之前，我们 一定要调用 super()
- 成员默认为 public；设置为 private 时就不能再声明它的类（包括派生类）外部使用；设置为 protected 时类似 private，但是派生类中可以访问
- ts 使用鸭子类型（结构性类型系统），成员类型兼容（private、protected修饰符也要兼容）即类兼容，类实例可以相互赋值
- 构造函数也可以被标记成 protected。 这意味着这个类不能在包含它的类外被实例化，但是能被继承
- readonly 修饰的成员必须在声明、构造函数中被初始化
- 静态成员，存在在类本身，使用时需要去访问类本身
- abstract 修饰的抽象类作为其他派生类的基类使用，不可直接实例化，但是可以作为一种类型。抽象类中的抽象（abstract 修饰的）方法必须在派生类中实现
- 定义类即定义了实例的类型，类可以当做接口使用

### 函数

- 函数类型由参数类型（参数名并不重要）与返回值类型组成，例如 let myAdd: (x: number, y: number) => number = function(x: number, y: number): number { return x + y; };
- 可选参数必须在参数之后，用 ? 表示，不传时值为 undefined。参数默认值用 = 表示。
- 剩余参数必须在参数之后，用 ...restOfParams 表示，restOfParams 作为参数数组使用。
- 如果需要对函数进行重载，需要给函数设置多个类型

### 泛型

当我们在设计函数或接口类型时，考虑到函数和接口的可复用性，可能会需要对参数和返回值类型进行动态传入，这时就需要类型的变量。反省函数指的就是可以传入类型变量作为参数，从而确定函数参数和返回值的类型。

```tsx
function loggingIdentity<T>(arg: T[]): T[] {
    console.log(arg.length);
    return arg;
}
```

- 泛型只用于函数，泛型使用 <T, U> 来表示（T 为类型参数，可以用任何参数名）
- 泛型接口：`interface GenericIdentityFn<T> { (arg: T): T; }`
- 泛型类：`class GenericNumber<T> { zeroValue: T; }`
- 泛型约束：给泛型增加条件，让泛型不再是任意类型：
    - 使用 extends 关键词实现约束：`<T extends Lengthwise>`
    - 传入参数为实例时：`function create<T>(c: {new(): T; }): T { eturn new c(); }`，`c:{new():T}` 和 `c:new()=>T` 是一样的。new() 在此处时构造函数类型字面量，而 new func() 中 new 是操作符。

### 枚举

- 枚举用来定义一组常量，并且可以给每个枚举定义其数字值，默认从 0 开始；如果定义其值为字符串则不会有自增的特性。
- 枚举是在运行时真正存在的对象，意思是枚举在运行时可以当做对象使用
- 数字值的枚举成员可以类似下标进行反向映射
- const 枚举：在编译阶段会被删除，`const enum Enum { A = 1 }`
- 外部枚举：`declare enum Enum { A = 1 }`

### 类型推论

TS 中，没有明确指出类型，类型推论会帮助提供类型。这种推论只发生在**初始化变量和成员**，**设置默认参数值**和**决定函数返回值**时。

### 类型兼容性

TypeScript里的类型兼容性是基于结构子类型的，结构类型是一种只使用其成员来描述类型的方式，也就是鸭子模型。如果 A 属于 B，那么 B 类型可以赋值给 A 类型（也就是说多属性可以赋值给少属性的，反之则不可以）。那么这两种类型（接口）就是兼容的（兼容指的是这两种类型的变量可以相互赋值）。

- 对象类型兼容：A、B 成员变量有包含关系时，多的可以赋值给少的。
- 函数类型兼容：参数有包含关系时，少参数的可以赋值给多参数的。返回值是简单值可以互相复制，如果是对象类型，遵循对象类型兼容。

### 高级类型

- 交叉类型：<A & B & C>，需要有 A、B、C 中的所有成员。
- 联合类型：`func(a: number | string)`，指一个变量可以是几种类型之一。需要使用类型断言（或 instanceof 操作符）判断到底是哪种类型。
- 类型别名：type NumOrStr = number | string，这不是一种新类型，而是一个名字来引用这些类型。类型别名不能被 extends/implements。
- 可辨识联合：联合的类型中有一个通用字段可以用来辨识各个类型
- keyof 可以直接将对象或对象类型接口的 key 变成联合类型

### Symbol

Symbol 是一种原生类型，通过 Symbol() 构造函数创建。

- Symbol 创建后均为不可变且唯一的（尽管构造时传参相同）。
- Symbol 构造的值可以被用作对象属性的 key，可以作为对象的成员属性名（包括方法）。
- Symbol 有内置的，包括 Symbol.hasInstance、Symbol.isConcatSpreadable、Symbol.iterator、Symbol.match...

### 迭代器和生成器

当一个对象实现了 Symbol.iterator 属性时，就认为这个对象是可迭代的。

- for...of：遍历可迭代的对象，调用对象上的Symbol.iterator方法，返回的是**属性值**。
- for...in：与 for...of 相同，但是返回的是**属性键**。

### 模块

模块在其自身的作用域里执行，而不是在全局作用域里；这意味着定义在一个模块里的变量，函数，类等等在模块外部是不可见的。如果想在不同作用域使用模块，必须使用 import 和 export 进行导入和导出。**若没有标注 import 或 export** 则视为全局作用域。

- 导出
    - 导出声明：任何声明（比如变量，函数，类，类型别名或接口）都能够通过添加export关键字来导出。
    - B 重新导出 A 中的内容，是不会在 B 中添加变量，而是直接导出。
    - 默认导出：每个模块有且仅有一个默认导出 default，类和函数声明可以直接被标记为默认导出。
    - 若使用 `export = someClass/someFunc` 导出一个模块，则必须使用TypeScript的特定语法 `import module = require("someModule")` 来导入此模块。
- 导入
    - 导入模块中某个内容 `import { content } from 'someModule'`；导入整个模块`import * as someModule`。

### 命名空间

一个 ts 文件内部如果想使用模块，可以使用 namespace 关键字来进行声明。

```tsx
namespace Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }
}

let validators: { [s: string]: Validation.StringValidator; } = {};
```

- 就算相同的 namespace 分割在不同文件中，使用时也是同一个 namespace，编译成一个文件时需要添加 --outFile 参数
- namespace 也可以嵌套：`namespace Shapes { export namespace Polygons { export class Triangle { } } }`

### 模块解析

编译器在查找导入的模块内容时，会遵循一定流程。

- 相对导入是以 /，./，../ 开头的文件，相对于导入这些模块的文件的文件地址，不能解析为一个外部模块声明
- 所有其他类型的导入是非相对导入，通过路径映射来解析，可以被解析为外部模块声明
- 模块解析策略有 Classic 和 Node 两种。
    - Classic：TS 以前的默认策略：在 `/root/src/folder/A.ts` 中 `import { b } from './moduleB';`从包含导入文件的目录依次向上级目录遍历
    /root/src/folder/moduleB.ts
    /root/src/folder/moduleB.d.ts
    /root/src/moduleB.ts
    /root/src/moduleB.d.ts
    /root/moduleB.ts
    /root/moduleB.d.ts
    /moduleB.ts
    /moduleB.d.ts
    - Node：通常在 NodeJS 中是通过 require 函数来导入模块的。而根据相对/非相对路径会有不同查找行为
        - 相对路径：在文件 `/root/src/folder/A.ts` 中 `import x = require('./moduleB');`
            - 检查 `/root/src/moduleB.js` 文件是否存在
            - 检查 `/root/src/moduleB`目录是否包含一个 package.json 文件，且 package.json 文件指定了一个 "main" 模块
            - `/root/src/moduleB` 目录是否包含一个 index.js 文件
        - 非相对路径：在文件 `/root/src/folder/A.ts` 中 `import x = require('moduleB');`
            - 首先在同一级目录下的 node_modules 目录中进行寻找，其次会在父级目录下的 node_modules 目录中进行寻找，以此向根目录方向查找：
            `/root/src/node_modules/moduleB.js` => `/root/src/node_modules/moduleB/package.json` => `/root/src/node_modules/moduleB/index.js/root/node_modules/moduleB.js` => `/root/node_modules/moduleB/package.json` => `/root/node_modules/moduleB/index.js`
    - TypeScript（模仿 Node.js 运行时的解析策略来编译定位模块定义文件）：
        - 引入相对路径的文件时：
            - 从本级目录向根目录方向查找
            - 在同级目录下，会依据 moduleB.ts => moduleB.tsx => moduleB.d.ts => moduleB/package.json => moduleB/index.ts => moduleB/index.tsx => moduleB/index.tsx => moduleB/index.d.ts
        - 引入绝对路径的模块时：
            - 从本级目录下的 node_modules 向根目录下的 node_modules 方向查找
            - 在同级目录下，同“引入相对路径”相同
    
    在引入的文件是相对路径时，路径计算都是相对于当前路径计算；如果引入文件是绝对路径时，会根据 “tsconfig.json” 里的 baseUrl 属性值来计算；如果想指定某些 npm 包的解析路径，可以在 ”tsconfig.json” 中的 compilerOptions.paths 中进行设置，而 paths 的值是基于 baseUrl 进行解析。
    
    ### 声明合并
    
    指编译器将针对同一个名字的两个独立声明合并为单一声明。任何数量的声明都可被合并；不局限于两个声明。
    
    TypeScript 中的声明可以产生**命名空间**、**类型**和**值**三种实体。
    
    - 接口合并
        - 接口的非函数的**成员应该是唯一**的。如果**不是唯一的，那么必须是相同的类型**。如果同名但类型不同，则编译器会报错。
        - 如果成员为函数，每个同名函数声明都会被当成这个函数的一个重载。
        - 接口内部成员声明顺序不变，接口间则是后声明的接口成员重载出现会排在前面
    
    ```tsx
    interface Box {
        height: number;
        width: number;
    }
    
    interface Box {
        scale: number;
    }
    
    let box: Box = {height: 5, width: 6, scale: 10};
    ```
    
    - 合并命名空间
        - namespace 中的 export 成员相互是可访问
        - namespace 中的非 export 成员，其他命名空间合并来的是无法访问的
    
    ```tsx
    namespace Animal {
        let haveMuscles = true;
    
        export function animalsHaveMuscles() {
            return haveMuscles;
        }
    }
    
    namespace Animal {
        export function doAnimalsHaveMuscles() {
            return haveMuscles;  // Error, because haveMuscles is not accessible here
        }
    }
    ```
    
    - 合并命名空间和类：可以实现内部类
    
    ```tsx
    class Album {
        label: Album.AlbumLabel;
    }
    namespace Album {
        export class AlbumLabel { }
    }
    ```
    
    - 合并命名空间和函数
    
    ```tsx
    function buildLabel(name: string): string {
        return buildLabel.prefix + name + buildLabel.suffix;
    }
    
    namespace buildLabel {
        export let suffix = "";
        export let prefix = "Hello, ";
    }
    ```
    
    - 合并命名空间和枚举：扩展枚举型
    
    ```tsx
    enum Color {
        red = 1,
        green = 2,
        blue = 4
    }
    
    namespace Color {
        export function mixColor(colorName: string) {
            if (colorName == "yellow") {
                return Color.red + Color.green;
            }
            else if (colorName == "white") {
                return Color.red + Color.green + Color.blue;
            }
            else if (colorName == "magenta") {
                return Color.red + Color.blue;
            }
            else if (colorName == "cyan") {
                return Color.green + Color.blue;
            }
        }
    }
    ```
    
    ### JSX
    
    - 使用 JSX 需要：文件改为 .tsx 扩展名，启用 jsx 选项（preserve、react 和 react-native）
    - JSX 结果类型默认为 any，可以指定 JSX.Element 接口进行自定义
    - JSX 允许使用 {} 内嵌表达式
    - as 操作符：.tsx 文件不支持 <foo>bar 的类型断言，必须使用 bar as foo
    - 元素类型检查：
        - 固有元素会生成字符串，以小写字母开头，在 JSX.IntrinsicElements 接口中查找
        - 值元素（无状态函数组件、类组件），以大写字母开头，在其作用域里按标识符查找
            - 无状态函数组件：组件被定义成  Javascript 函数，第一个参数为 props 对象
            - 类组件：<Expr />，元素类的类型即 Expr 的类型；如果组件是 ES6 的类，类类型就是类的**构造函数+静态部分**，如果组件是工厂函数，类类型就是这个**函数**。元素的实例类型必须赋值给 `JSX.ElementClass` 或抛出一个错误。
    
    ```tsx
    class MyComponent {
        render() {}
    }
    var myComponent = new MyComponent();
    // 元素类的类型 => MyComponent
    // 元素实例的类型 => { render: () => void }
    
    function MyFactoryFunction() {
        return {
    		    render: () => {
    		    }
        }
    }
    var myComponent = MyFactoryFunction();
    // 元素类的类型 => FactoryFunction
    // 元素实例的类型 => { render: () => void }
    ```
    
    - 元素的属性类型检查
        - 固有元素：还是在 JSX.IntrinsicElements 接口中查找
        - 值元素：在元素实例的 props 中指定属性
    
    ### 装饰器
    
    装饰器是一种特殊类型的声明，它能够被附加到[类声明](https://www.tslang.cn/docs/handbook/decorators.html#class-decorators)，[方法](https://www.tslang.cn/docs/handbook/decorators.html#method-decorators)， [访问符](https://www.tslang.cn/docs/handbook/decorators.html#accessor-decorators)，[属性](https://www.tslang.cn/docs/handbook/decorators.html#property-decorators)或[参数](https://www.tslang.cn/docs/handbook/decorators.html#parameter-decorators)上。 装饰器使用 `@expression` 这种形式，`expression` 求值后必须为一个函数，它会在运行时被调用，被装饰的声明信息做为参数传入。
    
    - 装饰器工厂函数
    
    ```tsx
    function color(value: string) { // 这是一个装饰器工厂，传入的是与 target 无关的参数
        return function (target) { //  这是装饰器，target 就是类、方法等
            // do something with "target" and "value"...
        }
    }
    ```
    
    - 装饰器组合：类似复合函数 **f(g(x))**，由上至下对装饰器表达式求值，由下至上将求值结果进行传参调用
    
    ```tsx
    @f
    @g
    x
    ```
    
    - 装饰器分为：类装饰器、参数装饰器、方法装饰器、访问符装饰器和属性装饰器。
        
        类装饰器应用于**构造函数**，用于监视、修改和替换类定义，不能用于声明文件（.d.ts）和外部上下文（declare 的类）。传参仅构造函数一个。
        
        ```tsx
        // @sealed 类装饰器：类将不可增减属性、属性也不可配置
        function sealed(constructor: Function) {
            Object.seal(constructor);
            Object.seal(constructor.prototype);
        }
        ```
        
    - 方法装饰器
        
        方法声明之前声明，会被应用到方法的属性描述符上，用来监视、修改和替换方法定义。传参有三个：类的构造函数（对于静态成员）/类实例的原型对象（对于实例成员）、成员名字、成员的属性描述符。返回值会被用作方法的*属性描述符*（如果输出版本大于ES5）
        
        ```tsx
        function enumerable(value: boolean) {
            return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
                descriptor.enumerable = value;
            };
        }
        ```
        
    - 访问器装饰器：和成员方法装饰器类似，返回值会被用作方法的*属性描述符*（如果输出版本大于ES5）
        
        ```tsx
        function configurable(value: boolean) {
            return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
                descriptor.configurable = value;
            };
        }
        class Point {
            // ...
            @configurable(false)
            get x() { return this._x; }
        }
        ```
        
    - 属性装饰器
        
        属性装饰器作用是用来监视类中是否有某个名字的属性。传参有两个：类的构造函数（对于静态成员）/类实例的原型对象（对于实例成员）、成员名字。返回值会被忽略。
        
        ```tsx
        function format(formatString: string) {
            return Reflect.metadata(formatMetadataKey, formatString);
        }
        class Greeter {
            @format("Hello, %s")
            greeting: string;
        }
        ```
        
    - 参数装饰器
        
        运行时当做函数被调用，返回值会被忽略。
        
        ```tsx
        function required(target: Object, propertyKey: string | symbol, parameterIndex: number) {}
        function validate(target: any, propertyName: string, descriptor: TypedPropertyDescriptor<Function>) {}
        class Greeter {
            @validate
            greet(@required name: string) {
                return "Hello " + name + ", " + this.greeting;
            }
        }
        ```
        

### Mixins

设计一个类可以同时具有多个类的属性和方法。

```tsx
class Disposable {
  isDisposed: boolean;
  dispose() {
    this.isDisposed = true;
  }
}

class Activatable {
  isActive: boolean;
  activate() {
    this.isActive = true;
  }
  deactivate() {
    this.isActive = false;
  }
}

class SmartObject implements Disposable, Activatable {
	// 因为要混合多个类，所以只能用 implements 而非 extends，但是原有类的方法实现没法直接使用
	constructor() {
    setInterval(() => console.log(this.isActive + " : " + this.isDisposed), 500);
  }

  interact() {
    this.activate();
  }

  // Disposable
  isDisposed: boolean = false;
  dispose: () => void;

  // Activatable
  isActive: boolean = false;
  activate: () => void;
  deactivate: () => void;
}
applyMixins(SmartObject, [Disposable, Activatable]);

// 这里就是将具体方法实现也混合到结果类中
function applyMixins(derivedCtor: any, baseCtors: any[]) {
  baseCtors.forEach(baseCtor => {
    Object.getOwnPropertyNames(baseCtor.prototype).forEach(name => {
      derivedCtor.prototype[name] = baseCtor.prototype[name];
    });
  });
}
```

### 三斜线指令

三斜线指令是包含单个XML标签的单行注释。 注释的内容会做为编译器指令使用。三斜线指令只能放在文件的最顶端，它的前面只能出现单行/多行注释或其他三斜线指令，否则会被当成普通注释。

```tsx
/// <reference path="..." />
// 最常用的指令，用于声明文件间的依赖，告诉编译器在编译过程中要引入的额外的文件
```

- 预处理输入文件
    
    编译器会对输入文件进行预处理来解析所有三斜线引用指令。 在这个过程中，额外的文件会加到编译过程中。
    
- 错误：引用不存在的文件会报错。 一个文件用三斜线指令引用自己会报错。
- 使用 `--noResolve` ：指定该编译选项，三斜线指令会被忽略。
- `/// <reference types="..." />` 和 `/// <reference path="..." />` 类似，用来声明依赖，与 import 类似。
- `/// <reference no-default-lib="true"/>` 将一个文件标记成默认库，在编译过程中不能编译这个默认库，类似 —noLib 命令
- `/// <amd-module />`

### Javascript 文件类型检查

TypeScript 2.3以后的版本支持使用`--checkJs`对`.js`文件进行类型检查和错误提示。

- 设置是否进行检查的方式：
    - 如果设置了 —checkJs 配置，可以通过添加`// @ts-nocheck`注释来忽略类型检查。
    - 如果没有 —checkJs 配置，可以通过去掉`--checkJs`设置并添加一个`// @ts-check`注释来选则检查某些`.js`文件。
    - 可以使用`// @ts-ignore`来忽略本行的错误。
- 使用 JSDoc 表示类型信息，表示类属性的类型信息
    
    ```tsx
    /** @type {number} */
    var x;
    
    class C {
        constructor() {
            /** @type {number | undefined} */
            this.prop = undefined;
            /** @type {number | undefined} */
            this.count;
        }
    }
    ```
    
- ES2015中，构造函数等同于类
- .js 文件中，ts能够识别出 CommonJS 模块
- 类、函数和对象字面量都是命名空间
- 对象属性字面量是开放的，对象中的属性可以被赋值任意值，也可以添加任意属性
- ****null，undefined，和空数组的类型是any或any[]****
- 函数参数是默认可选的，但是不能传过多的参数；未指定类型的参数默认为 any