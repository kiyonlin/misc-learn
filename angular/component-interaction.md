# 组件交互

## 通过输入型绑定(@Input)把数据从父组件传到子组件。

```ts
import { Component, Input } from '@angular/core';

import { Hero } from './hero';

@Component({
  selector: 'app-hero-child',
  template: `
    <h3>{{hero.name}} says:</h3>
    <p>I, {{hero.name}}, am at your service, {{masterName}}.</p>
  `
})
export class HeroChildComponent {
  @Input() hero: Hero;
  @Input('master') masterName: string;
}
```

第二个 `@Input` 为子组件的属性名 `masterName` 指定一个别名 `master`(译者注：不推荐为起别名，请参见风格指南)。

## 通过 `setter` 截听输入属性值的变化
使用一个输入属性的 `setter`，以拦截父组件中值的变化，并采取行动。

子组件 `NameChildComponent` 的输入属性 `name` 上的这个 `setter`，会 `trim` 掉名字里的空格，并把空值替换成默认字符串。

```ts
import { Component, Input } from '@angular/core';
 
@Component({
  selector: 'app-name-child',
  template: '<h3>"{{name}}"</h3>'
})
export class NameChildComponent {
  private _name = '';
 
  @Input()
  set name(name: string) {
    this._name = (name && name.trim()) || '<no name set>';
  }
 
  get name(): string { return this._name; }
}
```

## 通过`ngOnChanges()`来截听输入属性值的变化
使用 `OnChanges` 生命周期钩子接口的 `ngOnChanges()` 方法来监测输入属性值的变化并做出回应。

> 当需要监视多个、交互式输入属性的时候，本方法比用属性的 `setter` 更合适。

```ts
import { Component, Input, OnChanges, SimpleChange } from '@angular/core';
 
@Component({
  selector: 'app-version-child',
  template: `
    <h3>Version {{major}}.{{minor}}</h3>
    <h4>Change log:</h4>
    <ul>
      <li *ngFor="let change of changeLog">{{change}}</li>
    </ul>
  `
})
export class VersionChildComponent implements OnChanges {
  @Input() major: number;
  @Input() minor: number;
  changeLog: string[] = [];
 
  ngOnChanges(changes: {[propKey: string]: SimpleChange}) {
    let log: string[] = [];
    for (let propName in changes) {
      let changedProp = changes[propName];
      let to = JSON.stringify(changedProp.currentValue);
      if (changedProp.isFirstChange()) {
        log.push(`Initial value of ${propName} set to ${to}`);
      } else {
        let from = JSON.stringify(changedProp.previousValue);
        log.push(`${propName} changed from ${from} to ${to}`);
      }
    }
    this.changeLog.push(log.join(', '));
  }
}
```

## 父组件监听子组件的事件

子组件暴露一个 `EventEmitter` 属性，当事件发生时，子组件利用该属性 `emits` 事件。父组件绑定到这个事件属性，并在事件发生时作出回应。

子组件的 `EventEmitter` 属性是一个输出属性，通常带有 `@Output` 装饰器，就像在 `VoterComponent` 中看到的。

```ts
import { Component, EventEmitter, Input, Output } from '@angular/core';
 
@Component({
  selector: 'app-voter',
  template: `
    <h4>{{name}}</h4>
    <button (click)="vote(true)"  [disabled]="voted">Agree</button>
    <button (click)="vote(false)" [disabled]="voted">Disagree</button>
  `
})
export class VoterComponent {
  @Input()  name: string;
  @Output() onVoted = new EventEmitter<boolean>();
  voted = false;
 
  vote(agreed: boolean) {
    this.onVoted.emit(agreed);
    this.voted = true;
  }
}
```

点击按钮会触发 `true` 或 `false` (布尔型有效载荷)的事件。

父组件 `VoteTakerComponent` 绑定了一个事件处理器(`onVoted()`)，用来响应子组件的事件(`$event`)并更新一个计数器。

