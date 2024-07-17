---
layout: post
title: "理解 Angular 的依赖注入"
date: 2022-04-19 23:00:01
comments: true
categories: 
- Document
tags:
- Angular
- Dependency Injection
- Framework
---

### 概览

依赖注入是一种设计模式，它的主要作用是让类与其依赖相互独立。它能够解决的现实问题是，当一个类 classA 需要依赖另一些类 classB、classC 时，不再需要 classA 中去实例化 classB、classC，即一个类不需要知道它所依赖的其他服务的具体实现，而是直接去使用其依赖。另外还能够便于编写单测、减少冗余代码、使应用便于扩展。

注入的依赖可以是任何类或值。
Angular 框架中有三种依赖注入的类型：

- Constructor Injection：injector 将依赖（服务）在类构建时注入其中
- Setter Injection：injector 将依赖注入到 setter 属性方法中
- Interface Injection： 依赖本身自带 injector，可以将依赖注入任何传入其中的使用者。接受该依赖的使用者必须通过 interface 暴露一个 setter 方法。

<!-- more -->

### Injector

Injector 是能够存储指令来实现 service 的数据结构，能够将 service 实例化，是 DI 系统里的一个中介角色。
Module、directive 和 component 类都拥有自己的 metadata 装饰器，其中 providers: [] 可以声明使用哪些将哪些 service 注册到类中。每一个 injector 都会陪伴某一个类，也就是说整棵应用树会对应一棵 injector tree。

在 service 里，通过 Injectable({providedIn: string}) 表示 injector 在哪里创建 service。providedIn 属性指明该 service 应该用哪个 injector 去注册。这样可以很容易实现 tree-shaking 来减少不必要的 service 实例化。该属性默认是 'root'，也就是说该服务在应用内任何地方均可实例化使用。
注册好的 service 是单例的——只有一个 injector 可以去初始化该 service。

在 @Directive、@Component  和 @Module  元数据中则是通过 providers 来表示。 
Module 和 Component都拥有自己的 injector，注册在其中的 service 是跟随其生命周期存在。

### 实例化 References

在 Angular 中，reference 也是 service 的一种。但是我们从来不会手动将 ElementRef、Renderer2 这些服务写在任何 providers 中，这是因为 angular 会自动将很多 service 注册在 root provider 中。

Injection Token

Angular 的 DI 系统使用 tokens 来标识一个 Provider。有三种 tokens：Type Token、String Token、Injection Token。

```jsx
providers :[{ provide: ProductService, useClass: ProductService }]
```

其中 provide 属性值就是一个 DI Token 标识，userClass 属性值就是一个 Provider，服务的具体实现。

#### Type Token

最常见的是当依赖是一个 service 时，provide 属性值就是这个服务类的 type。

#### String Token

当注入的依赖没有具体的类型时，可以直接使用 string token 进行标识：

```jsx
providers: [{ provide: 'USE_FAKE', useValue: true }]

export class AppComponent {
	products: Product[];
	constructor(@Inject('USE_FAKE') private fake: boolean) {}
}
```

但是 string token 有一个很严重的问题，就是在一个应用中，可能不同的开发者会使用相同的 string token 去注入不同的依赖，甚至可能会和第三方包中的 token 发生重复和冲突。

所以需要另外一种： Injection Token

#### Injection Token

Injection Token 能够创建一个 token，它可以是任意类型

```jsx
import { InjectionToken } from '@angular/core';
export const APIURL = new InjectionToken<string>('');
providers: [{ provide: APIURL, useValue: 'http://SomeEndPoint.com/api' }]

export class AppComponent {
	constructor(@Inject(APIURL) public ApiUrl: String) { }
}
```

可以看到，Injection Token 类似与 String Token，但是能够保证全局唯一。

参考文章：

[https://angular.io/guide/dependency-injection](https://angular.io/guide/dependency-injection)

[https://angular.io/guide/dependency-injection-overview](https://angular.io/guide/dependency-injection-overview)

[https://angular.io/guide/architecture-services](https://angular.io/guide/architecture-services)

[https://www.simplilearn.com/tutorials/angular-tutorial/angular-dependency-injection](https://www.simplilearn.com/tutorials/angular-tutorial/angular-dependency-injection)

[https://www.codesolutionstuff.com/dependency-injection-in-angular-with-example](https://www.codesolutionstuff.com/dependency-injection-in-angular-with-example/)

[https://www.tektutorialshub.com/angular/injection-token-in-angular](https://www.tektutorialshub.com/angular/injection-token-in-angular/)