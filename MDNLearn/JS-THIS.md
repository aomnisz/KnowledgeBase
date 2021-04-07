# 理解 JavaScript 中的 this

[CreateTime]: # (2021.04.07)
[ModifyTime]: # (2021.04.07)

似乎学前端的，就没能逃脱过这个的折磨。许多人都或参考一些资料或根据自己的理解来解释 this 。有些解释得很棒，而有些解释却是懂的都懂，不懂的还是不懂。尤其是那些一上来就显隐式绑定的，看都看腻了，却还是不明白。在翻阅了很多别人的文章之后（详见[参考资料](#参考资料)），总算是有一些些自己的理解了，于是想着把这个理解写下来，造福以后的我。

## this 与函数的调用方式有关

首先我想要讲述的是一种通用情况（Default Binding），也就是所谓的隐式绑定。说起来“隐式”好像很高大上的样子，实际上就是 JS 引擎的默认行为。举个可能不太恰当的例子，在 C++ 语言中申请一块内存，默认是返回 void 指针的，这样也可以说是 C++ “隐式”将指针类型设置成 void 。

**通常情况下，`this` 在函数执行的时候才确定，而不是在函数定义的时候。**可以看看 [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this) 的说明：

> In most cases, the value of this is determined by how a function is called (runtime binding).
> 
> 在大多数情况，this 的值是由函数被调用的方式所决定（运行时绑定）。

换句话说，**谁**调用函数，那么函数内部的 `this` 就指向**谁**。

那么，确定 this 的值的工作就转换成：确定函数的调用者。~~也就是 [ECMAScript](https://www.ecma-international.org/publications-and-standards/standards/ecma-262/) 中所说的 ThisBinding 。（不确定）~~

```javascript
function whatIsThis() {
    console.log(this);
}

var obj = {
    whatIsThis: whatIsThis
};

obj.whatIsThis();  // obj
window.whatIsThis();  // window
```

此外，还需要注意三点的就是：

1. JavaScript 中，用来切分变量的最小作用范围 (scope)，也就是我们说的有效范围的单位，就是 function 。也就是说，function 中的 this 和 function 中的 function 中的 this 是不一样的。
    ```javascript
    var obj = {
        func1: function () {
            console.log(this);  // obj
            var func2 = function () {
                console.log(this);  // window
            };
            func2();
        }
    };

    obj.func1();
    ```
2. 在没有特定指明 this 的情況下，默认绑定 (Default Binding) 的 this 是 window 。而在严格模式 `"use strict";` 下，会禁止 this 自动指定为 window ，此时 this 为 undefined 。
    ```javascript
    function notStrict() {
        console.log(this);
    }

    function isStrict() {
        "use strict";
        console.log(this);
    }

    // 无特定指明调用者
    notStrict();  // window
    isStrict();   // undefined

    // 有特定指明调用者
    window.notStrict();  // window
    window.isStrict();   // window
    ```
3. 在处理 HTML 事件中，函数是由 DOM 元素调用的，也就是说 this 指向该 DOM 元素。
    ```javascript
    btn = document.createElement("button");

    btn.addEventListener("click", function(event) {
        console.log(this);
    }, false);

    btn.click();  // <button></button>
    ```

## 手动指定 this 的几种方式

手动确定 this 的绑定有 4 种方法，分别是：`bind()`、`apply()`、`call()`、`new`。

### 1. bind(object)

`bind` 会创建一个新的函数，该函数的 this 会指定为 object ，且不会再受运行时绑定的影响。

```javascript
var obj = {
    note: "this is obj"
}

function func() {
    console.log(this);
    return this;
}

var objFunc = func.bind(obj);

objFunc();  // obj
window.objFunc();  // obj

objFunc === func;  // false
```
   
### 2. apply(object) & call(object)

`apply` 和 `call` 不会创建新的函数，只会在运行时绑定 this 。

`apply` 和 `call` 的行为十分相似，只是使用方式不一样。除了第一个参数是用于绑定的 object ，`apply` 接受的剩余参数是一个数组，可以用同为 `a` 开头的 `array` 帮助记忆；而 `call` 接受的剩余参数是用逗号分离的参数，可以用同为 `c` 开头的 `comma` 帮助记忆。

```javascript
var obj = {
    note: "this is obj"
}

function func(...args) {
    console.log(this);
    console.log(args);
    return this;
}

func.apply(obj, [1, 2], 3, 4);          // obj, [1, 2]
func.call (obj, [1, 2], 3, 4);          // obj, [[1, 2], 3, 4]

func.apply(obj, [[1, 2], 3, 4]);        // obj, [[1, 2], 3, 4]

window.func.apply(obj, [[1, 2], 3, 4]); // obj, [[1, 2], 3, 4]
window.func.call (obj,  [1, 2], 3, 4 ); // obj, [[1, 2], 3, 4]
```

### 3. new

本质上就是调用了 apply 或 call 。~~虽然我还没找到相关的代码说明，但是看网上那些手写 new 的代码，都是调用这两个函数，所以就这样大胆猜测了。~~

## 箭头函数的 this 绑定

**箭头函数的 this 是在声明时确定的，而不是运行时确定的。箭头函数内的 this 值是箭头函数声明时所处的环境的 this 值。**这种绑定的理解就有些类似于其他语言的理解方式了，但还是会有一丢丢不同。虽然说箭头函数的 this 是声明时确定，理解方式跟其他语言相同，但是箭头函数所处的函数的 this 却是运行时确定的。

```javascript
var obj1 = {
    func1: function () {
        console.log(this);
        var func11 = function () {
            console.log(this);
        };
        func11();
    }
};
var obj2 = {
    func2: function () {
        console.log(this);
        var func22 = () => {
            console.log(this);
        };
        func22();
    }
};

obj1.func1();  // obj1 window
obj2.func2();  // obj2 obj2
```

这样做虽然有些怪异，要让人用两种办法去确定 this 的值，但也确实是有它存在的道理。它在回调函数中十分有用。在没有箭头函数的时候，我们想要在回调函数中使用 this ，就需要在函数外将 this 保存到 that ，而有了箭头函数之后，就需要再这么操作了。

```javascript
var obj = {
    func1: function() {
        console.log(this);
        setTimeout(function() {
            console.log(this);
        }, 0);
    },
    func2: function() {
        console.log(this);
        var that = this;
        setTimeout(function() {
            console.log(that);
        }, 0);
    },
    func3: function() {
        console.log(this);
        setTimeout(() => {
            console.log(this);
        }, 0);
    }
}

obj.func1();  // obj window
obj.func2();  // obj obj
obj.func3();  // obj obj
```

## 参考资料

- [What's THIS in JavaScript ? [上] | Kuro's Blog](https://kuro.tw/posts/2017/10/12/What-is-THIS-in-JavaScript-%E4%B8%8A/)
- [What's THIS in JavaScript ? [中] | Kuro's Blog](https://kuro.tw/posts/2017/10/17/What-s-THIS-in-JavaScript-%E4%B8%AD/)
- [What's THIS in JavaScript ? [下] | Kuro's Blog](https://kuro.tw/posts/2017/10/20/What-is-THIS-in-JavaScript-%E4%B8%8B/)
- [JavaScript 深入之从 ECMAScript 规范解读 this](https://github.com/mqyqingfeng/Blog/issues/7)
