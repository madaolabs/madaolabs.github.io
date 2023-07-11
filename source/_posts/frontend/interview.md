---
title: 面试题
date: 2023-07-11 08:23:30
---

# 知识点考察

### 1. redux-thunk 和 redux-saga的区别

redux-thunk 处理 side effect 的方式是：dispatch一个对象时不做处理；dispatch一个函数，在函数中处理异步动作，然后再dispatch一个对象

redux-saga 处理 side effect的方式是：自行建立了一套事件监听机制，接管所有的action。effects 方法有take, put, call, select, fork, takeEvery, takeLatest

### 2. 高阶函数和闭包

高阶函数的概念：一个函数的参数能够接受另一个函数作为参数，称之为高阶函数

闭包的概念：是通过函数返回函数的的方式，改变JS的回收机制

### 3. redux-sage中涉及了generator

生成器函数处理循环流程中控制各个步骤的呈现方式非常厉害，如：实现点击按钮，把逐个往同一类元素上添加className

```javascript
const strings = document.querySelectorAll('.string');
const btn = document.querySelector('#btn');
const className = 'darker';

function * addClassToEach(elements, className) {
  for (const el of Array.from(elements))
    yield el.classList.add(className);
}

const addClassToStrings = addClassToEach(strings, className);

btn.addEventListener('click', (el) => {
  if (!addClassToStrings.next().done)
    el.target.classList.add(className);
});
```

      不用递归实现菲波那切数列，用生成器函数实现

```javascript
function* fabonacci(seed1, seed2) {
	while(true){
		yield(() => {
			seed2 = seed1 + seed2;
			seed1 = seed2 - seed1;
			return seed2;
		})
	}
}
```

### 4. Promise

为什么使用Promise？

promise的功能是可以将复杂的异步处理轻松地进行模式化

Promise的并发控制

