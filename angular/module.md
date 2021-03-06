# 模块
*NgModules* 用于配置注入器和编译器，并帮你把那些相关的东西组织在一起。

*NgModules* 是一个带有 `@NgModule` 装饰器的类。 `@NgModule` 的参数是一个元数据对象，用于描述如何编译组件的模板，以及如何在运行时创建注入器。 它会标出该模块自己的组件、指令和管道，通过 `exports` 属性公开其中的一部分，以便外部组件使用它们。 *NgModules* 还能把一些服务提供商添加到应用的依赖注入器中。

## 元数据
component
- declarations（可声明对象表） —— 那些属于本 NgModule 的组件、指令、管道。
- exports（导出表） —— 公开其中的部分组件、指令和管道，以便其它模块中的组件模板中可以使用它们。
- imports（导入表） —— 导入其它带有组件、指令和管道的模块，这些模块中的元件都是本模块所需的。
- providers —— 提供一些供应用中的其它组件使用的服务。
- bootstrap —— 应用的主视图，称为根组件。它是应用中所有其它视图的宿主。只有根模块才应该设置这个 bootstrap 属性。


## 功能模块

功能模块是用来对代码进行组织的模块。

随着应用的增长，你可能需要组织与特定应用有关的代码。 这将帮你把特性划出清晰的边界。使用功能模块，你可以把与特定的功能或特性有关的代码从其它代码中分离出来。 为应用勾勒出清晰的边界，有助于开发人员之间、小组之间的协作，有助于分离各个指令，并帮助管理根模块的大小。

与核心的 `Angular API` 的概念相反，功能模块是最佳的组织方式。功能模块提供了聚焦于特定应用需求的一组功能，比如用户工作流、路由或表单。 虽然你也可以用根模块做完所有事情，不过功能模块可以帮助你把应用划分成一些聚焦的功能区。功能模块通过它提供的服务以及共享出的组件、指令和管道来与根模块和其它模块合作。

### 功能模块的分类

|功能模块|指导原则|声明|提供商|导出什么|被谁导入|
| ----- | ----- | ----- | ----- | ----- | ----- |
|领域模块|业务模块|有|罕见|顶级组件|功能模块、AppModule|
|路由功能模块|顶层组件会作为路由导航时的目标组件|有|罕见|无|无|
|路由模块|为其它模块提供路由配置|无|是（守卫）|RouterModule|功能模块（供路由使用）|
|服务模块|提供了一些工具服务，完全由服务提供商组成（[HttpClientModule](https://angular.cn/api/common/http/HttpClientModule)）|无|有|无|AppModule|
|组件模块|为外部模块提供组件、指令和管道|有|罕见|有|功能模块|

## 入口组件
从分类上说，入口组件是 `Angular` 命令式加载的任意组件（也就是说没有在模板中引用过它）， 可以在 `NgModule` 中引导它，或把它包含在路由定义中来指定入口组件。

入口组件有两种主要的类型：

- 引导用的根组件。
- 在路由定义中指定的组件。

```ts
const routes: Routes = [
  {
    path: '',
    component: CustomerListComponent
  }
];
```

所有路由组件都必须是入口组件。这需要把同一个组件添加到两个地方（路由中和 `entryComponents` 中），但编译器足够聪明，可以识别出这里是一个路由定义，因此它会自动把这些路由组件添加到 `entryComponents` 中。

## NgModule API

### @NgModule 的设计意图
宏观来讲，`NgModule` 是组织 `Angular` 应用的一种方式，它们通过 `@NgModule` 装饰器中的元数据来实现这一点。 这些元数据可以分成三类：

- 静态的：编译器配置，用于告诉编译器指令的选择器并通过选择器匹配的方式决定要把该指令应用到模板中的什么位置。它是通过 `declarations` 数组来配置的。
- 运行时：通过 `providers` 数组提供给注入器的配置。
- 组合/分组：通过 `imports` 和 `exports` 数组来把多个 `NgModule` 放在一起，并彼此可用。

```ts
@NgModule({
  // Static, that is compiler configuration
  declarations: [], // Configure the selectors
  entryComponents: [], // Generate the host factory

  // Runtime, or injector configuration
  providers: [], // Runtime injector configuration

  // Composability / Grouping
  imports: [], // composing NgModules together
  exports: [] // making NgModules available to other parts of the app
})
```

### [@NgModule 元数据](https://angular.cn/guide/ngmodule-api#codengmodulecode-metadata)
