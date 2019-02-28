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
  7. DOM event handler ->触发事件的dom元素_
_在箭头函数中，this由词法/静态作用域设置（set lexically）。它被设置为包含它的execution context的this，并且不再被调用方式影响（call/apply/bind）。_
_注意，当用call和apply而传进去作为this的不是对象时，将会调用内置的ToObject操作转换成对象。所以4将会装换成new Number(4)，而null/undefined由于无法转换成对象，全局对象将作为this。_
 _f.bind(someObject)会创建新的函数（函数体和作用域与原函数一致），但this被永久绑定到someObject，不论你怎么调用。_
_As a DOM event handler, this自动设置为触发事件的dom元素_

## 作用域
JavaScript采用Lexical Scope。

我们仅仅通过查看代码（因为JavaScript采用Lexical Scope），就可以确定各个变量到底指代哪个值。

变量的查找是从里往外的，直到最顶层（全局作用域），并且一旦找到，即停止向上查找。所以内层的变量可以shadow外层的同名变量。

_JS有eval和with两种机制，但两者都会导致代码性能差。_
