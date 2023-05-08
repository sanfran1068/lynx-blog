layout: post
title: "Angular 路由"
date: 2021-07-07 22:01:00
comments: true
categories: 
- Document
tags:
- Angular
- 基础知识
---

使用 Angular 路由首先要在 index.html 的 <head> 标签下先添加一个 <base> 元素，来告诉路由器该如何合成导航用的 URL。Angular 路由并不在核心库，而是在其自己的 @angular/router 库中，使用时需要进行引入： `import { RouterModule, Routes } from '@angular/router';` 。

<!-- more -->

下面是一个在根模块的路由实例：

```jsx
const appRoutes: Routes = [
  { path: 'crisis-center', component: CrisisListComponent },
  { path: 'hero/:id',      component: HeroDetailComponent },
  {
    path: 'heroes',
    component: HeroListComponent,
    data: { title: 'Heroes List' }
  },
  { path: '',
    redirectTo: '/heroes',
    pathMatch: 'full'
  },
  { path: '**', component: PageNotFoundComponent }
];

@NgModule({
  imports: [
    RouterModule.forRoot(
      appRoutes,
      { enableTracing: true } // <-- debugging purposes only
    )
    // other imports here
  ],
  ...
})
export class AppModule { }
```

使用 `RouterModule.forRoot()` 方法来配置路由器，并添加到 AppModule 的 imports 数组中。

路由 Routes 类有以下的特点：

- 路由都是一个 path （**path不能以斜杠开头！**）对应一个组件；
- :id 是一个路由参数的令牌(Token)，例如 '/home/323'，就对应 'home/:id' 中的 id；
- 每个路由对象中都可以传入 data 字段来存放关于该路由的信息；
- '' 空字符串的 path 表示应用的默认路径，通常作为起点，redirectTo 可以设置重定向的 path；
- '**' 表示通配符，一般作为默认路径放在最后，一般用作 404 页面或重定向到其他路由（**先匹配者优先原则**）；

### 路由出口

上面提到的是路由的详情配置，还需要将这个路由应用在对应的地方。Angular 的路由出口标签为： `<router-outlet></router-outlet>` 。这个标签就表示路由组件在这个地方进行相应的展示和切换。

### 路由器链接

除了在浏览器地址栏去输入 url 导航到制定的路由页面，一般还需要一些用户操作进行路由导航，可以使用 routerLink 的指令来指明某个链接对应的跳转路由：

```jsx
<a routerLink="/crisis-center" routerLinkActive="active">Crisis Center</a>
```

routerLinkActive 指令会根据当前链接对应的路由是否激活来切换所绑定的 css 类。

### 路由器状态

可以在应用中的任何地方用 Router 服务（注入的 Router 类实例）及其 routerState 属性来访问当前的 RouterState 值。路由的路径和参数可以通过**注入**进来的一个名叫 ActivatedRoute 的路由服务来获取，其中包括以下有用信息：

- url：路由路径的 Observable 对象，是一个由路由路径中的各个部分组成的字符串数组。
- data：一个 Observable，其中包含提供给路由的 data 对象。也包含由解析守卫（resolve guard）解析而来的值。
- paramMap：一个 Observable，其中包含一个由当前路由的必要参数和可选参数组成的 map 对象。用这个 map 可以获取来自同名参数的单一值或多重值（有 has()/get()/getAll()/keys() 等 API）。
- queryParamMap：一个 Observable，其中包含一个对所有路由都有效的查询参数组成的map对象。 用这个 map 可以获取来自查询参数的单一值或多重值。
- fragment：一个适用于所有路由的 URL 的 fragment（片段）的 Observable。
- outlet：要把该路由渲染到的 RouterOutlet 的名字。**对于无名路由，它的路由名是 primary**，而不是空串。
- routeConfig：用于该路由的路由配置信息，其中包含原始路径。
- parent：当该路由是一个子路由时，表示该路由的父级 ActivatedRoute。
- firstChild：包含该路由的子路由列表中的第一个 ActivatedRoute。
- children：包含当前路由下所有已激活的子路由。

### 路由事件

在每次导航中，Router 都会通过 Router.events 属性发布一些导航事件。这些事件的范围涵盖了从开始导航到结束导航之间的很多时间点。

