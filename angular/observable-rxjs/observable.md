# 可观察对象（Observable）

## 基本用法和词汇

`Observable` 的实例，定义了一个订阅者（`subscriber`）函数。`subscribe()` 调用会返回一个 `Subscription` 对象，该对象具有一个 `unsubscribe()` 方法。

``` ts
// Create an Observable that will start listening to geolocation updates
// when a consumer subscribes.
const locations = new Observable((observer) => {
  // Get the next and error callbacks. These will be passed in when
  // the consumer subscribes.
  const {next, error} = observer;
  let watchId;
 
  // Simple geolocation API check provides values to publish
  if ('geolocation' in navigator) {
    watchId = navigator.geolocation.watchPosition(next, error);
  } else {
    error('Geolocation not available');
  }
 
  // When the consumer unsubscribes, clean up data ready for next subscription.
  return {unsubscribe() { navigator.geolocation.clearWatch(watchId); }};
});
 
// Call subscribe() to start listening for updates.
const locationsSubscription = locations.subscribe({
  next(position) { console.log('Current Position: ', position); },
  error(msg) { console.log('Error Getting Location: ', msg); }
});
 
// Stop listening for location after 10 seconds
setTimeout(() => { locationsSubscription.unsubscribe(); }, 10000);
```

## 定义观察者

用于接收可观察对象通知的处理器要实现 Observer 接口。这个对象定义了一些回调函数来处理可观察对象可能会发来的三种通知：

|通知类型|	说明|
|-------|-----|
|next	|必要。用来处理每个送达值。在开始执行后可能执行零次或多次。|
|error	|可选。用来处理错误通知。错误会中断这个可观察对象实例的执行过程。|
|complete	|可选。用来处理执行完毕（complete）通知。当执行完毕后，这些值就会继续传给下一个处理器。|

## 订阅

只有当有人订阅 `Observable` 的实例时，它才会开始发布值。 订阅时要先调用该实例的 `subscribe()` 方法，并把一个*观察者对象*传给它，用来接收通知。

- Observable.of(...items) —— 返回一个 `Observable` 实例，它用同步的方式把参数中提供的这些值发送出来。
- Observable.from(iterable) —— 把它的参数转换成一个 `Observable` 实例。 该方法通常用于把一个数组转换成一个（发送多个值的）可观察对象。

## 创建可观察对象
使用 `Observable` 构造函数可以创建任何类型的可观察流。 当执行可观察对象的 `subscribe()` 方法时，这个构造函数就会把它接收到的参数作为订阅函数来运行。 订阅函数会接收一个 `Observer` 对象，并把值发布给观察者的 `next()` 方法。

```ts
// 要创建一个与前面的 Observable.of(1, 2, 3) 等价的可观察对象
// This function runs when subscribe() is called
function sequenceSubscriber(observer) {
  // synchronously deliver 1, 2, and 3, then complete
  observer.next(1);
  observer.next(2);
  observer.next(3);
  observer.complete();
 
  // unsubscribe function doesn't need to do anything in this
  // because values are delivered synchronously
  return {unsubscribe() {}};
}
 
// Create a new Observable that will deliver the above sequence
const sequence = new Observable(sequenceSubscriber);
 
// execute the Observable and print the result of each notification
sequence.subscribe({
  next(num) { console.log(num); },
  complete() { console.log('Finished sequence'); }
});
 
// Logs:
// 1
// 2
// 3
// Finished sequence
```


## 多播

