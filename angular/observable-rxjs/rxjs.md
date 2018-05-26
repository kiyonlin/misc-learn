# RxJS 库

`RxJS`（响应式扩展的 `JavaScript` 版）是一个使用可观察对象进行响应式编程的库，它让组合异步代码和基于回调的代码变得更简单 ([RxJS Docs](https://rxjs-dev.firebaseapp.com/))。

RxJS 提供了一种对 Observable 类型的实现，直到 Observable 成为了 JavaScript 语言的一部分并且浏览器支持它之前，它都是必要的。这个库还提供了一些工具函数，用于创建和使用可观察对象。这些工具函数可用于：

- 把现有的异步代码转换成可观察对象
- 迭代流中的各个值
- 把这些值映射成其它类型
- 对流进行过滤
- 组合多个流

## 创建可观察对象的函数

`RxJS` 提供了一些用来创建可观察对象的函数。这些函数可以简化根据某些东西创建可观察对象的过程，比如事件、定时器、承诺等等。比如：

从promise创建

```ts
import { fromPromise } from 'rxjs';

// Create an Observable out of a promise
const data = fromPromise(fetch('/api/endpoint'));
// Subscribe to begin listening for async result
data.subscribe({
 next(response) { console.log(response); },
 error(err) { console.error('Error: ' + err); },
 complete() { console.log('Completed'); }
});
```

从定时器创建

```ts
// Create an observable from a counter

import { interval } from 'rxjs';

// Create an Observable that will publish a value on an interval
const secondsCounter = interval(1000);
// Subscribe to begin publishing values
secondsCounter.subscribe(n =>
  console.log(`It's been ${n} seconds since subscribing!`));
```

从事件创建

```ts
import { fromEvent } from 'rxjs';
 
const el = document.getElementById('my-element');
 
// Create an Observable that will publish mouse movements
const mouseMoves = fromEvent(el, 'mousemove');
 
// Subscribe to start listening for mouse-move events
const subscription = mouseMoves.subscribe((evt: MouseEvent) => {
  // Log coords of mouse movements
  console.log(`Coords: ${evt.clientX} X ${evt.clientY}`);
 
  // When the mouse is over the upper-left of the screen,
  // unsubscribe to stop listening for mouse movements
  if (evt.clientX < 40 && evt.clientY < 40) {
    subscription.unsubscribe();
  }
});
```

从 `Ajax` 创建

```ts
import { ajax } from 'rxjs/ajax';

// Create an Observable that will create an AJAX request
const apiData = ajax('/api/data');
// Subscribe to create the request
apiData.subscribe(res => console.log(res.status, res.response));
```

## 操作符

操作符是基于可观察对象构建的一些对集合进行复杂操作的函数。`RxJS` 定义了一些操作符，比如 `map()`、`filter()`、`concat()` 和 `flatMap()`。

```ts
import { map } from 'rxjs/operators';
 
const nums = of(1, 2, 3);
 
const squareValues = map((val: number) => val * val);
const squaredNums = squareValues(nums);
 
squaredNums.subscribe(x => console.log(x));
 
// Logs
// 1
// 4
// 9
```

可以使用管道来把这些操作符链接起来。管道让你可以把多个由操作符返回的函数组合成一个。`pipe()` 函数以你要组合的这些函数作为参数，并且返回一个新的函数，当执行这个新函数时，就会顺序执行那些被组合进去的函数。

```ts
import { filter, map } from 'rxjs/operators';

const squareOdd = of(1, 2, 3, 4, 5)
  .pipe(
    filter(n => n % 2 !== 0),
    map(n => n * n)
  );

// Subscribe to get values
squareOdd.subscribe(x => console.log(x));
```

## 错误处理

除了可以在订阅时提供 `error()` 处理器外，`RxJS` 还提供了 `catchError` 操作符，它允许你在管道中处理已知错误。

假设有一个可观察对象，它发起 `API` 请求，然后对服务器返回的响应进行映射。如果服务器返回了错误或值不存在，就会生成一个错误。如果捕获这个错误并提供了一个默认值，流就会继续处理这些值，而不会报错。

下面是使用 `catchError` 操作符实现这种效果的例子：

```ts
import { ajax } from 'rxjs/ajax';
import { map, catchError } from 'rxjs/operators';
// Return "response" from the API. If an error happens,
// return an empty array.
const apiData = ajax('/api/data').pipe(
  map(res => {
    if (!res.response) {
      throw new Error('Value expected!');
    }
    return res.response;
  }),
  catchError(err => of([]))
);
 
apiData.subscribe({
  next(x) { console.log('data: ', x); },
  error(err) { console.log('errors already caught... will not run'); }
});
```

## 重试失败的可观察对象

`catchError` 提供了一种简单的方式进行恢复，而 `retry` 操作符让你可以尝试失败的请求。

可以在 `catchError` 之前使用 `retry` 操作符。它会订阅到原始的来源可观察对象，它可以重新运行导致结果出错的动作序列。如果其中包含 `HTTP` 请求，它就会重新发起那个 `HTTP` 请求。

下列代码为前面的例子加上了捕获错误前重发请求的逻辑：

```ts
import { ajax } from 'rxjs/ajax';
import { map, retry, catchError } from 'rxjs/operators';
 
const apiData = ajax('/api/data').pipe(
  retry(3), // Retry up to 3 times before failing
  map(res => {
    if (!res.response) {
      throw new Error('Value expected!');
    }
    return res.response;
  }),
  catchError(err => of([]))
);
 
apiData.subscribe({
  next(x) { console.log('data: ', x); },
  error(err) { console.log('errors already caught... will not run'); }
});
```

## 可观察对象的命名约定

 `Angular` 的应用几乎都是用 `TypeScript` 写的，可观察对象的名字以`“$”`符号结尾。如果希望用某个属性来存储来自可观察对象的最近一个值，它的命名惯例是与可观察对象同名，但不带`“$”`后缀。

```ts
import { Component } from '@angular/core';
import { Observable } from 'rxjs';
 
@Component({
  selector: 'app-stopwatch',
  templateUrl: './stopwatch.component.html'
})
export class StopwatchComponent {
 
  stopwatchValue: number;
  stopwatchValue$: Observable<number>;
 
  start() {
    this.stopwatchValue$.subscribe(num =>
      this.stopwatchValue = num
    );
  }
}
```
