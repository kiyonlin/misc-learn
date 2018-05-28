# 依赖注入

依赖注入（`DI`）是用来创建对象及其依赖的其它对象的一种方式。 当依赖注入系统创建某个对象实例时，会负责提供该对象所依赖的对象（称为该对象的依赖）。

## 依赖注入模式

注入器自动给一个对象注入依赖。

## 注入器

在把 `Angular` 中的服务类（比如 `HeroService` ）注册进依赖注入器（`injector`）之前，它只是个普通类而已。

`Angular` 的依赖注入器负责创建服务的实例，并把它们注入到像 `HeroListComponent` 这样的类中。

当 `Angular` 运行本应用时，首先会在引导过程中创建一个根注入器。所以不需要自己创建 `Angular` 的依赖注入器。

`Angular` 本身没法自动判断你是打算自行创建服务类的实例，还是等注入器来创建它。你必须通过为每个服务指定服务提供商来配置它。

提供商会告诉注入器如何创建该服务。 如果没有提供商，注入器既不知道它该负责创建该服务，也不知道如何创建该服务。

### @Injectable 的 providers 数组
`@Injectable` 装饰器会指出这些服务或其它类是用来注入的。它还能用于为这些服务提供配置项。

这里我们使用类上的 `@Injectable` 装饰器来为 `HeroService` 配置了一个提供商。

```ts
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root',
})
export class HeroService {
  constructor() { }
}
```

`providedIn` 告诉 `Angular`，它的根注入器要负责调用 `HeroService` 类的构造函数来创建一个实例，并让它在整个应用中都是可用的。

有可能用户希望显式选择要使用的服务，或者应该在一个惰性加载的环境下提供该服务。这种情况下，服务提供商应该关联到一个特定的 `@NgModule` 类，而且应该用于该模块包含的任何一个注入器中。

下面这段代码中，`@Injectable` 装饰器用来配置一个服务提供商，它可以用在任何包含了 `HeroModule` 的注入器中。

```ts
import { Injectable } from '@angular/core';
import { HeroModule } from './hero.module';
import { HEROES }     from './mock-heroes';

@Injectable({
  // we declare that this service should be created
  // by any injector that includes HeroModule.

  providedIn: HeroModule,
})
export class HeroService {
  getHeroes() { return HEROES; }
}
```

### @NgModule 中的 providers

在下面的代码片段中，根模块 `AppModule` 在自己的 `providers` 数组中注册了两个提供商。

```ts
providers: [
  UserService,
  { provide: APP_CONFIG, useValue: HERO_DI_CONFIG }
],
```

第一条使用 `UserService` 这个注入令牌（`injection token`）注册了 `UserService` 类（代码中未显示）。 第二条使用 `APP_CONFIG` 这个注入令牌注册了一个值（`HERO_DI_CONFIG`）。

### 在组件中注册提供商

```ts
import { Component } from '@angular/core';
import { HeroService } from './hero.service';

@Component({
  selector: 'app-heroes',
  providers: [ HeroService ],
  template: `
    <h2>Heroes</h2>
    <app-hero-list></app-hero-list>
  `
})
export class HeroesComponent { }
```

### @Injectable、@NgModule 还是 @Component ？

该使用 `@Injectable` 装饰器、`@NgModule`还是 `@Component` 来提供服务呢？ 这几个选择的差别在于最终的打包体积、服务的范围和服务的生命周期。

当在服务本身的 `@Injectable` 装饰器中注册提供商时，优化工具（比如 CLI 产品模式构建时所用的）可以执行摇树优化，这会移除所有没在应用中使用过的服务。摇树优化会导致更小的打包体积。

`Angular` 模块中的 `providers`（`@NgModule.providers`）是注册在应用的根注入器下的。 因此，`Angular` 可以往它所创建的任何类中注入相应的服务。 一旦创建，服务的实例就会存在于该应用的全部生存期中，`Angular` 会把这一个服务实例注入到需求它的每个类中。

你可能想要把这个 `UserService` 注入到应用中的很多地方，并期望每次注入的都是同一个服务实例。 这时候如果不能用 `@Injectable`，那么就可以在 `Angular` 的模块中提供 `UserService`。

> 严格来说，`Angular` 模块中的服务提供商会注册到根注入器上，但是，惰性加载的模块是例外。 在这个例子中，所有模块都是在应用启动时立即加载的，因此模块上的所有服务提供商都注册到了应用的根注入器上。

组件的提供商（`@Component.providers`）会注册到*每个组件实例自己的注入器*上。

因此 `Angular` 只能在*该组件及其各级子组件*的实例上注入这个服务实例，而不能在其它地方注入这个服务实例。

注意，由组件提供的服务，也同样具有有限的生命周期。*组件的每个实例都会有它自己的服务实例*，并且，当组件实例被销毁的时候，服务的实例也同样会被销毁。

在这个范例应用中，`HeroComponent` 会在应用启动时创建，并且它从未销毁，因此，由 `HeroComponent` 创建的 `HeroService` 也同样会活在应用的整个生命周期中。