多播用来让可观察对象在一次执行中同时广播给多个订阅者。借助支持多播的可观察对象，你不必注册多个监听器，而是复用第一个`（next）`监听器，并且把值发送给各个订阅者。
```ts
function multicastSequenceSubscriber() {
  const seq = [1, 2, 3];
  // Keep track of each observer (one for every active subscription)
  const observers = [];
  // Still a single timeoutId because there will only ever be one
  // set of values being generated, multicasted to each subscriber
  let timeoutId;
 
  // Return the subscriber function (runs when subscribe()
  // function is invoked)
  return (observer) => {
    observers.push(observer);
    // When this is the first subscription, start the sequence
    if (observers.length === 1) {
      timeoutId = doSequence({
        next(val) {
          // Iterate through observers and notify all subscriptions
          observers.forEach(obs => obs.next(val));
        },
        complete() {
          // Notify all complete callbacks
          observers.forEach(obs => obs.complete());
        }
      }, seq, 0);
    }
 
    return {
      unsubscribe() {
        // Remove from the observers array so it's no longer notified
        observers.splice(observers.indexOf(observer), 1);
        // If there's no more listeners, do cleanup
        if (observers.length === 0) {
          clearTimeout(timeoutId);
        }
      }
    };
  };
}
 
// Run through an array of numbers, emitting one value
// per second until it gets to the end of the array.
function doSequence(observer, arr, idx) {
  return setTimeout(() => {
    observer.next(arr[idx]);
    if (idx === arr.length - 1) {
      observer.complete();
    } else {
      doSequence(observer, arr, idx++);
    }
  }, 1000);
}
 
// Create a new Observable that will deliver the above sequence
const multicastSequence = new Observable(multicastSequenceSubscriber);
 
// Subscribe starts the clock, and begins to emit after 1 second
multicastSequence.subscribe({
  next(num) { console.log('1st subscribe: ' + num); },
  complete() { console.log('1st sequence finished.'); }
});
 
// After 1 1/2 seconds, subscribe again (should "miss" the first value).
setTimeout(() => {
  multicastSequence.subscribe({
    next(num) { console.log('2nd subscribe: ' + num); },
    complete() { console.log('2nd sequence finished.'); }
  });
}, 1500);
 
// Logs:
// (at 1 second): 1st subscribe: 1
// (at 2 seconds): 1st subscribe: 2
// (at 2 seconds): 2nd subscribe: 2
// (at 3 seconds): 1st subscribe: 3
// (at 3 seconds): 1st sequence finished
// (at 3 seconds): 2nd subscribe: 3
// (at 3 seconds): 2nd sequence finished
```

## `Angular` 中的可观察对象

`Angular` 使用可观察对象作为处理各种常用异步操作的接口。比如：

- `EventEmitter` 类派生自 `Observable。`
- `HTTP` 模块使用可观察对象来处理 `AJAX` 请求和响应。
- 路由器和表单模块使用可观察对象来监听对用户输入事件的响应。

### 事件发送器 `EventEmitter`

`Angular` 提供了一个 `EventEmitter` 类，它用来从组件的` @Output()` 属性中发布一些值。`EventEmitter` 扩展了 `Observable`，并添加了一个 `emit()` 方法，这样它就可以发送任意值了。当你调用 `emit()` 时，就会把所发送的值传给订阅上来的观察者的 `next()` 方法。

```ts
@Component({
  selector: 'zippy',
  template: `
  <div class="zippy">
    <div (click)="toggle()">Toggle</div>
    <div [hidden]="!visible">
      <ng-content></ng-content>
    </div>
  </div>`})
 
export class ZippyComponent {
  visible = true;
  @Output() open = new EventEmitter<any>();
  @Output() close = new EventEmitter<any>();
 
  toggle() {
    this.visible = !this.visible;
    if (this.visible) {
      this.open.emit(null);
    } else {
      this.close.emit(null);
    }
  }
}
```

### HTTP

`Angular` 的 `HttpClient` 从 `HTTP` 方法调用中返回了可观察对象。例如，`http.get(‘/api’)` 就会返回可观察对象。相对于基于承诺（`Promise`）的 `HTTP API`，它有一系列优点：

- 可观察对象不会修改服务器的响应（和在承诺上串联起来的 `.then()` 调用一样）。反之，你可以使用一系列操作符来按需转换这些值。
- `HTTP` 请求是可以通过 `unsubscribe()` 方法来取消的。
- 请求可以进行配置，以获取进度事件的变化。
- 失败的请求很容易重试。

### `Async` 管道

`AsyncPipe` 会订阅一个可观察对象或承诺，并返回其发出的最后一个值。当发出新值时，该管道就会把这个组件标记为需要进行变更检查的（译注：因此可能导致刷新界面）。

```ts
@Component({
  selector: 'async-observable-pipe',
  template: `<div><code>observable|async</code>:
       Time: {{ time | async }}</div>`
})
export class AsyncObservablePipeComponent {
  time = new Observable(observer =>
    setInterval(() => observer.next(new Date().toString()), 1000)
  );
}
```

### 路由器 (`router`)

`Router.events` 以可观察对象的形式提供了其事件。 可以使用 `RxJS` 中的 `filter()` 操作符来找到感兴趣的事件，并且订阅它们，以便根据浏览过程中产生的事件序列作出决定。

