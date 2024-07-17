---
layout: post
title: "Angular 学习笔记"
date: 2021-06-01 22:01:00
comments: true
categories: 
- Document
tags:
- Angular
- 基础知识
---

## 快速上手

[Angular - Angular 入门：你的第一个应用](https://angular.cn/start)

首先跟随 Angular 官方文档的快速上手部分进行一个简单项目的代码编写尝试和 Firebase 部署。该示例项目中包括了对

- **模板语法**（的一些基本特性 *ngFor、*ngIf、差值双花括号、属性绑定[]、事件绑定()）
- **组件**（由一个组件类、一个 HTML 模板和组件专属样式表组成）
- 父组件向子组件**输入**信息（@Input）和**输出**（@Output、EventEmitter）
- **路由**（在 app.module.ts 中注册路由、routerLink 属性、通过 ActivatedRoute 作为构造函数参数来使用 this.route 对象）
- **管理数据**（通过创建 service 类对数据进行管理，service 在引用后需要作为参数传入构造函数）
- **数据获取**（使用 Angular 内置的 HttpClientModule 类实例 http 作为参数传入构造函数后，可以使用 this.http 对象请求数据）
- 表单（使用内置的 FormBuilder 构建表单模型）
- 构建和部署（ng build、firebase 部署）

<!-- more -->

## Angular 架构概览

### 基本概念

Angular 是一个用 HTML 和 TypeScript 构建客户端应用的平台与框架。Angular 的**基本构造块是模块 NgModule**，它为组件提供了编译的上下文环境。应用**至少会有一个用于引导应用的根模块 AppModule**，通常还会有很多特性模块。模块通常由组件 Component 构成，组件定义视图（每个应用都至少有一个根组件），组件使用服务（service 提供与视图无关的逻辑功能，服务可注入到组件中）。

模块、组件和服务都是使用装饰器的类。

组件类一般紧跟随在 @Component() 装饰器后。组件提供两种数据绑定类型：事件绑定（处理用户输入的数据）和属性绑定（计算结果插入模板中），也支持双向数据绑定（ngModel），还可以通过**管道**转换要显示的数据。

服务类一般紧跟随在 @Injectable() 装饰器后。该装饰器上的元数据可以将服务作为以来注入到组件中。（服务可以使组件更简洁，复用度更高）

Angular 还提供了 Router 服务来帮助你定义视图之间的导航路径。

### 模块

Angular 应用是模块化的，NgModule 是其模块系统。一个 NgModule 就是一个存放内聚代码块的容器，包含一些组件、服务提供者或其它代码文件。

每个 Angular 应用都**至少有一个 NgModule 类**，也就是根模块，它习惯上命名为 **AppModule**，并位于一个名叫 **app.module.ts** 的文件中。引导这个根模块就可以启动你的应用。应用可以有其他特性模块，但是根模块可以包含任意深度、层次的子模块。

NgModule 是一个带有 @NgModule() 装饰器的类。@NgModule() 装饰器是一个函数，它接受一个元数据对象，该对象的属性用来描述这个模块：

- `declarations`（可声明对象表）：声明属于本 NgModule 的组件、指令、管道。
- `exports`（导出表）：那些能在其它模块的**组件模板**中使用的可声明对象的子集。
- `imports`（导入表）：本 NgModule 中所需要使用的其他模块。
- `providers`：本模块向全局服务中贡献的那些服务的创建器。 这些服务能被本应用中的任何部分使用。（你也可以在组件级别指定服务提供者，这通常是首选方式。）
- `bootstrap` —— 应用的主视图，称为**根组件**。它是应用中所有其它视图的宿主。只有**根模块**才应该设置这个 `bootstrap` 属性

### 组件

**组件元数据**

组件是跟在 @Component() 装饰器后的一个 TypeScript 类，装饰器中需要设置一些元数据，其中包括：selector（CSS 选择器）、templateUrl/template（模板文件或模板）、providers（当前组件所需的服务提供者数组）和 styleUrl/styles（样式文件数组或样式数组）等。

**模板语法**

模板语法很像标准的 HTML 文档，但是有以下几种特性：

- 数据绑定（**绑定语法中，等号左边的不是 HTML 的 attr，而是 DOM 的 property 和事件！但是如果一定要设置 attr 且 DOM 的 property 中没有对应属性，可以使用 attr.someAttr 进行绑定**）：
    
    ```jsx
    // 单项数据绑定
    // 插值
    <li>{{hero.name}}</li>  
    // 属性绑定，父组件向子组件传数据  
    <app-hero-detail [hero]="selectedHero"></app-hero-detail> 
    // 事件绑定，调用方法进行数据处理     
    <li (click)="selectHero(hero)"></li>           
    
    // 双向数据绑定，属性绑定+事件绑定
    <input [(ngModel)]="hero.name">
    
    // attribute 绑定
    <tr><td [attr.colspan]="1 + 1">One-Two</td></tr>
    <button [attr.aria-label]="actionName">{{actionName}} with Aria</button>
    
    // CSS 类绑定使用 [class]时，如果badCurly 有值会替换前面 class 所有类，建议指定某个特定的类
    <div class="bad curly special" [class]="badCurly">Bad curly</div>
    <div [class.special]="isSpecial">The class binding is special</div>
    // 使用ngClass 可以设置一个类对象
    this.currentClasses =  {
        'saveable': this.canSave,
        'modified': !this.isUnchanged,
        'special':  this.isSpecial
      };
    <div [ngClass]="currentClasses">This div is initially saveable, unchanged, and special</div>
    ```
    

ngClass、ngStyle 都是类似的，将一个类/样式的对象绑定在元素上；

ngModel 可以用来双向绑定，使用时**一定需要引入 FormsModule**（在相应的组件/模块中 imports FormsModule 模块），ngModel 同时还提供了一个 (ngModelChange) 的绑定的输出属性可以处理数据变化时的逻辑。

**管道**

可以在模板中展示数据时进行一定的逻辑变换（左侧的数据经过右侧管道函数处理得到的值）。**管道可以进行串联，可以传参（管道：参数）**Angular 自带了很多管道，例如 date、currency 等：

```html
<p>Today is {{today | date: 'longDate' | lowercase }}</p>
```

自定义管道

```jsx
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({name: 'exponentialStrength'})
export class ExponentialStrengthPipe implements PipeTransform {
  transform(value: number, exponent: string): number {
    let exp = parseFloat(exponent);
    return Math.pow(value, isNaN(exp) ? 1 : exp);
  }
}
```

纯管道与非纯管道

数据更新会触发管道执行数据处理，Angular 只在检测到纯变更（指对**原始类型值的更改**，或对对象**引用的更改**），对对象、数组内部的改变是不会调用纯管道的。非纯管道是可以侦测到引用类型内部变化的，Angular 会在每个组件的变更检测周期执行该管道，成本很高。

```jsx
@Pipe({
  name: 'flyingHeroesImpure',
  pure: false / true // 默认为true
})
```

AsyncPipe，接受一个 Promise 或 Observable，自动订阅这个输入最终返回他们发送的值。 

**安全导航操作符 ?. 和非空断言操作符 !.**

?. 避免出现TypeError: Cannot read property 'name' of null in [null].的错误

```html
<p>The current hero's name is {{currentHero?.name}}</p>
```

!. 表示.之前的变量一定存在，一般是在 *ngIf 判断后使用

```html
<div *ngIf="hero">
  The hero's name is {{hero!.name}}
</div>
```

**类型转换函数 $any()**

绑定的模板表达式中不清楚某个变量类型，可以强制转换为 any 类型

```html
<div>The hero's marker is {{$any(hero).marker}}</div>
```

**用户输入**

一般会在表单中为表单元素绑定用户输入事件，可以通过传入 $event 来获取用户输入。但是 $event 不仅很大而且有时并没有 value 属性。

采用模板引用变量 #someInput 是更好的方法。

```jsx
// template
<input (keyup)="onKey1($event)">
<input #input1 (keyup)="onKey2(input1.value)">

// $event bad
onKey1(event: KeyboardEvent) { // with type info
  this.values += (<HTMLInputElement>event.target).value + ' | ';
}

// #input1 good
onKey1(value: any) { // with type info
  this.values += value + ' | ';
}
```

用户输入事件有很多，有按键事件（keyup、keydown）、失焦事件（blur）和鼠标事件（mouseup）按键事件过滤，采用类似 `keyup.enter` 表示只有当 enter 键释放时才处理。

**指令**

Angular 中有**结构型指令**（通过添加、移除或替换 DOM 元素来修改布局，例如 *ngIf 和 *ngFor）和**属性型指令**（修改现有元素的外观或行为，例如 ngModel）。也可以通过 @Directive() 装饰器自定义一些指令。

*ngIf 与 v-if 类似，是控制元素是否存在，与显隐逻辑不通；

*ngFor 的模板语法是 let...of...，需要带索引时，则需要声明索引变量 let i = index；Angular 中提供了一个 trackBy（是一个函数类型，需要在数据中提供），其作用与 key 相同能够提升性能。

```html
trackByHeroes(index: number, hero: Hero): number { return hero.id; }
<div *ngFor="let hero of heroes; let i=index; trackBy: trackByHeroes">{{i + 1}} - {{hero.name}}</div>
```

ngSwitch 指令是属性型指令（修改元素行为而非 DOM 结构）由三部分构成：ngSwitch，*ngSwitchCase，*ngSwitchDefault

```html
<div [ngSwitch]="currentHero.emotion">
  <app-happy-hero    *ngSwitchCase="'happy'"    [hero]="currentHero"></app-happy-hero>
  <app-sad-hero      *ngSwitchCase="'sad'"      [hero]="currentHero"></app-sad-hero>
  <app-confused-hero *ngSwitchCase="'confused'" [hero]="currentHero"></app-confused-hero>
  <app-unknown-hero  *ngSwitchDefault           [hero]="currentHero"></app-unknown-hero>
</div>
```

创建指令，必须使用 @Directive 装饰器（ElementRef 是标签元素引用的类型）：

```jsx
import { Directive, ElementRef } from '@angular/core';

@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
    constructor(el: ElementRef) {
       el.nativeElement.style.backgroundColor = 'yellow';
    }
}
```

创建根据事件触发的指令（HostListener 装饰器可以订阅该指令所在的 DOM 元素的事件）：

```jsx
<p appHighlight [highlightColor]="color">Highlighted with parent component's color</p>

import { Directive, ElementRef, HostListener } from '@angular/core';
@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
	@Input highlightColor: string;
  constructor(el: ElementRef) {}
	@HostListener('mouseenter') onMouseEnter() {
	  this.highlight(this.highlightColor);
	}
	
	@HostListener('mouseleave') onMouseLeave() {
	  this.highlight(null);
	}
	
	private highlight(color: string) {
	  this.el.nativeElement.style.backgroundColor = color;
	}
}
```

<ng-template> 指令，单独使用时，里面的元素都不会显示（转换为注释）。除非有 ngIf，ngFor 或 ngSwitch 等结构型指令。此时应该使用[ngIf](是 *ngif 的原始版本，* ngif 会转换成[ngif]*)

可以为 <ng-template> 添加模板引用 #someTemp，然后使用 *ngTemplateOutlet 指令去服用这些模板引用。

```jsx
<div *ngIf="hero" >{{hero.name}}</div>
// equals to 
<ng-template [ngIf]="hero">
  <div>{{hero.name}}</div>
</ng-template>`

```

<ng-container> 是一个分组元素，且不会污染样式和布局，不存在 DOM 中。

**模板引用变量**

用来引用模板中的某个 DOM 元素，它还可以引用 Angular 组件或指令或Web Component。**不要在同一个模板中多次定义同一个变量名!**

```jsx
// #phone 代表 input 这个元素
<input #phone placeholder="phone number">
<button (click)="callPhone(phone.value)">Call</button>

// #heroForm="ngForm" 可以将引用内容进行修改
<form (ngSubmit)="onSubmit(heroForm)" #heroForm="ngForm"></form>
```

**输入和输出属性**

输入和输出属性在父组件上呈现为数据绑定属性，在子组件中需要有 @Input 和 @Output 装饰器声明流入的成员。

输入属性是一个带有 @Input 装饰器的可设置属性，输入属性通常接收数据值。

输出属性是一个带有 @Output 装饰器的可观察对象型的属性。 这个属性几乎总是返回 Angular 的**EventEmitter**，输出属性暴露事件生产者。

输入和输出属性可以使用别名，第一种方式是在装饰器中传入别名字符串，第二种是在元数据的 inputs 和 outputs 数组中用冒号右边表示别名：

```jsx
@Output('myClick') clicks = new EventEmitter<string>();
@Directive({
  outputs: ['clicks:myClick']  // propertyName:alias
})
```

### 服务

服务是一个广义的概念，它包括应用所需的任何值、函数或特性。但是狭义上，服务就是脱离视图的一些定义了用途的类。将组件和服务分开，可以提高模块性和复用性，让代码更加精简。组件的工作只管用户体验，而应该把诸如从服务器获取数据、验证用户输入或直接往控制台中写日志等工作委托给各种服务。

**依赖注入**

服务是用 @Injectable() 装饰器来提供元数据的类。提供服务需要在 service 类的装饰器 @Injectable() 中注册服务提供者 providedIn 字段。服务还可以在模块 NgModule 和组件级别进行注入，都是在装饰器中添加 providers 字段。

## 主要概念

### 组件

**组件的构成**

每一个组件可以使用一个目录进行包含，其中包括 HTML 模板、一个 TypeScript 类（包含应用数据和逻辑）和一个模板的样式（可选）。其中在 TypeScript 类中需要指明组件的 CSS 选择器（即组件标签名），以及模板文件和样式文件。

```jsx
// [name].component.ts
@Component({
  selector: 'app-component-overview',
  templateUrl: './component-overview.component.html',
  styleUrls: ['./component-overview.component.css']
})
```

CSS 选择器一般以 app 开头，例如一个组件标签为 <app-hello-world>，那么它的 CSS 选择器（selector）就应当是 app-hello-world，组件 ts 类文件最好命名为 hello-world.component.ts。模板和样式文件亦然。

HTML 模板可以使用外部的 templateUrl，也可以使用 template 属性指定一段 HTML 代码，可以使用模板字符串来实现。**templateUrl 和 template 两个字段不能同时出现**。

styleUrls 字段以**字符串数组**形式指定模板的样式文件，也可以使用 styles 字段（字符串数组类型）来内置样式。

**组件生命周期和钩子**

Angular 中生命周期钩子由 TypeScript 的 Interface 实现，在组件或指令类实现中，需要 implements 相应的钩子接口，每个接口都有唯一的一个钩子方法，方法名即接口名前加 on。

```jsx
@Directive()
export class PeekABooDirective implements OnInit {
  constructor(private logger: LoggerService) { }

  // implement OnInit's `ngOnInit` method
  ngOnInit() { this.logIt(`OnInit`); }

  logIt(msg: string) {
    this.logger.log(`#${nextId++} ${msg}`);
  }
}
```

生命周期顺序如下：

- ngOnChanges()：设置或重新设置数据绑定的输入属性时响应。 该方法接受当前和上一属性值的 SimpleChanges 对象，该钩子频繁发生，钩子中的操作会十分影响性能
- ngOnInit()：第一次显示数据绑定和设置指令/组件的输入属性之后，初始化指令/组件。在第一轮 ngOnChanges() 完成之后调用，只调用一次
- ngDoCheck()：检测，并在发生 Angular 无法或不愿意自己检测的变化时作出反应
- ngAfterContentInit()：把外部内容投影进组件视图或指令所在的视图之后调用
- ngAfterContentChecked()：检查完被投影到组件或指令中的内容之后调用
- ngAfterViewInit()：初始化完组件视图及其子视图或包含该指令的视图之后调用
- ngAfterViewChecked()：每当 Angular 做完组件视图和子视图或包含该指令的视图的变更检测之后调用
- ngOnDestroy()：每当 Angular 每次销毁指令/组件之前调用并清扫，事件监听解绑、反订阅可观察对象和停止计时器等操作可在此钩子进行。

**视图封装**

组件的 CSS 样式被封装进了自己的视图中，而不会影响到应用程序的其它部分。视图封装有三种模式：

- `ShadowDom` 模式使用浏览器原生的 Shadow DOM 实现，来为组件的宿主元素附加一个 Shadow DOM。组件的视图被附加到这个 Shadow DOM 中，组件的样式也被包含在这个 Shadow DOM 中。(不进不出，没有样式能进来，组件样式出不去。)
- `Emulated` 模式（**默认值**）通过预处理（并改名）CSS 代码来模拟 Shadow DOM 的行为，以达到把 CSS 样式局限在组件视图中的目的。(只进不出，全局样式能进来，组件样式出不去)
- `None` 意味着不使用视图封装。 Angular 会把 CSS 添加到全局样式中。而不会应用上前面讨论过的那些作用域规则、隔离和保护等。 从本质上来说，这跟把组件的样式直接放进 HTML 是一样的。

**组件交互**

- 输入型绑定（父传子）：父组件通过在模板上添加属性指令 [] 将数据传入子组件，子组件需要在类中使用 @Input 装饰器引入对应的数据变量名
    - setter 拦截输入数据变化：在上面输入型绑定的基础上，子组件中可以对传入的属性进行 get 和 set 重写，在 set 方法中可以拦截输入数据变化
    - ngOnChanges() 钩子监听数据变化：在上面输入型绑定的基础上，子组件的类可以实现 onChanges 接口，在 ngOnChanges 方法中监听输入数据的变化
    
    ```jsx
    // parent component
    import { Component } from '@angular/core';
    
    @Component({
      selector: 'app-name-parent',
      template: `<app-name-child *ngFor="let name of names" [name]="name"></app-name-child>`
    })
    export class NameParentComponent {
      names = ['Dr IQ', '   ', '  Bombasto  '];
    }
    ```
    
    ```jsx
    // child component
    import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';
    
    @Component({
      selector: 'app-name-child',
      template: '<h3>"{{name}}"</h3>'
    })
    export class NameChildComponent implements OnChanges {
      @Input()
      get name(): string { return this._name; }
      set name(name: string) {
        this._name = (name && name.trim()) || '<no name set>';
      }
      private _name = '';
    
    	ngOnChanges(changes: SimpleChanges) {
    		console.log(changes);
    	}
    }
    ```
    
- 父组件监听子组件的事件：子组件暴露一个 EventEmitter 属性，当事件发生时，子组件利用该属性 emits (向上弹射)事件。父组件绑定到这个事件属性，并在事件发生时作出回应。
    
    ```jsx
    // parent component
    import { Component } from '@angular/core';
    
    @Component({
      selector: 'app-vote-taker',
      template: `
    		<p>{{clickCount}}</p>
        <app-child (changeMsg)="onMsg($event)"></app-child>
    		`
    })
    export class VoteTakerComponent {
      message = '';
    
      onMsg(msg: string) {
        this.message = msg.length > 0 ? msg : '';
      }
    }
    ```
    
    ```jsx
    import { Component, EventEmitter, Input, Output } from '@angular/core';
    
    @Component({
      selector: 'app-voter',
      template: `
        <button (click)="onChangeMsg('a')">Show A</button>
        <button (click)="onChangeMsg('b')">Show B</button>
      `
    })
    export class VoterComponent {
      @Input()  name: string;
      @Output() changeMsg = new EventEmitter<string>();
    
      onChangeMsg(str: string) {
        this.changeMsg.emit(str);
      }
    }
    ```
    
- 父组件、子组件通过本地变量交互，父组件在使用子组件时，在子组件标签中添加 #[tagName] 可以以此来表示子组件引用，通过 tagName 来访问子组件的属性和方法等（该方法只能在父组件模板中进行子组件引用的使用）
- 为了能够在父组件的类中调用子组件引用，父组件可以调用 @ViewChild()：
    
    ```jsx
    export class ParentComponent {
      @ViewChild(ChildComponent)
      private childComponent: ChildComponent;
    
      start() { this.childComponent.someFun(); }
    }
    ```
    
- 父组件和它的子组件共享同一个服务，利用该服务在组件家族内部实现双向通讯。父组件中需要在装饰器的 provider 传入该服务，父子组件构造函数中都需要有该服务作为参数传入。

**组件样式**

组件在 @Component 装饰器里可以通过 styles 字段（字符串数组）设置组件样式，也可以通过 styleUrls 字段（字符串数组）设置外部的样式文件。这两种方式都只会影响本组件，不会影响其他组件、模块以及投影进来的组件。

可以通过模板中内联样式来设置组件的样式：

```jsx
@Component({
  selector: 'app-hero-controls',
  template: `
    <style>
      button {
        background-color: white;
        border: 1px solid #777;
      }
    </style>
    <button (click)="activate()">Activate</button>
  `
})
```

还可以通过模板中添加 link 标签设置组件样式

```jsx
@Component({
  selector: 'app-hero-team',
  template: `
    <link rel="stylesheet" href="../assets/hero-team.component.css">
    <ul>
      <li *ngFor="let member of hero.team">
        {{member}}
      </li>
    </ul>`
})
```

组件目录下的样式文件中可以使用 `@import` 语法引用其他样式文件。

**动态组件**

**Angular 元素和自定义元素**

### 模板

**插值**

在模板中，可以使用双花括号在标记文本中嵌入表达式，Angular 对所有双花括号中的表达式求值，把求值的结果转换成字符串，并把它们跟相邻的字符串字面量连接起来。

模板表达式不能使用具有或可能引发副作用的 JavaScript 表达式：

- 赋值 (`=`, `+=`, `=`, `...`)
- `new`、`typeof`、`instanceof` 等运算符
- 使用 `;` 或 `,` 串联起来的表达式
- 自增和自减运算符：`++` 和 `-`
- 一些 ES2015+ 版本的运算符
- 不支持位运算，比如 | 和 &
- 只能使用表达式上下文的成员，不能使用 window、document 等全局命名空间的任何东西

模板表达式上下文，指的是在这个组件实例中各种绑定值的来源。由模板变量、指令的上下文变量（如果有）和组件的成员叠加而成的，其优先级为模板变量 > 指令的上下文变量 > 组件的成员。

**模板语句**

是可在 HTML 中用于响应用户事件的方法或属性。 `(click)="deleteHero()"` 等号右边引号中的就是模板语句。模板语句不允许使用以下 JavaScript 和模板表达式语法：

- `new`
- 递增和递减运算符 `++` 和 `-`
- 赋值运算符，例如 `+=` 和 `=`
- 按位运算符，例如 `|` 和 `&`
- [管道操作符](https://angular.cn/guide/pipes)

使用插值表达式还是属性绑定？**如果数据类型不是字符串，必须使用属性绑定**。

**管道**

管道是一些简单的函数，用来对字符串、货币金额、日期和其他显示数据进行转换和格式化。可以在**模板表达式**中用来接受输入值并返回一个转换后的值。

```html
<p>The hero's birthday is {{ birthday | date | uppercase}}</p>
```

为自定义数据转换创建管道，可以使用 @Pipe() 装饰器（ng generate pipe）：

```jsx
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({name: 'exponentialStrength'})
export class ExponentialStrengthPipe implements PipeTransform {
  transform(value: number, exponent?: number): number {
    return Math.pow(value, isNaN(exponent) ? 1 : exponent);
  }
}
```

### Angular 表单

**表单底层构造块**

- `[FormControl](https://v7.angular.cn/api/forms/FormControl)` 实例用于追踪单个表单控件的值和验证状态。
- `[FormGroup](https://v7.angular.cn/api/forms/FormGroup)` 用于追踪一个表单控件组的值和状态。
- `[FormArray](https://v7.angular.cn/api/forms/FormArray)` 用于追踪表单控件数组的值和状态。
- `[ControlValueAccessor](https://v7.angular.cn/api/forms/ControlValueAccessor)` 用于在 Angular 的 `[FormControl](https://v7.angular.cn/api/forms/FormControl)` 实例和原生 DOM 元素之间创建一个桥梁。

**表单模型**

响应式表单建立表单模型（表单模型是数据源），使用 FormControl 来建立单个表单控件的模型：

```jsx
import { Component } from '@angular/core';
import { FormControl } from '@angular/forms';

@Component({
  selector: 'app-reactive-favorite-color',
  template: `
    Favorite Color: <input type="text" [formControl]="favoriteColorControl">
  `
})
export class FavoriteColorComponent {
  favoriteColorControl = new FormControl('');
}
```

模板驱动表单（模板是数据源），使用 ngModel 与示例的数据进行联动：

```jsx
import { Component } from '@angular/core';

@Component({
  selector: 'app-template-favorite-color',
  template: `
    Favorite Color: <input type="text" [(ngModel)]="favoriteColor">
  `
})
export class FavoriteColorComponent {
  favoriteColor = '';
}
```

**更新表单的值：**

- setValue()：单个控件设置新值
- patchValue()：用传入对象中的属性进行对应值的替换
    
    ```jsx
    updateProfile() {
      this.profile.patchValue({
        name: 'Keefe'
      });
    }
    // 只有 name 替换了，age 还是原来的值
    ```
    

**FormBuilder**

```jsx
import { FormBuilder } from '@angular/forms';

export class ProfileEditorComponent {
  profileForm = this.fb.group({
    firstName: [''],
    lastName: [''],
    address: this.fb.group({
      street: [''],
      city: [''],
      state: [''],
      zip: ['']
    }),
  });
 
  constructor(private fb: FormBuilder) { }
}
```

响应式表单：使用 FormArray 可以动态添加表单控件

**表单验证**

模板驱动表单的验证，Angular 会对 HTML 自带的一些属性进行同名函数验证，这里的appForbiddenName 是一个验证器的指令

```jsx
<input 
	id="name" 
	name="name" 
	class="form-control"
  required minlength="4" 
	appForbiddenName="bob"
  [(ngModel)]="hero.name" 
	#name="ngModel" >
// 这里的 #name="ngModel" 是将 ngModel 导出成 name 变量，方便下面检测其 dirty 和touched 属性（判断是否填写过）

<div 
	*ngIf="name.invalid && (name.dirty || name.touched)"
  class="alert alert-danger">
  <div *ngIf="name.errors.required">
    Name is required.
  </div>
  <div *ngIf="name.errors.minlength">
    Name must be at least 4 characters long.
  </div>
  <div *ngIf="name.errors.forbiddenName">
    Name cannot be Bob.
  </div>
</div>
```

响应式表单验证，FormControl的第一个参数是表单绑定的数据变量，第二个是验证条件（可以是数组，也可以是单个验证函数）

```jsx
this.heroForm = new FormGroup({
    'name': new FormControl(this.hero.name, [
       Validators.required,
       Validators.minLength(4),
       forbiddenNameValidator(/bob/i) 
    ]),
    'alterEgo': new FormControl(this.hero.alterEgo),
    'power': new FormControl(this.hero.power, Validators.required)
});
```

**自定义验证器**

验证器是一个工厂，返回一个验证器函数

**控件状态 CSS 类**

- `.ng-valid`
- `.ng-invalid`
- `.ng-pending`
- `.ng-pristine`
- `.ng-dirty`
- `.ng-untouched`
- `.ng-touched`