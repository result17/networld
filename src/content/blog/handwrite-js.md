---
author: cosin
pubDatetime: 2023-06-30T11:12:57+08:00 
title: js基本功能函数实现
postSlug: handwrite0js
featured: true
tags:
  - release
description: js基本功能函数简单实现
---

## 千分位分割

```js
const format = (n) => {
  let ans = "";
  let count = 0;
  let abs = Math.abs(n);
  while (abs) {
    let rest = abs % 10;
    abs = parseInt(abs / 10);
    ans += rest;
    count++;
    if (count % 3 === 0 && abs > 0) ans += ",";
  }
  let res = "";
  let i = ans.length;
  while (i > 0) {
    res += ans[--i];
  }
  return `${n < 0 ? "-" : ""}${res}`;
};
```

## debounce

```js
const debounce = (fn, duration, imd) => {
  let timer = null;
  return function (...args) {
    const context = this;
    const f = function () {
      timer = null;
      if (!imd) fn.apply(context, args);
    };
    const callNow = imd && !timer;
    clearTimeout(timer);
    timer = setTimeout(f, duration);
    if (callNow) fn.apply(context, args);
  };
};

const cat = {
  name: "rose",
  count: 0,
  print: function () {
    this.count++;
    console.log(`${this.name}: ${this.count}`);
  },
};
setInterval(debounce(() => cat.print(), 500), 1000);
```

## throttle

```js
const throttle = function (fn, limit) {
  let inThrottle = false;
  let timer = null;
  let lastRun = 0;

  return function (...args) {
    const context = this;
    if (!inThrottle) {
      fn.apply(context, args);
      inThrottle = true;
      lastRun = Date.now();
    } else {
      clearTimeout(timer);
      timer = setTimeout(() => {
        const now = Date.now();
        if ((now - lastRun) >= limit) {
          fn.apply(context, args);
          lastRun = now;
          inThrottle = false;
        }
      }, limit - (Date.now() - lastRun));
    }
  };
};
```

## package.json 中的 peerDependencies

在我们进行一些插件开发的时候会经常用到，比如 jQuery-ui 的开发依赖于
jQuery，html-webpack-plugin 的开发依赖于 webpack等。

## vue core hasOwn

```ts
const hasOwnProperty = Object.prototype.hasOwnProperty;
export const hasOwn = (
  val: object,
  key: string | symbol,
): key is keyof typeof val => hasOwnProperty.call(val, key);
```

## NaN !== NaN

```ts
const a = NaN;
const b = a;
// false
console.log(a === b);
```

## Promise

<!-- https://juejin.cn/post/7242493101088555064 -->
<!-- https://juejin.cn/post/7250383386183221285 -->