```ts
import { Component }      from '@angular/core';
 
@Component({
  selector: 'app-vote-taker',
  template: `
    <h2>Should mankind colonize the Universe?</h2>
    <h3>Agree: {{agreed}}, Disagree: {{disagreed}}</h3>
    <app-voter *ngFor="let voter of voters"
      [name]="voter"
      (onVoted)="onVoted($event)">
    </app-voter>
  `
})
export class VoteTakerComponent {
  agreed = 0;
  disagreed = 0;
  voters = ['Mr. IQ', 'Ms. Universe', 'Bombasto'];
 
  onVoted(agreed: boolean) {
    agreed ? this.agreed++ : this.disagreed++;
  }
}
```

## 父组件与子组件通过本地变量互动

```ts
import { Component }                from '@angular/core';
import { CountdownTimerComponent }  from './countdown-timer.component';
 
@Component({
  selector: 'app-countdown-parent-lv',
  template: `
  <h3>Countdown to Liftoff (via local variable)</h3>
  <button (click)="timer.start()">Start</button>
  <button (click)="timer.stop()">Stop</button>
  <div class="seconds">{{timer.seconds}}</div>
  <app-countdown-timer #timer></app-countdown-timer>
  `,
  styleUrls: ['../assets/demo.css']
})
export class CountdownLocalVarParentComponent { }
```

父组件不能通过数据绑定使用子组件的 `start` 和 `stop` 方法，也不能访问子组件的 `seconds` 属性。

把本地变量(`#timer`)放到(`<countdown-timer>`)标签中，用来代表子组件。这样父组件的模板就得到了*子组件的引用*，于是可以*在父组件的模板中访问子组件的所有属性和方法*。

这个例子把父组件的按钮绑定到子组件的 `start` 和 `stop` 方法，并用插值表达式来显示子组件的 `seconds` 属性。

## 父组件调用 `@ViewChild()`

本地变量方法是个简单便利的方法。但是它也有局限性，因为父组件-子组件的连接必须全部在父组件的模板中进行。父组件本身的代码对子组件没有访问权。

当父组件类需要访问子组件时，可以把子组件作为 ViewChild，注入到父组件里面。

```ts
import { AfterViewInit, ViewChild } from '@angular/core';
import { Component }                from '@angular/core';
import { CountdownTimerComponent }  from './countdown-timer.component';
 
@Component({
  selector: 'app-countdown-parent-vc',
  template: `
  <h3>Countdown to Liftoff (via ViewChild)</h3>
  <button (click)="start()">Start</button>
  <button (click)="stop()">Stop</button>
  <div class="seconds">{{ seconds() }}</div>
  <app-countdown-timer></app-countdown-timer>
  `,
  styleUrls: ['../assets/demo.css']
})
export class CountdownViewChildParentComponent implements AfterViewInit {
 
  @ViewChild(CountdownTimerComponent)
  private timerComponent: CountdownTimerComponent;
 
  seconds() { return 0; }
 
  ngAfterViewInit() {
    // Redefine `seconds()` to get from the `CountdownTimerComponent.seconds` ...
    // but wait a tick first to avoid one-time devMode
    // unidirectional-data-flow-violation error
    setTimeout(() => this.seconds = () => this.timerComponent.seconds, 0);
  }
 
  start() { this.timerComponent.start(); }
  stop() { this.timerComponent.stop(); }
}
```

首先，使用 `ViewChild` 装饰器导入这个引用，并挂上 `AfterViewInit` 生命周期钩子。

接着，通过 `@ViewChild` 属性装饰器，将子组件 `CountdownTimerComponent` 注入到私有属性 `timerComponent` 里面。

组件元数据里就不再需要 `#timer` 本地变量了。而是把按钮绑定到父组件自己的 `start` 和 `stop` 方法，使用父组件的 `seconds` 方法的插值表达式来展示秒数变化。

这些方法可以直接访问被注入的计时器组件。

`ngAfterViewInit()` 生命周期钩子是非常重要的一步。被注入的计时器组件只有在 `Angular` 显示了父组件视图之后才能访问，所以它先把秒数显示为 `0`.

然后 `Angular` 会调用 `ngAfterViewInit` 生命周期钩子，但这时候再更新父组件视图的倒计时就已经太晚了。`Angular` 的单向数据流规则会阻止在同一个周期内更新父组件视图。应用在显示秒数之前会被迫再等一轮。

使用 `setTimeout()` 来等下一轮，然后改写 `seconds()` 方法，这样它接下来就会从注入的这个计时器组件里获取秒数的值。