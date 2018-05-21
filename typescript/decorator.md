# 修饰器

## 类的修饰
```ts
@testable
class MyTestableClass {
  // ...
}

function testable(target) {
  target.isTestable = true;
}

MyTestableClass.isTestable // true
```

上面代码中，`@testable` 就是一个修饰器。它修改了 `MyTestableClass` 这个类的行为，为它加上了静态属性 `isTestable`。`testable` 函数的参数 `target` 是`MyTestableClass` 类本身。

基本上，修饰器的行为就是下面这样。

```ts
@decorator
class A {}

// 等同于

class A {}
A = decorator(A) || A;
```

也就是说，修饰器是一个对类进行处理的函数。修饰器函数的第一个参数，就是所要修饰的目标类。

```ts
function testable(target) {
  // ...
}
```

上面代码中，`testable` 函数的参数 `target`，就是会被修饰的类。

如果觉得一个参数不够用，可以在修饰器外面再封装一层函数。

```ts
function testable(isTestable) {
  return function(target) {
    target.isTestable = isTestable;
  }
}

@testable(true)
class MyTestableClass {}
MyTestableClass.isTestable // true

@testable(false)
class MyClass {}
MyClass.isTestable // false
```

上面代码中，修饰器 `testable` 可以接受参数，这就等于可以修改修饰器的行为。

注意，修饰器对类的行为的改变，是代码编译时发生的，而不是在运行时。这意味着，修饰器能在编译阶段运行代码。也就是说，**修饰器本质就是编译时执行的函数**。

前面的例子是为类添加一个静态属性，如果想添加实例属性，可以通过目标类的 `prototype` 对象操作。

```ts
function testable(target) {
  target.prototype.isTestable = true;
}

@testable
class MyTestableClass {}

let obj = new MyTestableClass();
obj.isTestable // true
```
上面代码中，修饰器函数 `testable` 是在目标类的 `prototype` 对象上添加属性，因此就可以在实例上调用。

下面是另外一个例子。

```ts
// mixins.js
export function mixins(...list) {
  return function (target) {
    Object.assign(target.prototype, ...list)
  }
}

// main.js
import { mixins } from './mixins'

const Foo = {
  foo() { console.log('foo') }
};

@mixins(Foo)
class MyClass {}

let obj = new MyClass();
obj.foo() // 'foo'
```

上面代码通过修饰器 `mixins`，把 `Foo` 对象的方法添加到了 `MyClass` 的实例上面。

## 方法的修饰

```ts
class Person {
  @readonly
  name() { return `${this.first} ${this.last}` }
}
```

上面代码中，修饰器 `readonly` 用来修饰“类”的 `name` 方法。

修饰器函数 `readonly` 一共可以接受三个参数。

```ts
function readonly(target, name, descriptor){
  // descriptor对象原来的值如下
  // {
  //   value: specifiedFunction,
  //   enumerable: false,
  //   configurable: true,
  //   writable: true
  // };
  descriptor.writable = false;
  return descriptor;
}

readonly(Person.prototype, 'name', descriptor);
// 类似于
Object.defineProperty(Person.prototype, 'name', descriptor);
```

修饰器第一个参数是类的原型对象，上例是 `Person.prototype`，修饰器的本意是要“修饰”类的实例，但是这个时候实例还没生成，所以只能去修饰原型（这不同于类的修饰，那种情况时 `target` 参数指的是类本身）；第二个参数是所要修饰的属性名，第三个参数是该属性的描述对象。

另外，上面代码说明，修饰器（`readonly`）会修改属性的描述对象（`descriptor`），然后被修改的描述对象再用来定义属性。

下面的 `@log` 修饰器，可以起到输出日志的作用。

```ts
class Math {
  @log
  add(a, b) {
    return a + b;
  }
}

function log(target, name, descriptor) {
  var oldValue = descriptor.value;

  descriptor.value = function() {
    console.log(`Calling ${name} with`, arguments);
    return oldValue.apply(this, arguments);
  };

  return descriptor;
}

const math = new Math();

// passed parameters should get logged now
math.add(2, 4);
```

上面代码中，`@log` 修饰器的作用就是在执行原始的操作之前，执行一次 `console.log`，从而达到输出日志的目的。