```ts
enum IPromiseState {
  Pending,
  Fulfilled,
  Rejected,
}

const runAsync = (cb: () => void) => {
  setTimeout(cb, 0);
};

const isThenable = (obj: any): obj is IPromise => {
  return typeof obj?.then === "function";
};

type Handler = {
  handleOnFulfilled: (value?: any) => void;
  handleOnRejected: (reason?: any) => void;
};

class IPromise {
  private state: IPromiseState = IPromiseState.Pending;

  private result?: any;

  private handlersList: Handler[] = [];

  constructor(
    executor: (
      resolve: (val: any) => void,
      reject: (reason: any) => void,
    ) => void,
  ) {
    try {
      executor(this.resolve, this.reject);
    } catch (error) {
      console.log(error);
      this.reject(error);
    }
  }

  private setResultAndState(value: any, state: IPromiseState) {
    if (this.state !== IPromiseState.Pending) {
      throw new Error("Promise is not pending.");
    }

    /**
     * const p = new Promise((res) => {
     *  setTimeout(() => res(1), 3000)
     *  })
     * const cp = Promise.resolve(p)
     * Promise {<pending>}
     */

    if (isThenable(value)) {
      value.then(this.resolve, this.reject);
      return;
    }
    this.result = value;
    this.state = state;
    this.executorHandlers();
  }

  private resolve = (val: any | IPromise) => {
    this.setResultAndState(val, IPromiseState.Fulfilled);
  };

  private reject = (reason?: any) => {
    this.setResultAndState(reason, IPromiseState.Rejected);
  };

  private executorHandlers = () => {
    if (this.state === IPromiseState.Pending) {
      throw new Error("Promise is pending, when executing handlers.");
    }

    runAsync(() => {
      for (const handler of this.handlersList) {
        const isFulFilled = this.state === IPromiseState.Fulfilled;
        isFulFilled
          ? handler.handleOnFulfilled(this.result)
          : handler.handleOnRejected(this.result);
      }
      this.handlersList.length = 0;
    });
  };

  then(
    onFullfilled?: (value: any) => any | IPromise,
    onRejected?: (reason: any) => any | IPromise,
  ) {
    return new IPromise((resolve: any, reject: any) => {
      const handler: Handler = {
        handleOnFulfilled: (value: any) => {
          if (!onFullfilled || typeof onFullfilled !== "function") {
            resolve(value);
          } else {
            try {
              resolve(onFullfilled(value as any));
            } catch (error) {
              reject(error);
            }
          }
        },
        handleOnRejected: (reason: any) => {
          if (!onRejected || typeof onRejected !== "function") {
            reject(reason);
          } else {
            try {
              resolve(onRejected(reason as any | IPromise));
            } catch (error) {
              reject(error);
            }
          }
        },
      };

      this.handlersList.push(handler);
      this.executorHandlers();
    });
  }

  catch(onRejected) {
    this.then(undefined, onRejected);
  }

  finally(onFinally: () => void) {
    this.then(
      (value) => {
        onFinally();
        return value;
      },
      (reason) => {
        onFinally();
        throw reason;
      },
    );
  }
}
```

## curry 函数柯里化

```ts
type AnyFunc = (...args: any[]) => any;

type Curry<Fn extends AnyFunc> = Parameters<Fn> extends
  [infer FirstArg, ...infer Rest]
  ? (arg: FirstArg) => Curry<(...args: Rest) => ReturnType<Fn>>
  : ReturnType<Fn>;

function curry<T extends AnyFunc, TAgg extends unknown[]>(
  func: T,
  agg?: TAgg,
): Curry<T> {
  const aggregatedArgs = agg ?? [];
  if (func.length === aggregatedArgs.length) return func(...aggregatedArgs);
  return ((arg: any) => curry(func, [...aggregatedArgs, arg])) as any;
}

function add(a: number, b: number, c: number): number {
  return a + b + c;
}

const curriedAdd = curry(add);

const add5 = curriedAdd(5);
const add5And6 = add5(6);
console.log(add5And6(7));
```

函数定义还是有点小问题

## Event bus

```ts
export interface Registry {
  unregister: () => void;
}

export interface Callable {
  // id
  [key: string]: Function;
}

export interface Subscriber {
  // event
  [key: string]: Callable;
}

export interface IEventBus {
  dispatch<T>(event: string, arg?: T): void;
  register(event: string, callback: Function): Registry;
}

export class EventBus implements IEventBus {
  private subscribers: Subscriber;
  private static nextId = 0;

  constructor() {
    this.subscribers = {};
  }

  public dispatch<T>(event: string, arg?: T): void {
    const subscriber = this.subscribers[event];
    if (subscriber === undefined) {
      return;
    }
    Object.keys(subscriber).forEach((key) => subscriber[key](arg));
  }

  public register(event: string, callback: Function): Registry {
    const id = this.getNextId();
    if (!this.subscribers[event]) this.subscribers[event] = {};
    this.subscribers[event][id] = callback;
    return {
      unregister: () => {
        delete this.subscribers[event][id];
        if (Object.keys(this.subscribers[event]).length === 0) {
          delete this.subscribers[event];
        }
      },
    };
  }

  private getNextId(): number {
    return EventBus.nextId++;
  }
}
```

## this指向

