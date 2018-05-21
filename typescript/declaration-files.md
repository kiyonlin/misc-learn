# 声明文件
当使用第三方库时，我们需要引用它的声明文件。

## 声明语句
假如我们想使用第三方库，比如 `jQuery`，我们通常这样获取一个 `id` 是 `foo` 的元素：

```ts
$('#foo');
// or
jQuery('#foo');
```

但是在 `TypeScript` 中，我们并不知道 `$` 或 `jQuery` 是什么东西：

```ts
jQuery('#foo');

// index.ts(1,1): error TS2304: Cannot find name 'jQuery'.
```

这时，我们需要使用 `declare` 关键字来定义它的类型，帮助 `TypeScript` 判断我们传入的参数类型对不对：

```ts
declare var jQuery: (string) => any;

jQuery('#foo');
```

`declare` 定义的类型只会用于编译时的检查，编译结果中会被删除。

上例的编译结果是：

```ts
jQuery('#foo');
```

## 声明文件
通常我们会把类型声明放到一个单独的文件中，这就是声明文件：

```ts
// jQuery.d.ts

declare var jQuery: (string) => any;
```

我们约定声明文件以 `.d.ts` 为后缀。

然后在使用到的文件的开头，用「三斜线指令」表示引用了声明文件：

```ts
/// <reference path="./jQuery.d.ts" />

jQuery('#foo');
```