```ts
import { Router, NavigationStart } from '@angular/router';
import { filter } from 'rxjs/operators';
 
@Component({
  selector: 'app-routable',
  templateUrl: './routable.component.html',
  styleUrls: ['./routable.component.css']
})
export class Routable1Component implements OnInit {
 
  navStart: Observable<NavigationStart>;
 
  constructor(private router: Router) {
    // Create a new Observable the publishes only the NavigationStart event
    this.navStart = router.events.pipe(
      filter(evt => evt instanceof NavigationStart)
    ) as Observable<NavigationStart>;
  }
 
  ngOnInit() {
    this.navStart.subscribe(evt => console.log('Navigation Started!'));
  }
}
```

`ActivatedRoute` 是一个可注入的路由器服务，它使用可观察对象来获取关于路由路径和路由参数的信息。比如，`ActivateRoute.url` 包含一个用于汇报路由路径的可观察对象。

```ts
import { ActivatedRoute } from '@angular/router';
 
@Component({
  selector: 'app-routable',
  templateUrl: './routable.component.html',
  styleUrls: ['./routable.component.css']
})
export class Routable2Component implements OnInit {
  constructor(private activatedRoute: ActivatedRoute) {}
 
  ngOnInit() {
    this.activatedRoute.url
      .subscribe(url => console.log('The URL changed to: ' + url));
  }
}
```

### 响应式表单 (`reactive forms`)

响应式表单具有一些属性，它们使用可观察对象来监听表单控件的值。 `FormControl` 的 `valueChanges` 属性和 `statusChanges` 属性包含了会发出变更事件的可观察对象。订阅可观察的表单控件属性是在组件类中触发应用逻辑的途径之一。

```ts
import { FormGroup } from '@angular/forms';
 
@Component({
  selector: 'my-component',
  template: 'MyComponent Template'
})
export class MyComponent implements OnInit {
  nameChangeLog: string[] = [];
  heroForm: FormGroup;
 
  ngOnInit() {
    this.logNameChange();
  }
  logNameChange() {
    const nameControl = this.heroForm.get('name');
    nameControl.valueChanges.forEach(
      (value: string) => this.nameChangeLog.push(value)
    );
  }
}
```

## 可观察对象用法实战

### 输入提示（`type-ahead`）建议
可观察对象可以简化输入提示建议的实现方式。典型的输入提示要完成一系列独立的任务：

- 从输入中监听数据。
- 移除输入值前后的空白字符，并确认它达到了最小长度。
- 防抖（这样才能防止连续按键时每次按键都发起 `API` 请求，而应该等到按键出现停顿时才发起）
- 如果输入值没有变化，则不要发起请求（比如按某个字符，然后快速按退格）。
- 如果已发出的 `AJAX` 请求的结果会因为后续的修改而变得无效，那就取消它。

```ts
import { fromEvent } from 'rxjs';
import { ajax } from 'rxjs/ajax';
import { map, filter, debounceTime, distinctUntilChanged, switchMap } from 'rxjs/operators';
 
const searchBox = document.getElementById('search-box');
 
const typeahead = fromEvent(searchBox, 'input').pipe(
  map((e: KeyboardEvent) => e.target.value),
  filter(text => text.length > 2),
  debounceTime(10),
  distinctUntilChanged(),
  switchMap(() => ajax('/api/endpoint'))
);
 
typeahead.subscribe(data => {
 // Handle the data from the API
});
```

### 指数化退避

指数化退避是一种失败后重试 `API` 的技巧，它会在每次连续的失败之后让重试时间逐渐变长，超过最大重试次数之后就会彻底放弃。 如果使用承诺和其它跟踪 `AJAX` 调用的方法会非常复杂，而使用可观察对象，这非常简单：

```ts
import { pipe, range, timer, zip } from 'rxjs';
import { ajax } from 'rxjs/ajax';
import { retryWhen, map, mergeMap } from 'rxjs/operators';
 
function backoff(maxTries, ms) {
 return pipe(
   retryWhen(attempts => range(1, maxTries)
     .pipe(
       zip(attempts, (i) => i),
       map(i => i * i),
       mergeMap(i =>  timer(i * ms))
     )
   )
 );
}
 
ajax('/api/endpoint')
  .pipe(backoff(3, 250))
  .subscribe(data => handleData(data));
 
function handleData(data) {
  // ...
}
```