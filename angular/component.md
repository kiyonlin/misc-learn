# 组件
组件控制屏幕上被称为视图的一小片区域。

## 元数据

`@Component` 装饰器会指出紧随其后的那个类是个组件类，并为其指定元数据。 

组件的元数据告诉 `Angular` 到哪里获取它需要的主要构造块，以创建和展示这个组件及其视图。 具体来说，它把一个模板（无论是直接内联在代码中还是引用的外部文件）和该组件关联起来。 该组件及其模板，共同描述了一个视图。

除了包含或指向模板之外，`@Component` 的元数据还会配置要如何在 `HTML` 中引用该组件，以及该组件需要哪些服务等等。

最常用的 `@Component` 配置选项：

- selector：是一个 CSS 选择器，它会告诉 Angular，一旦在模板 HTML 中找到了这个选择器对应的标签，就创建并插入该组件的一个实例。 比如，如果应用的 HTML 中包含 <app-hero-list></app-hero-list>，Angular 就会在这些标签中插入一个 HeroListComponent 实例的视图。
- templateUrl：该组件的 HTML 模板文件相对于这个组件文件的地址。 另外，你还可以用 template 属性的值来提供内联的 HTML 模板。 这个模板定义了该组件的宿主视图。
- providers 是当前组件所需的依赖注入提供商的一个数组。在这个例子中，它告诉 Angular，该组件的构造函数需要一个 HeroService 实例，以获取要显示的英雄列表。

## 数据绑定

![数据绑定](https://angular.cn/generated/images/guide/architecture/databinding.png)

### 双向数据绑定
```html
<input [(ngModel)]="hero.name">
```

![双向数据绑定](https://angular.cn/generated/images/guide/architecture/component-databinding.png)

### 组件通信
![组件通信](https://angular.cn/generated/images/guide/architecture/parent-child-binding.png)

## 指令

指令就是一个带有 `@Directive` 装饰器的类。

### 结构型指令
结构型指令通过添加、移除或替换 DOM 元素来修改布局。 这个范例模板使用了两个内置的结构型指令来为要渲染的视图添加程序逻辑：
```html
<li *ngFor="let hero of heroes"></li>
<app-hero-detail *ngIf="selectedHero"></app-hero-detail>
```

### 属性型指令
属性型指令会修改现有元素的外观或行为。 在模板中，它们看起来就像普通的 HTML 属性一样，因此得名“属性型指令”。

`ngModel` 指令就是属性型指令的一个例子，它实现了双向数据绑定。 `ngModel` 修改现有元素（一般是 `<input>`）的行为：设置其显示属性值，并响应 `change` 事件。

```html
<input [(ngModel)]="hero.name">
```