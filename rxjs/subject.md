# Subject
首先 `Subject` 可以拿去订阅 `Observable(source)` 代表他是一个 `Observer`，同时 `Subject` 又可以被 `Observer(observerA, observerB)` 订阅，代表他是一个 `Observable`。

总结成两句话
- `Subject` 同时是 `Observable` 又是 `Observer`
- `Subject` 会对内部的 `observers` 清单进行组播(`multicast`)

## BehaviorSubject
很多时候我们会希望 `Subject` 能代表当下的状态，而不是单纯的事件发送，也就是说如果今天有一个新的订阅，我们希望 `Subject` 能立即给出最新的值，而不是没有回应，例如下面这个例子
```ts
var subject = new Rx.Subject();

var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

subject.subscribe(observerA);

subject.next(1);
// "A next: 1"
subject.next(2);
// "A next: 2"
subject.next(3);
// "A next: 3"

setTimeout(() => {
    subject.subscribe(observerB); // 3 秒後才訂閱，observerB 不會收到任何值。
},3000)
```

以上面这个例子来说，`observerB` 订阅的之后，是不会有任何元素送给 `observerB` 的，因为在这之后没有执行任何 `subject.next()`，但很多时候我们会希望 `subject` 能够表达当前的状态，在一订阅时就能收到最新的状态是什么，而不是订阅后要等到有变动才能接收到新的状态，以这个例子来说，我们希望 `observerB` 订阅时就能立即收到`3`，希望做到这样的效果就可以用`BehaviorSubject`。

`BehaviorSubject` 跟 `Subject` 最大的不同就是 `BehaviorSubject` 是用来呈现当前的值，而不是单纯的发送事件。 `BehaviorSubject` 会记住最新一次发送的元素，并把该元素当作目前的值，在使用上 `BehaviorSubject` 建构式需要传入一个参数来代表起始的状态，例子如下

```ts
var subject = new Rx.BehaviorSubject(0); // 0 為起始值
var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

subject.subscribe(observerA);
// "A next: 0"
subject.next(1);
// "A next: 1"
subject.next(2);
// "A next: 2"
subject.next(3);
// "A next: 3"

setTimeout(() => {
    subject.subscribe(observerB); 
    // "B next: 3"
},3000)
```

从上面这个例子可以看得出来 `BehaviorSubject` 在建立时就需要给定一个状态，并在之后任何一次订阅，就会先送出最新的状态。其实这种行为就是一种状态的表达而非单纯的事件，就像是年龄跟生日一样，年龄是一种状态而生日就是事件；所以当我们想要用一个 `stream` 来表达年龄时，就应该用 `BehaviorSubject` 。

## ReplaySubject
在某些时候我们会希望 `Subject` 代表事件，但又能在新订阅时重新发送最后的几个元素，这时我们就可以用 `ReplaySubject`，例子如下
```ts
var subject = new Rx.ReplaySubject(2); // 重複發送最後 2 個元素
var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

subject.subscribe(observerA);
subject.next(1);
// "A next: 1"
subject.next(2);
// "A next: 2"
subject.next(3);
// "A next: 3"

setTimeout(() => {
    subject.subscribe(observerB);
    // "B next: 2"
    // "B next: 3"
},3000)
```
`ReplaySubject(1)` 不同于 `BehaviorSubject`，`BehaviorSubject` 在建立时就会有起始值，比如`BehaviorSubject(0)` 起始值就是`0`，`BehaviorSubject` 是代表着状态而 `ReplaySubject` 只是事件的重放而已。

## multicast
`multicast` 可以用来挂载 `subject` 并回传一个可连结(`connectable`)的 `observable`，如下
```ts
var source = Rx.Observable.interval(1000)
             .take(3)
             .multicast(new Rx.Subject());

var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

source.subscribe(observerA); // subject.subscribe(observerA)

source.connect(); // source.subscribe(subject)

setTimeout(() => {
    source.subscribe(observerB); // subject.subscribe(observerB)
}, 1000);
```
必须真的等到 执行 `connect()` 后才会真的用 `subject` 订阅 `source`，并开始送出元素，如果没有执行 `connect()`, `observable` 是不会真正执行的。