在非严格模式， 函数执行时没有明确执行上下文，则会指向全局对象（浏览器window,
Node或Deno指向Global） 在严格模式，this找不到会报undefined错误

## instanceof

```ts
const instanceOf = (left: Object, right: Function) => {
  let left_proto = Object.getPrototypeOf(left);
  const right_proto = right.prototype;
  while (left_proto) {
    if (left_proto === right_proto) return true;
    left_proto = Object.getPrototypeOf(left_proto);
  }
  return false;
};
```

## call

```ts
type AnyFnc = (...args: any[]) => any;

const iCall = <T extends AnyFnc>(
  fn: T,
  context: Object,
  ...args: any[]
): ReturnType<T> => {
  const uKey = Symbol("fn");
  context[uKey] = fn;
  const result = context[uKey](...args);
  delete result[uKey];
  return result;
};
```

## apply

```ts
type AnyFnc = (...args: any[]) => any;

const iApply = <T extends AnyFnc>(
  fn: T,
  context: Object,
  args: any[],
): ReturnType<T> => {
  const uKey = Symbol("fn");
  context[uKey] = fn;
  const result = context[uKey](...args);
  delete result[uKey];
  return result;
};
```

## bind

Object.setPrototypeOf() 静态方法可以将一个指定对象的原型（即内部的 [[Prototype]]
属性）设置为另一个对象或者 null。

Object.setPropertyOf 和 Object.create 的差别在于：

- Object.setPropertyOf，给我两个对象，我把其中一个设置为另一个的原型。

- Object.create，给我一个对象，它将作为我创建的新对象的原型。

```js
Function.prototype.bind = function (context, ...args) {
  if (typeof this !== "function") {
    throw new Error("Type Error");
  }
  // 保存this的值
  const self = this;

  function F() {
    // 考虑new的情况
    if (this instanceof F) {
      return self.call(this, ...args, ...arguments);
    }
    return self.call(context, ...args, ...arguments);
  }

  Object.setPrototypeOf(F.prototype, self.prototype);
  return F;
};

function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function () {
  return this.x + "," + this.y;
};

var p = new Point(1, 2);
p.toString(); // '1,2'

var emptyObj = {};
var YAxisPoint = Point.bind(emptyObj, 0 /*x*/);
// 以下这行代码在 polyfill 不支持,
// 在原生的bind方法运行没问题:
//(译注：polyfill的bind方法如果加上把bind的第一个参数，即新绑定的this执行Object()来包装为对象，Object(null)则是{}，那么也可以支持)
var YAxisPoint = Point.bind(null, 0 /*x*/);

var axisPoint = new YAxisPoint(5);

console.log(axisPoint instanceof Point); // true
console.log(axisPoint instanceof YAxisPoint);
console.log(axisPoint.toString());
```

```js
Function.prototype.bind = function (content) {
  if (typeof this != "function") {
    throw Error("not a function");
  }
  // 若没问参数类型则从这开始写
  let fn = this;
  let args = [...arguments].slice(1);

  let resFn = function () {
    return fn.apply(
      this instanceof resFn ? this : content,
      args.concat(...arguments),
    );
  };
  function tmp() {}
  tmp.prototype = this.prototype;
  resFn.prototype = new tmp();

  return resFn;
};
```

## Object.create

```js
const create = function (obj) {
  function F() {}
  F.prototype = obj;
  return new F();
};
```

## deepClone

```js
function deepClone(source) {
  let target = Object.create(
    Object.getPrototypeOf(source),
  );

  let stack = [{
    target,
    source,
  }];
  const sourceMap = new Map();
  Map.set(source, target);
  while (stack.length) {
    let { target, source } = stack.pop();
    for (let prop of Object.getOwnPropertyNames(source)) {
      let descriptor = Object.getOwnPropertyDescriptor(source, prop);
      if (descriptor.hasOwnProperty("value")) {
        const val = source[prop];
        if (typeof val === "object" && val !== null) {
          if (sourceMap.has(val)) {
            target[prop] = sourceMap.get(val);
          } else {
            target[prop] = {};
            sourceMap.set(target[prop], val);
            stack.push({
              target: target[prop],
              source: val,
            });
          }
        } else {
          target[prop] = val;
        }
      }
      Object.defineProperty(target, prop, descriptor);
    }
  }

  if (Object.isFrozen(source)) {
    Object.freeze(target);
  } else if (Object.isSealed(source)) {
    Object.seal(target);
  } else if (!Object.isExtensible(source)) {
    Object.preventExtensions(target);
  }
  sourceMap.clear();
  return target;
}
```