[https://www.yuque.com/felance/sfhx1u/gz8ogd](https://www.yuque.com/felance/sfhx1u/gz8ogd)

```javascript
class LimitPromise {
	constructor(max){
		// 最多执行的任务个数
    this.max = max;
    // 当前正在执行的任务个数
    this.count = 0;
    // 任务队列
    this.queue = [];
	}
	call(caller, ...args) {
		return new Promise((resolve, reject) => {
			const task = this.createTask(caller, args, resolve, reject);
			if (this.count >= this.max) {
        this.queue.push(task);
      } else {
        task()
      }
		})
	}

	createTask(caller, args, resolve, reject) => {
		return () => {
			this.count++
			caller(...args)
						.then(resolve)
						.catch(reject)
						.finally(() => {
							this.count--;
							if (this.queue.length) {
								const task = this.queue.shift();
		            task();
							}
						})
		}
	}
}
```

### 5. 节流函数

```javascript
function throttle(func, duration，delay = 100) {
    let timer = null, begin = new Date();
    return function() {
        current = new Date();
        clearTimeout(timer);
        if (current - begin > duration) {
            func();
            begin=current;
        } else {
            timer = setTimeout(func, delay);
        }
    }
}

const handler = function() {
    console.log('123');
}

window.onresize = throttle(handler, 100)
```

### 6. 面试知识点

[https://www.cnblogs.com/chenwenhao/p/11267238.html](https://www.cnblogs.com/chenwenhao/p/11267238.html)

### 7. 原型链

1. 什么是构造函数？

    ```javascript
    // A是一个构造函数
    function A() {
    	this.name = 'A';
    	console.log('123123');
    }
    ```

2. 原型对象  **原型对象的意义是 原型对象是子函数可以访问到的区域，可以实现继承。**

    ```javascript
    // A.prototype是原型对象
    // 原型对象有属性
    		constructor：指向构造函数, 
    		__proto__：指向父节点的prototype
    ```

3. 实例对象

    ```javascript
    // a是实例对象
    // 实例对象的__proto__属性 === 构造函数的prototype，即 a.__proto__ === A.prototype
    const a = new A();
    ```

4. 原型链就是__proto__指向，层层指向，知道为null到头

### 8. js继承

```javascript
function Parent() {

}
// 原型链继承
function SubType() {

}
SubType.prototype = new Parent();

// 借用构造函数继承
function SubType() {
	Parent.call(this)
}

// 原型链结合构造函数继承
function SubType() {
	Parent.call(this);
}
SubType.prototype = new Parent();
```

### 9. Express和Koa的中间件模型有什么区别

Express中间件模型是线性模型，中间件的执行过程没有对async，await做处理

Koa2中间件模型是洋葱模型，中间件的执行返回了Promise对象

### 10.http属性中控制缓存的哪些？

浏览器请求数据时经历的流程：1.强制缓存，2.协商缓存，3请求

1.强制缓存响应的header中有Expires/Cache-control 

**Cache-Control 优先级大于 Expires**

Expires的值为服务端返回的到期时间，即下一次请求时，请求时间小于服务端返回的到期时间，直接使用缓存数据。

Cache-Control 是最重要的规则。常见的取值有private、public、no-cache、max-age，no-store，默认为private。

private: 客户端可以缓存

public: 客户端和代理服务器都可缓存（前端的同学，可以认为public和private是一样的）

max-age=xxx: 缓存的内容将在 xxx 秒后失效

no-cache: 需要使用协商缓存来验证缓存数据（后面介绍）

no-store: 所有内容都不会缓存，强制缓存，协商缓存都不会触发（对于前端开发来说，缓存越多越好，so...基本上和它说886）

2.协商缓存

此时的状态码为304。

方案有2种：

1.Last-Modified/If-Modified-Since

第一次请求响应的header中有Last-Modified字段，值为时间。再次请求时request的header中有If-Modified-Since

2.Etag/If-None-Match

响应header中有Etag字段，值为唯一标识码，请求header的If-None-Match携带改字段

**Etag/If-None-Match 优先级大于 Last-Modified/If-Modified-Since**

### **11.变量，函数提升**

变量提升是使用var定义的变量，会把var的**声明**提升到作用域最前面

函数的提升是吧function提升到作用域最前面

### **12. new之后发生了什么事情，如何不用new创建实例对象**

```javascript
function _new(Func, ...args) {
	let obj = Object.create(Func.prototype);
	const result = Func.call(obj, ...args);
	typeof result === 'object' ? result : obj;
}
```

### 13.ES6和ES5有什么区别

1. 块级作用域(let和const)
2. 箭头函数
    1. 箭头函数使用bind，call，apply传入的this是没有效果的
3. 解构(数组解构，对象解构)
4. class extends super
5. 生成器
6. Promise
7. Map和Object有什么不同：
    1. 一个 Object 的键只能是字符串或者 Symbols，但一个 Map 的键可以是任意值。
    2. Map 中的键值是有序的（FIFO 原则），而添加到对象中的键则不是。
    3. Map 的键值对个数可以从 size 属性获取，而 Object 的键值对个数只能手动计算

### 14. 排序算法

1. **选择排序**

    选择排序是一个简单直观的排序方法，它的工作原理很简单，首先从未排序序列中找到最大的元素，放到已排序序列的末尾，重复上述步骤，直到所有元素排序完毕。

    时间复杂度O(n^2), 空间复杂度O(1);

    ```javascript
    function selectSort(arr) {
    	var index;
    	for(var i = 0; i < arr.length; i++) {
    		index = i;
    		for(var j = i+1; j < arr.length; j++) {
    			if (arr[index] < arr[j]) {
    				index = j;
    			}
    		}
    		if (index != i) {
    			var temp = arr[index];
    			arr[index] = arr[i];
    			arr[i] = temp;
    		}
    	}
    	return arr;
    }
    ```

2. **快速排序**

快速排序的算法思想是选择一个值，把小于这个值的放在左边，大于这个值得放在右边

时间复杂度: O(nlogn)
空间复杂度: O()

```javascript
function quickSort(arr) {
	if (arr.length <= 1) {
		return arr;
	}
	var leftArr = [];
	var rightArr = [];
	for(var i = 1; i< arr.length; i++) {
		if (arr[i] <= arr[0]) {
			leftArr.push(arr[i])
		} else {
			rightArr.push(arr[i]);
		}
    }
	return quickSort(leftArr).concat([arr[0]], quickSort(rightArr))
}
```

**3. 冒泡排序**

```javascript
function bubbleSort(arr) {
	for(var i = 0; i < arr.length - 1; i++) {
		for(var j = 0; j < arr.length - 1; j++) {
			if (arr[j] > arr[j + 1]) {
				var temp = arr[j];
				arr[j] = arr[j + 1];
				arr[j] = temp;
			}
		}
	}
	return arr;
}
```

### 15. js的垃圾回收机制，有怎样的问题

    JavaScript中有两种垃圾回收策略，标记清除和引用计数

1、标记清除法：

      [javascript](http://lib.csdn.net/base/javascript)最常用的垃圾收集方式。当变量进入环境时，这个变量标记为“进入环境”；而当变量离开环境时，则将其标记为“离开环境”。可以使用一个“进入环境”的变量列表及一个“离开环境”的变量列表来跟踪变量的变化，也可以翻转某个特殊的位来记录一个变量何时进入环境及离开环境。

2. 引用计数

      不太常见的垃圾收集策略。引用计数的含义是跟踪记录每个值被引用的次数。当声明了一个变量并将一个引用类型值赋给该变量时，则该值的引用次数就是1；如果同一个值又被赋给另一个变量，则该值的引用次数加1；如果包含对该值引用的变量又取得了另外一个值，则该值的引用次数减1。当该值的引用次数变为0时，则可以回收其占用的内存空间。当垃圾回收器下一次运行时，就会释放那些引用次数为0的值所占用的内存。

问题：循环引用

循环引用可以使用第三方的JSON.recycle来解决，核心原理是使用WeakMap来解决

### 16. 手写Promise的的关键特点

Promise 中的then方法中返回的是一个新的Promise，在Promise中使用的的setTimeout来实现异步

### 17. 高阶组件的属性代理，反向继承

属性代理：参数为组件，返回值为新组件的函数

反向继承：反向继承最核心的两个作用，一个是渲染劫持，另一个是操作state吧。反向继承有两种写法，两种写法存在不同的能力

反向继承是通过传进去参数是组件，然后继承该组件后重写，如：

```javascript
imoprt ComponentChild from './ComponentChild.js'
let iihoc = WrapComponet => class extends WrapComponet {
    constructor(props) {
	    super(props)
	    this.state = {
            num: 2000
	    }
    }
    componentDidMount() {
        console.log('iihoc componentDidMount')
        this.clickComponent()
    }
    return (
        <div>
            <div onClick={this.clickComponent}>iiHoc 点击</div>
            <div>{super.render()}</div>
        </div>
    )
}
export default iihoc(ComponentChild)
```

### 18. 实现add(1)(2)(3)()
```javascript
function add(...args) {
	return args.reduce((a, b) => a + b)
}

function currying(fn) {
	let args = []
	return function _c(...argv) {
		if (argv.length) {
			args = [
				...args,
				...argv
			]
			return _c;
		} else {
			return add(...args)
		}
	}
}
```

### 19. JavaScript深入之作用域链

### 20. DOM事件等级

### 21. useCallback 和 useMemo
React.memo(fn, fn(prevPros, nextProps))

useCallback(fn, [])

useCallback 和 React.memo 用于缓存函数

useMemo(fn, [])用于缓存值

### 22. React的diff算法
https://segmentfault.com/a/1190000039021724
React分为两个方法处理：reconcileSingleElement 和 reconcileChildrenArray

diff算法应对三种场景：节点更新、节点增删、节点移动

单节点：单节点更新、单节点增删。

多节点：多节点更新、多节点增删、多节点移动。

节点的更新是tag和key相同的情况下，属性发了变化

多节点的比较最多遍历3次新节点，

第一次遍历针对节点自身属性更新，剩下的两轮依次处理节点的新增、移动

其中移动的算法比较复杂，这里说明一下：

在第一次遍历中，我们确定了oldFider和newElements中分别的的固定节点的位置（lastPlacedIndex）为最后一个更新节点的index

移动的时候会将剩余的oldFiber节点放入一个以key为键，值为oldFiber节点的map中。称为existingChildren
然后遍历新节点，根据每一个新节点找到oldFiber中的节点index，和lastPlaceIndex比较，在oldFiber和newElements中如果lastPlaceIndex < index, 不移动，lastPlaceIndex更新，
否则判断oldFiber和newElements中一个大于，一个小于，则移动

### 23. 什么是React的fiber
fiber是React在V16之后引入，在这之前组件的渲染是通过一次性全部渲染，在组件数很多的时候，渲染时间比较长，如果用户在此时输入，则会明显感受到卡顿。

解决这个问题的方法就是分片，把一个耗时长的任务分成很多小片，每一个小片的运行时间很短。维护每一个分片的数据结构，就是Fiber。



### 24. 页面优化手段有哪些
1. 资源压缩，减少http请求
   1. 图片懒加载：将图片地址先用data-src属性保存，监听scroll，显示的时候在再加图片
   2. 
2. 非核心代码异步加载
   1. defer: 在HTML解析完之后才会执行。如果是多个，则按照加载的顺序依次执行
   2. async: 在加载完之后立即执行。如果是多个，执行顺序和加载顺序无关。
3. 浏览器缓存
	1. Response header Cache-Control: public, private, no-cache, no-store, max-age=30s
	2. Response header Expires
	3. Etag 和 If-None-Match
	4. Last-Modified 和 If-Modified-Since
4. 使用CDN

### 25. 前端安全
1. CSRF

2. XSS

### 26. 浏览器JS事件循环
1. 宏任务
   1. JS代码
   2. setTimeout
   3. setInterval 
2. 微任务
   1. Promise.then


每次执行完一个宏任务后，就会把微任务队列里的东西都执行完

这里要特别强调一下async和await的js执行

我们知道Promise中的异步体现在then和catch中，所以写在Promise中的代码是被当做同步任务立即执行的。而在async/await中，在出现await出现之前，其中的代码也是立即执行的。那么出现了await时候发生了什么呢？

#### await 做了什么呢？
从字面意思上看await就是等待，await 等待的是一个表达式，这个表达式的返回值可以是一个promise对象也可以是其他值。

很多人以为await会一直等待之后的表达式执行完之后才会继续执行后面的代码，**实际上await是一个让出线程的标志。await后面的表达式会先执行一遍**，将await后面的代码加入到microtask中，然后就会跳出整个async函数来执行后面的代码。

比如：
```
async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
}
```
等价于
```
async function async1() {
    console.log('async1 start');
    Promise.resolve(async2()).then(() => {
                console.log('async1 end');
        })
}
```

经典面试题
```javascript
async function b() {
    console.log('b');
    await c();
    console.log('b1');
}
async function c() {
    console.log('c');
    await new Promise(function(resolve, reject) {
        console.log('promise1');
        resolve();
    }).then(() => {
        console.log('promise1-1');
    });
    setTimeout(() => {
        console.log('settimeout1');
    });
    console.log('c1');
}
new Promise(function(resolve, reject) {
    console.log('promise2');
    resolve();
    console.log('promise2-1');
    reject();
}).then(() => {
    console.log('promise2-2');
    setTimeout(() => {
        console.log('settimeout2');
        new Promise(function(resolve, reject) {
            resolve();
        }).then(function() {
            console.log('?');
        });
    });
}).catch(() => {
    console.log('promise-reject');
});
console.log('200');
b();
```

结果:
```javascript
步骤讲解：
1、从上自下执行， new Promise 函数立即执行              打印 ：promise2   一
2、resolve()   将  promise2-2   放入微任务队列中
3、继续向下执行                                       打印 ：promise2-1  二
4、settimeout2放入宏任务队列
5、继续执行、无立即执行                                  打印 ：200       三
6、执行  b() 函数                                              打印 ：b  四
7、执行  await c() 函数                                        打印：c   五
8、进入 await c() 函数  中  nen Promise 函数             打印 ：promise1  六
9、resolve()   将  promise1-1   放入微任务队列中
10、settimeout1  放入宏任务队列
11、暂无执行任务   去微任务中 执行第一个进入微任务的待执行动作  打印：promise2-2   七
12、继续执行微任务中序列中待执行动作                           打印：promise1-1  八
13、微任务中暂无执行 动作 继续执行  c()  函数中的待执行代码            打印：c1    九
14、c() 执行完毕，继续执行 b() 函数中待执行代码                         打印：b1  十
15、至此，立即执行以微任务执行完毕，执行宏任务队列中第一个进入的待执行任务       打印 ：settimeout2   十一
16、宏任务一中，执行 new Promise  函数    resolve()   将问号 放入微任务中   立即执行微任务  打印：？ 十二
17、继续执行宏任务中待执行动作                                                 打印：settimeout1   十三
```

### 27. setState是异步还是同步

我的回答是执行过程代码同步的，只是合成事件和钩子函数的调用顺序在更新之前，导致在合成事件和钩子函数中没法立马拿到更新后的值，形式了所谓的“异步”，所以表现出来有时是同步，有时是“异步”。

只在合成事件和钩子函数中是“异步”的，在原生事件和 setTimeout/setInterval等原生 API 中都是同步的。简单的可以理解为被 React 控制的函数里面就会表现出“异步”，反之表现为同步。

为什么会有这种现象呢？我们通过React版本18.0.0-rc.2找找原因
原因是在调用setState之后，对于由合成时间和钩子函数发起的setState与原生事件发起的setState，处理不同，原生事件是直接处理，而合成事件是事件方法结束后，再执行

```javascript
ensureRootIsScheduled(root, eventTime);
// 以下代码判断如果不是合成事件的处理
if (
	lane === SyncLane &&
	executionContext === NoContext &&
	(fiber.mode & ConcurrentMode) === NoMode &&
	// Treat `act` as if it's inside `batchedUpdates`, even in legacy mode.
	!(__DEV__ && ReactCurrentActQueue.isBatchingLegacy)
) {
	// Flush the synchronous work now, unless we're already working or inside
	// a batch. This is intentionally inside scheduleUpdateOnFiber instead of
	// scheduleCallbackForFiber to preserve the ability to schedule a callback
	// without immediately flushing it. We only do this for user-initiated
	// updates, to preserve historical behavior of legacy mode.
	resetRenderTimer();
	flushSyncCallbacksOnlyInLegacyMode();
}

```



### 28. JS判断数据类型的方法
JS的基本数据类型有7中：number, string, boolean, undefined, null, symbol, bigint,
复杂数据类型有：object, array, function 和 Date
判断JS类型，有以下几种方法：1、typeof 2、object.property.toString.call 3、instance of。
1. typeof
```javascript
typeof 1 // number
typeof '' // string
typeof true // boolean
typeof undefined // undefined
typeof Symbol() // symbol
typeof 43n // bigint
```
null 返回的是 "object", 这个是基础类型里面的特例
```javascript
typeof null // object
```
而复杂数据类型里，除了函数返回了"function"其他均返回“object”
```javascript 
typeof({a:1}) // "object" 普通对象直接返回“object”
typeof [1,3] // 数组返回"object"
typeof(new Date) // 内置对象 "object"
```
函数返回"function"
```javascript
typeof function(){} // "function"
```
所以我们可以发现，typeof可以判断基本数据类型，但是难以判断除了函数以外的复杂数据类型。于是我们可以使用第二种方法，通常用来判断复杂数据类型，也可以用来判断基本数据类型。

2. Object.prototype.toString.call()
object.prototype.toString.call方法 ,他返回"[object, 类型]",注意返回的格式及大小写,前面是小写，后面是首字母大写。
基本数据类型都能返回相应的类型。
```javascript
Object.prototype.toString.call(999) // "[object Number]"
Object.prototype.toString.call('') // "[object String]"
Object.prototype.toString.call(Symbol()) // "[object Symbol]"
Object.prototype.toString.call(42n) // "[object BigInt]"
Object.prototype.toString.call(null) // "[object Null]"
Object.prototype.toString.call(undefined) // "[object Undefined]"
Object.prototype.toString.call(true) // "[object Boolean]
```
复杂数据类型也能返回相应的类型
```javascript
Object.prototype.toString.call({a:1}) // "[object Object]"
Object.prototype.toString.call([1,2]) // "[object Array]"
Object.prototype.toString.call(new Date) // "[object Date]"
Object.prototype.toString.call(function(){}) // "[object Function]"
```
3. instanceof, 可以左边放你要判断的内容，右边放类型来进行JS类型判断，只能用来判断复杂数据类型,因为 instanceof 是用于检测构造函数（右边）的 prototype 属性是否出现在某个实例对象（左边）的原型链上。
```javascript
[1,2] instanceof Array  // true
(function(){}) instanceof Function // true
({a:1}) instanceof Object // true
(new Date) instanceof Date // true
```

### 29. React的优化手段
1. 避免使用内联函数，避免在render方法中使用bing或者箭头函数
2. render中的参数尽量使用同一对象 比如
```javascript
<Foo style={{color: 'red'}} />
```
3. 使用Immutable
4. React懒加载 React.Suspense 和 React.lazy
5. 列表数据使用单独组建进行渲染

### 30. webpack优化手段
1. DllPlugin, 将不经常修改的第三方库和自己业务代码分开打包
2. 合理使用loader, 使用include和exclude
3. cacheDirectory缓存编译过的文件
4. tree Shaking分析import/exports依赖关系，对于没有使用的代码。可以自动删除。
5. 多线程编译
6. code split
7. 提取公共样式，ExtracTextPlugin
8. 按需加载

### 31. 动画animation，transform 和 transition
#### animation包含8个属性
- name: @keyframes的名称
- duration: 动画持续时间
- timing-function: 动画速度曲线：ease ease-in ease-out ease-in-out steps cubic-bezier(x1, y1, x2, y2)
- delay: 延迟执行时间，可以为负数
- iteration-count: 循环次数
- direction: 是否反复执行动画，如果动画被设置为只播放一次，该属性将不起作用。
- fill-mode: forwards动画结束后的位置，backwards动画开始的位置
- play-state: paused 和 running，属性指定动画是否正在运行或已暂停
#### transform 属性应用于元素的2D或3D转换。这个属性允许你将元素旋转，缩放，移动，倾斜等
- translate
- scale
- rotate, transform-origin设置圆心
- skew
- perspective

### transition
- property
- duration
- timing-function
- delay

思考题：如何实现一个秒针转动？

### 32. 鼠标移到一个div上，如何计算鼠标距离改div左上角的位置？
要理解
screenX，screenY 浏览器左上角

clientX, clientY 视窗左上角

offsetX, offsetY 事件源左上角

### 33. AMD、CMD、CommonJs、ES6的对比
AMD、CMD、CommonJs是ES5中提供的模块化编程的方案，import/export是ES6中定义新增的

AMD是RequireJS在推广过程中对模块定义的规范化产出，它是一个概念，RequireJS是对这个概念的实现，就好比JavaScript语言是对ECMAScript规范的实现。AMD是一个组织，RequireJS是在这个组织下自定义的一套脚本语言

CMD---是SeaJS在推广过程中对模块定义的规范化产出，是一个同步模块定义，是SeaJS的一个标准，SeaJS是CMD概念的一个实现，SeaJS是淘宝团队提供的一个模块开发的js框架.

CommonJS规范---是通过module.exports定义的，在前端浏览器里面并不支持module.exports,通过node.js后端使用的。Nodejs端是使用CommonJS规范的，前端浏览器一般使用AMD、CMD、ES6等定义模块化开发的

ES6特性，模块化---export/import对模块进行导出导入的

### 34. NodeList 和 HTMLCollection的区别
首先NodeList 是DOM 快照，节点数量和类型的快照，就是对节点增删，NodeList 感觉不到，但是对节点内部内容修改，是可以感觉到的，比如修改innerHTML;
HtmlCollection 是live绑定的，节点的增删是敏感的；

直接对NodeList， HtmlCollection进行赋值，是失败的

元素是可读的，是对dom节点的引用

然后我就想将NodeList,或者HtmlCollection 排个序啥的，很常见的需求
既然直接修改不行，那我先存到数组


### 堆的数据结构

堆的数据结构是完全二叉树。

#### 什么是完全二叉树呢？

完全二叉树是对树中的结点按从上至下、从左到右的顺序进行编号，如果编号为i（1≤i≤n）的结点与满二叉树中编号为i的结点在二叉树中的位置相同，则这棵二叉树称为完全二叉树。
#### 堆排序的数据结构

完全二叉树内如果每个节点的值都大于或等于子节点则为大顶堆，如果每个节点的值都小于或等于子节点则为小顶堆。
以上逻辑映射到数组可以定义为：
大顶堆 arr[i] >= arr[2*i + 1] && arr[i] >= arr[2 * i + 2]
大顶堆 arr[i] <=> arr[2*i + 1] && arr[i] <>= arr[2 * i + 2]


堆排序的时间复杂度最好情况，最坏情况，平均复杂度都是为O(nlogn)，空间复杂度为O(1)，为非稳定排序。


```Rust
fn main() {
    let mut arr = [1, 2, 3, 4, 6];
    let length = arr.len();
    heap_sort(&mut arr, length);

    println!("result ===> {:?}", arr);
}

/** 堆排序总结
    第一步，反向构造大顶堆，
    第二部，首尾交换
    第三部：交换后正向修复大顶堆
*/

fn heap_sort(arr: &mut [i64], length: usize) {
    // 构建大顶堆, 从下至上，从右向左，反向遍历所有非叶子节点，把最大值置于父节点位置
    let mut i = length / 2 - 1;
    while i >= 0 {
        adjust_heap(arr, i, length);
        if i > 0 {
            i -= 1;
        } else {
            break;
        }
    }
    let mut j = length - 1;
    while j > 0 {
        swap(arr, 0, j);
        adjust_heap(arr, 0, j);
        j -= 1;
    }
}

fn adjust_heap(arr: &mut [i64], mut i: usize, length: usize) {
    let temp = arr[i];
    let mut k = i * 2 + 1;
    // 正向遍历，使二叉树符合大顶堆结构,。
    // 正向遍历步骤：1. 判断两个子节点的大小，将k指向较大值
    // 2.k指向的值与temp节点比较
    while k < length {
        if k + 1 < length && arr[k] < arr[k + 1] {
            k += 1;
        }
        if arr[k] > temp {
            arr[i] = arr[k];
            i = k;
        } else {
            break;
        }
        k = k * 2 + 1;
    }
    arr[i] = temp;
}

fn swap(arr: &mut [i64], i: usize, j: usize) {
    let temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}

```

#### 总结
堆排序实现分为 3 个步骤
第一步：反向构造大顶堆，
第二部：收尾交换
第三部：交换后正向修复大顶堆
