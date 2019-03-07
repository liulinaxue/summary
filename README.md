# summary
## 一个变量进入作用域中的方式有以下几种：
  - 1.language-defined:所有的作用域都会默认给出this和arguments两个变量名（global没有arguments）
  - 2.形参
  - 3.函数声明
  - 4.变量声明,包括函数声明
  
  _函数声明和变量声明总是会提升到他们所在作用域的顶部_
  
  _变量的解析顺序（优先级），与变量进入作用域的4种方式的顺序是一致的_
  
  ```
  function testOrder(arg) {
    console.log(arg); // arg是形参，不会被重新定义
    console.log(a); // 因为函数声明比变量声明优先级高，所以这里a是函数
    var arg = 'hello'; // var arg;变量声明被忽略， arg = 'hello'被执行
    var a = 10; // var a;被忽视; a = 10被执行，a变成number
    function a() {
        console.log('fun');
    } // 被提升到作用域顶部
    console.log(a); // 输出10
    console.log(arg); // 输出hello
}; 
testOrder('hi');
/* 输出：
hi 
function a() {
        console.log('fun');
    }
10 
hello 
*/
```
## this
当函数被调用，一个activation record（即 execution context）被创建。这个record包涵信息：函数在哪调用（call-stack），函数怎么调用的，参数等等。record的一个属性就是this，指向**函数执行期间**的this对象。
- 全局上下文 -> window
- 函数上下文
  1. 普通调用 -> window
  2. 方法调用 -> 方法所属的对象
  3. 构造函数调用 -> 返回的构造对象
  4. call/apply -> 指定的对象
  5. bind -> 指定对象 
  6. 箭头函数调用 -> execution context
  7. DOM event handler ->触发事件的dom元素
  
_在箭头函数中，this由词法/静态作用域设置（set lexically）。它被设置为包含它的execution context的this，并且不再被调用方式影响（call/apply/bind）。_
_注意，当用call和apply而传进去作为this的不是对象时，将会调用内置的ToObject操作转换成对象。所以4将会装换成new Number(4)，而null/undefined由于无法转换成对象，全局对象将作为this。_
 _f.bind(someObject)会创建新的函数（函数体和作用域与原函数一致），但this被永久绑定到someObject，不论你怎么调用。_
_As a DOM event handler, this自动设置为触发事件的dom元素_

## 作用域
JavaScript采用Lexical Scope。

我们仅仅通过查看代码（因为JavaScript采用Lexical Scope），就可以确定各个变量到底指代哪个值。

变量的查找是从里往外的，直到最顶层（全局作用域），并且一旦找到，即停止向上查找。所以内层的变量可以shadow外层的同名变量。

_JS有eval和with两种机制，但两者都会导致代码性能差。_

除了Global Scope，只有function可以创建新作用域（Function Scope）。不过es6中引入了块级作用域
with和try catch都可以创建Block Scope


## eventloop task(macrotask) microtask [不错的文章](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)

### promise.then(onFulfilled, onRejected)

onFulfilled or onRejected must not be called until the execution context stack contains only platform code. [3.1].

Here “platform code” means engine, environment, and promise implementation code. In practice, this requirement ensures that onFulfilled and onRejected execute asynchronously, after the event loop turn in which then is called, and with a fresh stack. This can be implemented with either a “macro-task” mechanism such as setTimeout or setImmediate, or with a “micro-task” mechanism such as MutationObserver or process.nextTick. Since the promise implementation is considered platform code, it may itself contain a task-scheduling queue or “trampoline” in which the handlers are called.

### Event Loop规范

每个浏览器环境，至多有一个event loop。
一个event loop可以有1个或多个task queue。
一个task queue是一列有序的task，用来做以下工作：Events task，Parsing task， Callbacks task， Using a resource task， Reacting to DOM manipulation task等。

### Jobs and Job Queues规范 _micro-task在ES2015规范中称为Job_
一个Job Queue是一个先进先出的队列。一个ECMAScript实现必须至少包含以下两个Job Queue：

1.ScriptJobs	Jobs that validate and evaluate ECMAScript Script and Module source text. See clauses 10 and 15.
2.PromiseJobs	Jobs that are responses to the settlement of a Promise (see 25.4).

单个Job Queue中的PendingJob总是按序（先进先出）执行，但多个Job Queue可能会交错执行。

_promise.then的执行其实是向PromiseJobs添加Job。_

**事件循环不间断在跑，执行任何进入队列的task。一个事件循环可以有多个task source，每个task source保证自己的任务列表的执行顺序，但由浏览器在（事件循环的）每轮中挑选某个task source的task。**

**microtask queue在回调之后执行，只要没有其它JS在执行中，并且在每个task的结尾。microtask中添加的microtask也被添加到microtask queue的末尾并处理。microtask包括mutation observer callbacks和promise callbacks。**

```
(function test() {
    setTimeout(function() {console.log(4)}, 0); //当前task运行，执行代码。首先setTimeout的callback被添加到tasks queue中；
    new Promise(function executor(resolve) {
        console.log(1);
        for( var i=0 ; i<10000 ; i++ ) {
            i == 9999 && resolve();
        }
        console.log(2);
    }).then(function() { //promise.then的callback被添加到microtasks queue中； 
        console.log(5); //已到当前task的end，执行microtasks，输出 5;
    });
    console.log(3); // output:1,2,3,5,4
})()
```
_setTimeout waits for a given delay then schedules a new task for its callback._

_Microtasks are usually scheduled for things that should happen straight after the currently executing script, or to make something async without taking the penalty of a whole new task._

_Any additional microtasks queued during microtasks are added to the end of the queue and also processed._

_Once a promise settles, or if it has already settled, it queues a microtask for its reactionary callbacks. This ensures promise callbacks are async even if the promise has already settled._

_the currently running script must finish before microtasks are handled_

_microtasks always happen before the next task_

**Some browsers running promise callbacks after setTimeout. It's likely that they're calling promise callbacks as part of a new task rather than as a microtask.**

**Treating promises as tasks leads to performance problems, as callbacks may be unnecessarily delayed by task-related things such as rendering. It also causes non-determinism due to interaction with other task sources, and can break interactions with other APIs, but more on that later.**

 - 1.asks execute in order, and the browser may render between them
 - 2.Microtasks execute in order, and are executed:
  - 1.after every callback, as long as no other JavaScript is mid-execution
  - 2.at the end of each task
  
解析结束后的操作
在此阶段，浏览器会将文档标注为交互状态，并开始解析那些**处于“deferred”**模式的脚本，也就是那些应在文档解析完成后才执行的脚本。然后，**文档状态将设置为“完成”**，一个**“加载”**事件将随之触发。

## [浏览器的工作原理](https://www.html5rocks.com/zh/tutorials/internals/howbrowserswork/)