## 隐式原型继承

```ts
const create = (proto: Object) => {
  function temp() {}
  temp.prototype = proto;
  return new temp();
};
```

## 显式原型继承

```ts
const createInstance = (constructor: Function, ...args: any[]): Object => {
  const instance = Object.create(constructor.prototype);
  constructor.call(instance, ...args);
  return instance;
};
```

## 初级排序

```ts
const source = [7, 64, 25, 12, 22, 11];

const swap = (arr: number[], i: number, j: number) => {
  let temp = arr[j];
  arr[j] = arr[i];
  arr[i] = temp;
  return arr;
};

// 相邻元素进行比较 有序的子数组出现在数组结尾
const bubbleSort = (src: number[]) => {
  let i = 0;
  let len = src.length;
  while (i < len) {
    let j = 0;
    while (j < len - i - 1) {
      if (src[j] > src[j + 1]) {
        swap(src, j, j + 1);
      }
      j++;
    }
    i++;
  }
  return src;
};

// 特定索引元素进行比较更新索引 有序的子数组出现在数组开头或开头，交换最多n次
const selectSort = (src: number[]) => {
  let i = 0;
  let len = src.length;
  while (i < len) {
    let minIdx = i;
    let j = i + 1;
    while (j < len) {
      if (src[minIdx] > src[j]) {
        minIdx = j;
      }
      j++;
    }
    swap(src, minIdx, i);
    i++;
  }
  return src;
};

const insertSort = (src: number[]) => {
  let i = 1;
  let len = src.length;
  while (i < len) {
    let j = i - 1;
    let n = src[i];
    while (j >= 0 && n < src[j]) {
      src[j + 1] = src[j];
      j--;
    }
    swap(src, i, j + 1);
    i++;
  }
  return src;
};

console.log(bubbleSort(source));
console.log(selectSort(source));
console.log(insertSort(source));
```

冒泡排序、选择排序和插入排序都是简单的排序算法，它们的时间复杂度都是O(n^2)，在处理大规模数据时效率较低。但是在某些特定情况下，它们的性能可能会比较好。

冒泡排序的优点是代码实现简单，容易理解。但是其时间复杂度较高，尤其是在数据规模较大时，效率明显下降，所以不太适合处理大规模数据。

选择排序的优点是交换次数比较少，每次只需要交换一次，因此适合于数据移动较为昂贵的场景。但是选择排序的比较次数较多，因此在数据比较次数较为昂贵的场景中，选择排序的效率就会比较低。

插入排序的优点是比较次数比较少，每次只需要比较相邻的两个元素，因此适合于数据比较次数较为昂贵的场景。而且插入排序对于部分有序的数据集合，排序的时间复杂度会比较低。但是插入排序的移动次数较多，因此在数据移动次数较为昂贵的场景中，插入排序的效率就会比较低。

因此，冒泡排序、选择排序和插入排序各有优劣，取决于具体的应用场景和数据特征。在实际应用中，我们需要根据具体的情况选择合适的排序算法。如果需要高效排序大规模数据，可以考虑使用更高效的排序算法，如快速排序、归并排序、堆排序等。

## 实现一个 ts 的工具函数 UnGenericPromise<T> ，提取 Promise 中的泛型类型

```ts
type UnGenericPromise<T extends Promise<any>> = T extends Promise<infer U> ? U
  : never;
```

## 使用装饰器实现单例模式

