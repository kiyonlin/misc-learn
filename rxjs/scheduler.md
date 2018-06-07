# Scheduler
`Observable` 有一个优势是可以同时处理同步和非同步行为，但这个优势也带来了一个问题，就是我们常常会搞不清处现在的 `observable` 执行方式是同步的还是非同步的。换句话说，我们很容易搞不清楚 `observable` 到底什么时候开始发送元素！

举例来说，我们可能很清楚 `interval` 是非同步送出元素的，但 `range` 呢？ `from` 呢？他们可能有时候是非同步有时候是同步，这就会变得有点困扰，尤其在除错时执行顺序就非常重要。

而 `Scheduler` 基本上就是拿来处理这个问题的！

## 什么是 Scheduler？
`Scheduler` 控制一个 `observable` 的订阅什么时候开始，以及发送元素什么时候送达，主要由以下三个元素所组成

- `Scheduler` 是一个数据结构。它知道如何根据优先级或其他标准来储存并存储任务。
- `Scheduler` 是一个执行环境。它意味着任务何时何地被执行，比如像是 立即执行、在回调(`callback`)中执行、`setTimeout` 中执行、`animation frame` 中执行
- `Scheduler` 是一个虚拟时钟。它透过 now() 这个方法提供了时间的概念，我们可以让任务在特定的时间点被执行。

简言之 `Scheduler` 会影响 `Observable` 开始执行及元素送达的时机，比如下面这个例子
```ts
var observable = Rx.Observable.create(function (observer) {
    observer.next(1);
    observer.next(2);
    observer.next(3);
    observer.complete();
});

console.log('before subscribe');
observable.observeOn(Rx.Scheduler.async) // 設為 async
.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
console.log('after subscribe');

// "before subscribe"
// "after subscribe"
// 1
// 2
// 3
// "complete"
```

## 有哪些 Scheduler 可以用
目前 `RxJS 5 Scheduler` 跟 `RxJS 4.x` 以前的版本完全不同，在 `RxJS 5` 当中有提供四个 `scheduler`，预设为 `undefined` 会直接以递归的方式执行

- queue
- asap
- async
- animationFrame

## 使用 Scheduler
其实我们在使用各种不同的 `operator` 时，这些 `operator` 就会各自预设不同的 `scheduler`，例如一个无限的 `observable` 就会预设为 `queue scheduler`，而 `timer` 相关的 `operator` 则预设为 `async scheduler`。

要使用 `Scheduler` 除了前面用到的 `observeOn()` 方法外，以下这几个 `creation operators` 最后一个参数都能接收 `Scheduler`

- bindCallback
- bindNodeCallback
- combineLatest
- concat
- empty
- from
- fromPromise
- interval
- merge
- of
- range
- throw
- timer

```ts
var observable = Rx.Observable.from([1,2,3,4,5], Rx.Scheduler.async);
```

### queue
`queue` 的运作方式跟预设的立即执行很像，但是当我们使用到迭代的方法时，他会存储这些行为而非直接执行，一个迭代的 `operator` 就是他会执行另一个 `operator``。queue` 很适合用在会有迭代的 `operator` 且具有大量资料时使用，在这个情况下 `queue` 能避免不必要的效能损耗。

### asap
`asap` 的行为很好理解，它是非同步的执行，在浏览器其实就是 `setTimeout` 设为 `0` 秒 (在 `NodeJS` 中是用 `process.nextTick`)，因为行为很好理解这里就不写例子了。`asap` 因为都是在 `setTimeout` 中执行，所以不会有 `block event loop` 的问题，很适合用在永远不会退订的 `observable`，例如在背景下持续监听 `server` 送来的通知。

### async
这个是在 `RxJS 5` 中新出现的 `Scheduler`，它跟 `asap` 很像但是使用 `setInterval` 来运作，通常是跟时间相关的 `operator` 才会用到。

### animationFrame
这个相信大家应该都知道，他是利用 `Window.requestAnimationFrame` 这个 `API` 去实现的，所以执行周期就跟 `Window.requestAnimationFrame` 一模一样。在做复杂运算，且高频率触发的 `UI` 动画时，就很适合使用 `animationFrame`，以可以搭配 `throttle operator` 使用。