另外值得注意的是这里要退订的话，要把 `connect()` 回传的 `subscription` 退订才会真正停止 `observable` 的执行，如下
```ts
var source = Rx.Observable.interval(1000)
             .do(x => console.log('send: ' + x))
             .multicast(new Rx.Subject()); // 無限的 observable 

var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

var subscriptionA = source.subscribe(observerA);

var realSubscription = source.connect();

var subscriptionB;
setTimeout(() => {
    subscriptionB = source.subscribe(observerB);
}, 1000);

setTimeout(() => {
    subscriptionA.unsubscribe();
    subscriptionB.unsubscribe(); 
    // 這裡雖然 A 跟 B 都退訂了，但 source 還會繼續送元素
}, 5000);

setTimeout(() => {
    realSubscription.unsubscribe();
    // 這裡 source 才會真正停止送元素
}, 7000);
```

上面这段的代码，必须等到 `realSubscription.unsubscribe()` 执行完，`source` 才会真的结束。

虽然用了 `multicast` 感觉会让我们处理的对象少一点，但必须搭配 `connect` 一起使用还是让代码有点复杂，通常我们会希望有 `observer` 订阅时，就立即执行并发送元素，而不要再多执行一个方法(`connect`)，这时我们就可以用`refCount`。

## refCount
`refCount` 必须搭配 `multicast` 一起使用，他可以建立一个只要有订阅就会自动 `connect` 的 `observable`，范例如下
```ts
var source = Rx.Observable.interval(1000)
             .do(x => console.log('send: ' + x))
             .multicast(new Rx.Subject())
             .refCount();

var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

var subscriptionA = source.subscribe(observerA);
// 訂閱數 0 => 1

var subscriptionB;
setTimeout(() => {
    subscriptionB = source.subscribe(observerB);
    // 訂閱數 0 => 2
}, 1000);
```

上面这段程式码，当 `source` 一被 `observerA` 订阅时(订阅数从 `0` 变成 `1`)，就会立即执行并发送元素，我们就不需要再额外执行 `connect`。

同样的在退订时只要订阅数变成 `0` 就会自动停止发送。

```ts
var source = Rx.Observable.interval(1000)
             .do(x => console.log('send: ' + x))
             .multicast(new Rx.Subject())
             .refCount();

var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

var subscriptionA = source.subscribe(observerA);
// 訂閱數 0 => 1

var subscriptionB;
setTimeout(() => {
    subscriptionB = source.subscribe(observerB);
    // 訂閱數 0 => 2
}, 1000);

setTimeout(() => {
    subscriptionA.unsubscribe(); // 訂閱數 2 => 1
    subscriptionB.unsubscribe(); // 訂閱數 1 => 0，source 停止發送元素
}, 5000);
```

## publish
其实 `multicast(new Rx.Subject())` 很常用到，我们有一个简化的写法那就是 `publish`，下面这两段程式码是完全等价的。

```ts
var source = Rx.Observable.interval(1000)
             .publish() 
             .refCount();
             
// var source = Rx.Observable.interval(1000)
//             .multicast(new Rx.Subject()) 
//             .refCount();

// 加上 Subject 的三種變形
var source = Rx.Observable.interval(1000)
             .publishReplay(1) 
             .refCount();
             
// var source = Rx.Observable.interval(1000)
//             .multicast(new Rx.ReplaySubject(1)) 
//             .refCount();
var source = Rx.Observable.interval(1000)
             .publishBehavior(0) 
             .refCount();
             
// var source = Rx.Observable.interval(1000)
//             .multicast(new Rx.BehaviorSubject(0)) 
//             .refCount();
var source = Rx.Observable.interval(1000)
             .publishLast() 
             .refCount();
             
// var source = Rx.Observable.interval(1000)
//             .multicast(new Rx.AsyncSubject(1)) 
//             .refCount();
```

## share

另外 `publish + refCount` 可以在简写成 `share`
```ts
var source = Rx.Observable.interval(1000)
             .share();
             
// var source = Rx.Observable.interval(1000)
//             .publish() 
//             .refCount();

// var source = Rx.Observable.interval(1000)
//             .multicast(new Rx.Subject()) 
//             .refCount();
```

## Subject 与 Observable 的差异
永远记得 `Subject` 其实是`Observer Design Pattern` 的实现，所以当 `observer` 订阅到 `subject` 时，`subject` 会把订阅者塞到一份订阅者清单，在元素发送时就是在遍历这份清单，并把元素一一送出，这跟 `Observable` 像是一个 `function` 执行是完全不同的。