```ts
export function singleton<T extends { new (...args: any[]): Object }>(
  constructor: T,
) {
  let instance: null | Object;

  return class Single extends constructor {
    static new(): Single {
      if (!instance) {
        instance = new constructor();
      }
      return instance;
    }
  };
}
```

## vue3 函数组件

```ts
<script setup lang='ts'>

import { ref, h } from "vue"

/**
 * Implement a functional component :
 * 1. Render the list elements (ul/li) with the list data
 * 2. Change the list item text color to red when clicked.
*/
const ListComponent = (props, context) => {
  const { emit } = context
  const children = props.list.map((it, index) => {
    return h('li', {
      onClick: () => emit('toggle', index),
      style: { color: props.activeIndex === index ? 'red' : '' }
    }, it.name)
  })

  return h("ul", children)
}

ListComponent.props = ['list', 'activeIndex']
ListComponent.emits = ['toggle']

const list = [{
  name: "John",
}, {
  name: "Doe",
}, {
  name: "Smith",
}]

const activeIndex = ref(0)

function toggle(index: number) {
  activeIndex.value = index
}

</script>

<template>
  <list-component
    :list="list"
    :active-index="activeIndex"
    @toggle="toggle"
  />
</template>
```

## rust 快速排序

```rs
fn quick_sort(array: &mut [i32]) {
    if array.len() <= 1 {
        return;
    }

    let pivot = array[0];
    let mut left = 1;
    let mut right = array.len() - 1;

    while left <= right {
        if array[left] < pivot {
            left += 1;
        } else if array[right] > pivot {
            right -= 1;
        } else {
            array.swap(left, right);
            left += 1;
            right -= 1;
        }
    }

    quick_sort(&mut array[0..left]);
    quick_sort(&mut array[left..array.len()]);
}
```

## css中间自适应两边定宽

```css
body {
  width: 100%;
  height: 400px;
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 50px 1fr 50px;
}

.left {
  background-color: red;
}

.right {
  background-color: aqua;
}

.mid {
  background-color: green;
}
```

```css
body {
  width: 100%;
  height: 400px;
  padding: 0;
  margin: 0;
  display: flex;
}

.left {
  background-color: red;
  width: 50px;
  height: 100%;
}

.right {
  background-color: aqua;
  width: 50px;
  height: 100%;
}

.mid {
  background-color: green;
  flex-grow: 1;
  height: 100%;
}
```

## new模拟

```js
function newOperator(ctor, ...args) {
  if (typeof ctor !== "function") {
    throw new TypeError("Type Error");
  }
  // 以ctor.prototype为原型创建一个对象
  const obj = Object.create(ctor.prototype);
  // 执行构造函数并将this绑定到新创建的对象上。
  const res = ctor.apply(obj, args);

  const isObject = typeof res === "object" && res !== null;
  const isFunction = typeof res === "function";
  return isObject || isFunction ? res : obj;
}
```

## simple p limit

```js
async function sleep(n, name = "test") {
  return new Promise((resolve) => {
    console.log(n, name, "start");
    setTimeout(() => {
      console.log(n, name, "end------------");
      resolve(n, name);
    }, n * 1000);
  });
}

async function asyncPool({ limit, tasks }) {


  const taskPool = new Set();

  for (const task of tasks) {
    const fn = async task => await task()
    const promiseTask = fn(task)
    taskPool.add(promiseTask);

    const clean = () => {
      taskPool.delete(promiseTask);
    };

    promiseTask.then(clean, clean);

    while (taskPool.size >= limit) {
      await Promise.race(taskPool);
    }
  }
  return Promise.allSettled(promiseTasks);
}

async function start() {
  await asyncPool({
    limit: 2,
    tasks: [
      () => sleep(3, "sleeping"),
      () => sleep(5, "gaming"),
      () => sleep(1, "eating"),
      () => sleep(1, "drinking"),
      () => sleep(3, "coding"),
      () => sleep(5, "running"),
      () => sleep(6, "crying"),
    ],
  });
}
start()
```
