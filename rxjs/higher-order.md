# Higher Order Observable
`Higher Order Observable` 就是指一个 `Observable` 送出的元素还是一个 `Observable`，就像是二维数组一样，一个数组中的每个元素都是数组。

有三个方法

- concatAll
- switch
- mergeAll

## concatAll

从下面 `Marble Diagram` 可以看得出来，当我们点击一​​下`click` 事件会被转成一个`observable` 而这个`observable` `会每一秒送出一个递增的数值，当我们用concatAll` 之后会把二维的 `observable` 摊平成一维的 `observable`，但 `concatAll` 会一个一个处理，一定是等前一个 `observable` 完成(`complete`)才会处理下一个`observable`，因为现在送出 `observable` 是无限的永远不会完成(`complete`)，就导致他永远不会处理第二个送出的`observable`!

```
click  : ---------c-c------------------c--.. 
        map(e => Rx.Observable.interval(1000))
source : ---------o-o------------------o--..
                   \ \
                    \ ----0----1----2----3----4--...
                     ----0----1----2----3----4--...
                     concatAll()
example: ----------------0----1----2----3----4--..
```

现在把送出的 `observable` 限制只取前三个元素，用 `Marble Diagram` 表示如下

```
click  : ---------c-c------------------c--.. 
        map(e => Rx.Observable.interval(1000))
source : ---------o-o------------------o--..
                   \ \                  \
                    \ ----0----1----2|   ----0----1----2|
                     ----0----1----2|
                     concatAll()
example: ----------------0----1----2----0----1----2--..
```

这里我们把送出的 `observable` 变成有限的，只会送出三个元素，这时就能看得出来 `concatAll` 不管两个 `observable` 送出的时间多么相近，一定会先处理前一个 `observable` 再处理下一个。

## switch
`switch` 最重要的就是他会在新的 `observable` 送出后直接处理新的 `observable` 不管前一个 `observable` 是否完成，每当有新的 `observable` 送出就会直接把旧的 `observable` 退订(`unsubscribe`)，永远只处理最新的`observable`!

```
click  : ---------c-c------------------c--.. 
        map(e => Rx.Observable.interval(1000))
source : ---------o-o------------------o--..
                   \ \                  \----0----1--...
                    \ ----0----1----2----3----4--...
                     ----0----1----2----3----4--...
                     switch()
example: -----------------0----1----2--------0----1--...
```

所以在这上面的 `Marble Diagram` 可以看得出来第一次送出的 `observable` 跟第二次送出的 `observable` 时间点太相近，导致第一个 `observable` 还来不及送出元素就直接被退订了，当下一次送出 `observable` 就又会把前一次的 `observable` 退订。

## mergeAll
`mergeAll` 会把二维的 `observable` 转成一维的，并且能够同时处理所有的 `observable`

```
click  : ---------c-c------------------c--.. 
        map(e => Rx.Observable.interval(1000))
source : ---------o-o------------------o--..
                   \ \                  \----0----1--...
                    \ ----0----1----2----3----4--...
                     ----0----1----2----3----4--...
                     switch()
example: ----------------00---11---22---33---(04)4--...
```

从 `Marble Diagram` 可以看出来，所有的 `observable` 是并行(`Parallel`)处理的，也就是说 `mergeAll` 不会像 `switch` 一样退订(`unsubscribe`)原先的 `observable` 而是并行处理多个 `observable`。点击越多下，最后送出的频率就会越快。

另外 `mergeAll` 可以传入一个数值，这个数值代表他可以同时处理的 `observable` 数量。

```
click  : ---------c-c----------o----------.. 
        map(e => Rx.Observable.interval(1000).take(3))
source : ---------o-o----------c----------..
                   \ \          \----0----1----2|     
                    \ ----0----1----2|  
                     ----0----1----2|
                     mergeAll(2)
example: ----------------00---11---22---0----1----2--..
```
当 `mergeAll` 传入参数后，就会等处理中的其中一个 `observable` 完成，再去处理下一个。以我们的例子来说，前面两个 observable 可以被并行处理，但第三个 `observable` 必须等到第一个 `observable` 结束后，才会开始。

我们可以利用这个参数来决定要同时处理几个 `observable`，如果我们传入 `1` 其行为就会跟 `concatAll` 是一模一样的，这点在源码可以看到他们是完全相同的。

## concatMap
`concatMap` 其实就是 `map` 加上 `concatAll` 的简化写法。

```ts
function getPostData() {
    return fetch('https://jsonplaceholder.typicode.com/posts/1')
    .then(res => res.json())
}
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source.concatMap(
                    e => Rx.Observable.from(getPostData()));

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
```

每当点击一下画面就会送出一个 `HTTP request`，如果我们快速的连续点击，可以在开发者工具的 `network` 看到每个 `request` 是等到前一个 `request` 完成才会送出下一个 `request`。

`concatMap` 还有第二个参数是一个 `selector callback`，这个 `callback` 会传入四个参数，分别是

- 外部 observable 送出的元素
- 内部 observable 送出的元素
- 外部 observable 送出元素的 index
- 内部 observable 送出元素的 index

```ts
function getPostData() {
    return fetch('https://jsonplaceholder.typicode.com/posts/1')
    .then(res => res.json())
}
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source.concatMap(
                e => Rx.Observable.from(getPostData()), 
                (e, res, eIndex, resIndex) => res.title);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
```

这个例子的外部 `observable` 送出的元素就是 `click event` 对象，内部 `observable` 送出的元素就是 `response` 对象，这里我们回传 `response` 对象的`title` 属性，这样一来我们就可以直接收到 `title`，这个方法很适合用在 `response` 要选取的值跟前一个事件或顺位(`index`)相关时。

## switchMap 和 mergeMap
和 `switch` 和 `merge` 作用类似，`mergeMap`可以传入第三个参数，限制并行处理数量。

## concatMap、switchMap、mergeMap总结
`switchMap`, `mergeMap`, `concatMap`有一个共同的特性，那就是这三个 `operators` 可以把第一个参数所回传的 `promise` 对象直接转成 `observable`，这样我们就不用再用 `Rx.Observable.from` 转一次。

### 使用情境

- `concatMap` 用在可以确定内部的 `observable` 结束时间比外部 `observable` 发送时间来快的情境，并且不希望有任何并行处理行为，适合少数要一次一次完成到底的的 `UI` 动画或特别的 `HTTP request` 行为。
- `switchMap` 用在只要最后一次行为的结果，适合绝大多数的使用情境。
- `mergeMap` 用在并行处理多个 `observable`，适合需要并行处理的行为，像是多个 `I/O` 的并行处理。
