# CDK简介

许多组件都有部分功能是共用的：
![cdk-slide-01](https://wellwind.idv.tw/blog/2018/01/12/angular-material-25-cdk-intro/01-angular-cdk-slide-01.png)

抽象出来的部分就是Angular CDK。
![cdk-slide-02](https://wellwind.idv.tw/blog/2018/01/12/angular-material-25-cdk-intro/02-angular-cdk-slide-02.png)

一些知名的 Angular 组件库：
- [Clarity](http://clarity.design)
- [wijmo](https://www.grapecity.com/en/wijmo)
- [ngBootstrap](https://ng-bootstrap.github.io/)
- [kendo UI](https://www.telerik.com/kendo-angular-ui/)
- [Lightning](https://www.lightningdesignsystem.com/)
- [Prime Faces](https://www.primefaces.org/primeng/)

但是在项目中，不可避免要依照需求设计自己的组件库。

> Every app makes their own components

## 目前拥有的功能

### Common Behaviors
`Common Behaviors` 主要是一些常见的互动需求，通常不会直接影响画面或者组件的呈现，但与它们的行为嘻嘻相关：
- Accessibility: 让组件操作更加容易，也更容易让显示器的功能理解。
- Observables: 主要是为基于 `Web` 平台提供的 `observers` 提供一层包装。
- Layout: 打造响应式网页(`RWD`)必备的一套工具，用来判断目前浏览器配置的变化，以回应不同的呈现需求。
- Overlay: 提供一些方法来在显示器上呈现一个操作画面(`panel`)，是 `dialog` 类型组件的核心。
- Portal: 提供在呈现 `template` 或 `component` 更加弹性的功能；对于需要动态载入的功能非常有用。
- Bidirectionality: 主要用来处理 `RTL` 和 `LTR` 变化。
- Scrolling: 针对滚动条的互动，提供了一些处理方法。

### Components
`Components` 主要是在设计一些常用组件时的辅助 `directive`，替我们的元件直接加上某个功能:
- Table: 方便建立一个 `data table`。
- Stepper: 方便建立一个向导功能。