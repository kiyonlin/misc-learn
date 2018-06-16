# 生命周期钩子

> 按顺序

| 钩子 |用途以及时机|
|-----|----------|
|ngOnChanges()|当 Angular（重新）设置数据绑定输入属性时响应。 该方法接受当前和上一属性值的 SimpleChanges 对象当被绑定的输入属性的值(对象是引用调用，对象里面的值发生变化时，不会监测到)发生变化时调用，首次调用一定会发生在 ngOnInit() 之前。 这对读取绑定值的更改非常有用。|
|ngOnInit()|在 Angular 第一次显示数据绑定和设置指令/组件的输入属性之后，初始化指令/组件。在第一轮 ngOnChanges() 完成之后调用，只调用一次。|
|ngDoCheck()|检测，并在发生 Angular 无法或不愿意自己检测的变化时作出反应。在每个 Angular 变更检测周期中调用，ngOnChanges() 和 ngOnInit() 之后。|
|ngAfterContentInit()|当把内容投影进组件之后调用。第一次 ngDoCheck() 之后调用，只调用一次。当任何内容子项已完全初始化时，此钩子将允许完成设置内容子项组件所需的任何初始工作，例如，如果您需要验证传入的内容是否有效。|
|ngAfterContentChecked()|每次完成被投影组件内容的变更检测之后调用。ngAfterContentInit() 和每次 ngDoCheck() 之后调用|
|ngAfterViewInit()|初始化完组件视图及其子视图之后调用。第一次 ngAfterContentChecked() 之后调用，只调用一次。|
|ngAfterViewChecked()|每次做完组件视图和子视图的变更检测之后调用。ngAfterViewInit() 和每次 ngAfterContentChecked() 之后调用。|
|ngOnDestroy()|当 Angular 每次销毁指令/组件之前调用并清扫。 在这儿反订阅可观察对象和分离事件处理器，停止侦听传入数据或清除计时器，以防内存泄漏。在 Angular 销毁指令/组件之前调用。|