如果你要把 `HeroService` 的访问权限定在 `HeroesComponent` 及其嵌套的 `HeroListComponent` 中，那么在 `HeroesComponent` 中提供这个 `HeroService` 就是一个好选择。

## 服务提供商们

服务提供商提供依赖值的一个具体的、运行时的版本。 注入器依靠提供商来创建服务的实例，注入器再将服务的实例注入组件、管道或其它服务。必须为注入器注册一个服务的提供商，否则它就不知道该如何创建该服务。

### 把类作为它自己的提供商

有很多方式可以提供一些实现 `Logger` 类的东西。 `Logger` 类本身是一个显而易见而且自然而然的提供商。
```ts
providers: [Logger]
```

### provide 对象字面量

```ts
[{ provide: Logger, useClass: Logger }]
```

`provide` 属性保存的是*令牌* (`token`)，它作为键值 (`key`) 使用，用于定位依赖值和注册提供商。
第二个是一个提供商定义对象。 可以把它看做是指导如何创建依赖值的*配方*。

### 备选的类提供商

某些时候，你会请求一个不同的类来提供服务。 下列代码告诉注入器，当有人请求 `Logger` 时，返回 `BetterLogger`。
```ts
[{ provide: Logger, useClass: BetterLogger }]
```

### 带依赖的类提供商
假设 `EvenBetterLogger` 可以在日志消息中显示用户名。 这个日志服务从注入的 `UserService` 中取得用户， `UserService` 通常也会在应用级注入。

```ts
@Injectable()
export class EvenBetterLogger extends Logger {
  constructor(private userService: UserService) { super(); }

  log(message: string) {
    let name = this.userService.user.name;
    super.log(`Message to ${name}: ${message}`);
  }
}
```

就像之前在 `BetterLogger` 中那样配置它。
```ts
[ UserService,
  { provide: Logger, useClass: EvenBetterLogger }]
```

### 别名类提供商

如果尝试通过 `useClass` 来把 `OldLogger` 作为 `NewLogger` 的别名，就会导致应用中有两个不同的 `NewLogger` 实例。

```ts
[ NewLogger,
  // Not aliased! Creates two instances of `NewLogger`
  { provide: OldLogger, useClass: NewLogger}]
```

解决方案：使用 useExisting 选项指定别名。

```ts
[ NewLogger,
  // Alias OldLogger w/ reference to NewLogger
  { provide: OldLogger, useExisting: NewLogger}]
```

## 依赖注入令牌
当向注入器注册提供商时，实际上是把这个提供商和一个 `DI` 令牌关联起来了。 注入器维护一个内部的令牌-提供商映射表，这个映射表会在请求依赖时被引用到。 令牌就是这个映射表中的键值。在前面的所有例子中，依赖值都是一个类实例，并且类的类型作为它自己的查找键值。这是一个特殊的规约，因为大多数依赖值都是以类的形式提供的。

### 非类依赖

配置对象不总是类的实例，它们可能是对象，如下面这个：
```ts
export const HERO_DI_CONFIG: AppConfig = {
  apiEndpoint: 'api.heroes.com',
  title: 'Dependency Injection'
};
```

### InjectionToken 值

```ts
import { InjectionToken } from '@angular/core';
export const TOKEN = new InjectionToken('desc');
```

可以在创建 `InjectionToken` 时直接配置一个提供商。该提供商的配置会决定由哪个注入器来提供这个令牌，以及如何创建它的值。 这和 `@Injectable` 的用法很像，不过没法用 `InjectionToken` 来定义标准提供商（比如 `useClass` 或 `useFactory`），而要指定一个工厂函数，该函数直接返回想要提供的值。

```ts
export const TOKEN = 
  new InjectionToken('desc', { providedIn: 'root', factory: () => new AppConfig(), })
```

现在，在 `@Inject` 装饰器的帮助下，这个配置对象可以注入到任何需要它的构造函数中:
```ts
constructor(@Inject(TOKEN));
```

## 可选依赖
可以把构造函数的参数标记为 `null` 来告诉 `Angular` 该依赖是可选的：
```ts
constructor(@Inject(Token, null));
```

如果要使用可选依赖，代码就必须准备好处理空值。

## 多级依赖注入器
`Angular` 有一个多级依赖注入系统。 实际上，应用程序中有一个与组件树平行的注入器树。每个组件实例都有自己的注入器！

### 注入器冒泡
当一个组件申请获得一个依赖时，`Angular` 先尝试用该组件自己的注入器来满足它。 如果该组件的注入器没有找到对应的提供商，它就把这个申请转给它父组件的注入器来处理。 如果那个注入器也无法满足这个申请，它就继续转给它的父组件的注入器。 这个申请继续往上冒泡 —— 直到找到了一个能处理此申请的注入器或者超出了组件树中的祖先位置为止。 如果超出了组件树中的祖先还未找到，`Angular` 就会抛出一个错误。

> 还可以“盖住”这次冒泡。一个中层的组件可以声称自己是“宿主”组件。 向上查找提供商的过程会截止于这个“宿主”组件。 