- NavigationStart：本事件会在导航开始时触发。
- RouteConfigLoadStart：本事件会在 Router 惰性加载 某个路由配置之前触发。
- RouteConfigLoadEnd：本事件会在惰性加载了某个路由后触发。
- RoutesRecognized：本事件会在路由器解析完 URL，并识别出了相应的路由时触发
- GuardsCheckStart：本事件会在路由器开始 Guard 阶段之前触发。
- ChildActivationStart：本事件会在路由器开始激活路由的子路由时触发。
- ActivationStar：本事件会在路由器开始激活某个路由时触发。
- GuardsCheckEnd：本事件会在路由器成功完成了 Guard 阶段时触发。
- ResolveStart：本事件会在 Router 开始解析（Resolve）阶段时触发。
- ResolveEnd：本事件会在路由器成功完成了路由的解析（Resolve）阶段时触发。
- ChildActivationEnd：本事件会在路由器激活了路由的子路由时触发。
- ActivationEnd：本事件会在路由器激活了某个路由时触发。
- NavigationEnd：本事件会在导航成功结束之后触发。
- NavigationCancel：本事件会在导航被取消之后触发。 这可能是因为在导航期间某个路由守卫返回了 false。
- NavigationError：这个事件会在导航由于意料之外的错误而失败时触发。
- Scroll：本事件代表一个滚动事件。

### 子路由配置

子路由可以在某个子模块的路由文件中进行实现：

```jsx
const crisisCenterRoutes: Routes = [
  {
    path: 'crisis-center',
    component: CrisisCenterComponent,
    children: [
      {
        path: '',
        component: CrisisListComponent,
        children: [
          {
            path: ':id',
            component: CrisisDetailComponent
          },
          {
            path: '',
            component: CrisisCenterHomeComponent
          }
        ]
      }
    ]
  }
];

@NgModule({
  imports: [
    RouterModule.forChild(crisisCenterRoutes)
  ],
  exports: [
    RouterModule
  ]
})
export class CrisisCenterRoutingModule { }
```

注意：除了在 app.module.ts 根模块中使用 `RouterModule.forRoot()` 来配置路由外，其他的路由文件中都必须使用 `RouterModule.forChild()` 来配置路由。否则多个根路由可能会发生问题。

### Secondary Outlets Primer

一般情况下， `<router-outlet></router-outlet>` 标签会展示和切换所有路由，但是如果要分区域设置多个 Router Outlets，就需要用到第二路由。那么就要使用以下的方法：

```jsx
<router-outlet name="sidebar"></router-outlet>

const ROUTES = [
  { path: 'home', component: HomeComponent },
  { path: 'chat', component: ChatComponent, outlet: 'sidebar' }
]
```

使用了第二路由后，url 可能会出现 `[localhost:4200/home(sidebar:chat)](http://localhost:4200/home(sidebar:chat))` 这种情况。

### 路由守卫

一般情况下，路由是需要有条件地进行跳转（例如某些路由需要登录后的权限）。在路由配置文件的 Routes 类实例中，每个路由对象都可以通过添加对应的守卫来控制路由的访问权限：

- `[CanActivate](https://v7.angular.cn/api/router/CanActivate)`来处理导航***到***某路由的情况。
- `[CanActivateChild](https://v7.angular.cn/api/router/CanActivateChild)`来处理导航***到***某子路由的情况。
- `[CanDeactivate](https://v7.angular.cn/api/router/CanDeactivate)`来处理从当前路由***离开***的情况.
- `[Resolve](https://v7.angular.cn/api/router/Resolve)`在路由激活***之前***获取路由数据。
- `[CanLoad](https://v7.angular.cn/api/router/CanLoad)`来处理***异步***导航到某特性模块的情况。

```jsx
// 配置路由
const adminRoutes: Routes = [
  {
    path: 'admin',
    component: AdminComponent,
    canActivate: [AuthGuard]
  }
];

// 实现 CanActivate 的守卫实例
export class AuthGuard implements CanActivate {
  canActivate(
    next: ActivatedRouteSnapshot,
    state: RouterStateSnapshot): boolean {
    console.log('AuthGuard#canActivate called');
    return true;
  }
}
```

### 异步路由

为了避免路由太多，一次性加载很多组件，可以引入异步路由对组件进行懒加载。异步路由中可以使用 canLoad 来控制是否加载，使用 canActivateChild 来保护子路由权限，loadChildren 设置为module 或 component 的地址，并使用 #moduleName 来添加导出的类名。

```jsx
const routes: Routes = [
  {
    path: 'personinfo',
    canLoad: [AuthGuardService],
    canActivateChild: [AuthGuardService],
    loadChildren: './components/pages/person-info/person-info.module#PersonInfoModule'
	  data: { preLoad: true } // 预加载开启      
	}
]
```