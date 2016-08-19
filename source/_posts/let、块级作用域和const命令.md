---
title: let、块级作用域和const命令
date: 2016-08-18 16:26:17
tags:
- ES6
---
## **一、let命令**

**特点（无变量提升，暂时性死区，相同作用域内不能重复声明，不能在函数内部声明参数）**

###### 1.基本用法


let 命令用于声明变量，用法类似与 var ，但所声明的变量只在     let 命令所在代码块内有效。

    {
        let a = 10;
        var b = 1;
    }
    a // ReferemceError: a is not defined
    b // 1

<!-- more -->

for循环的计数器就很适合用let

    var a = [];
    for(let i = 0;i<10;i++){
        a[i] = function(){
            console.log(i)
        };
    }
    a[6](); // 6

如果上面的代码将let换成var，则会打印出10；

###### 2.不存在变量提升

let不像var那样会有变量提升，所以变量一定要在声明后使用。

    console.log(foo); // ReferenceError
    let foo = 2;

    typeof(foo); // ReferenceError
    let foo;

###### 3.暂时性死区

只要块级作用域内存在let命令，它所声明的变量则绑定在这个区域：

    var a = 123;
    if(true){
        a = 'abc' // ReferenceError
        let a;
    }

在ES6中，如果区块中存在let和const命令，则这个区块对这些命令声明的变量从一开始就形成封闭的作用域。只要在声明变量前使用这些变量就会报错。这在语法上称为“暂时性死区”（tmporal dead zone,简称TDZ）

    if(true){
        // TDZ开始
        tmp = 'abc'; // ReferenceError
        console.log(tmp); // ReferenceError

        // TDZ结束
        let tmp;
        console.log(tmp); // undefined
    }

###### 4.不允许重复声明

let不允许在相同作用域内重复声明同一个变量。

    // 报错
    function(){
        var a = 10;
        let a = 10;
    }
    function(){
        let a = 10;
        let a = 1;
    }

因此，不能在函数内部重新声明参数：

    // 报错
    function(arg){
        let arg;
    }
    
    // 不报错
    function(arg){
        {
            let arg;
        }
    }

## 二、块级作用域

**为什么需要块级作用域**

ES5中只有全局作用域和函数作用域，没有块级作用域，这带来许多不合理场景。

场景1，内层变量覆盖外层变量：

    var tmp = new Date();
    function f(){
        console.log(tmp);
        if(false){
            var tmp = "hello world"
        }
    }

上面的代码中，输出结果为undefined,因为var会有变量提升，从而覆盖了外层的tmp变量。

场景2，用来计数的循环变量泄漏为全局变量：

    var s = 'hello';
    for(var i = 0;i<s.length;i++){
        console.log(s[i]);
    }
    console.log(i); // 5

上面的代码中，变量i用来控制循环。但是循环结束后，它并没有消失，而是泄漏成了全局变量。

**ES6的块级作用域**

###### 1.let实际上为JavaScript新增了块级作用域。
    
    function f1(){
        let n = 5;
        if(true){
            let n = 10;
        }
        console.log(n); // 5
    }

上面的函数有两个代码块，都声明了变量n，但是最终打印结果为5。这就说明外层代码不受内层代码影响。如果使用var声明，则打印结果为10。

###### 2.ES6允许块级作用域任意嵌套。

    {{{{{
        {let a = 'abc'}
        console.log(a); // 报错
    }}}}}

###### 3.内层作用域可以定义外层作用域的同名变量.

    {{{{{
        {let name = 'hello';}
        let name = 'world';
    }}}}}

###### 4.块级作用与的出现，使得立即执行匿名函数（IIFE）不再必要。

    // IIFE写法
    (function(){
        var tmp = ...;
        ...
    }());

    // 块级作用域写法
    {
        let tmp = ...;
        ...
    }

###### 5.另外ES6也规定，函数本身的作用域在其所在的块级作用域内。

    function f(){console.log('i am outside);};
    (function (){
        if(false){
            function f(){console.log('i am inside')};
        }
        f();
    }());

因为ES5中函数提升，不管会不会进入到if代码块，函数声明都会提升到当前作用域的顶部而得到执行，所以在ES5中会打印'i am inside'。而在ES6中，无论是否会进入if代码块，其内部声明的函数都不会影响到作用域外部。

## 三、const命令

**特点（声明常量，只在声明所在块级作用域内有效，暂时性死区，不能重复声明**）

###### 1.声明常量语法

    const PI = 3.1415;
    PI // 3.1415
    PI = 3; // TypeError:"PI" is read-only

const声明的常量不能改变值，这就意味着声明的时候必须初始化，只声明不赋值会报错。

###### 2.作用域

与let命令相同，只在声明的块级作用域内有效。

    if(true){
        const MAX = 5;
    }
    MAX; // ReferenceError:MAX is not defined

###### 3.暂时性死区

const命令声明的常量也不提升，同样存在暂时性死区，只能在声明后使用。

    if(true){
        console.log(MAX); // ReferenceErroe
        const MAX = 6;
    }

###### 4.不能重复声明

与let一样，const不能重复声明。

    var name = 'Jack';
    let age = 5;

    // 下面两行报错
    const name = 'Rose';
    const age = 15;

###### 5.指向

对于复合型的变量，变量名不指向数据，而是指向数据所在地址。const只能保证指向的地址不变，不能保证地址的数据不变，所以将一个对象声明为常量必须非常小心。

    const foo = {}
    foo.prop = 123;

    foo.prop // 123

    foo = {} // TypeError

上面的代码中，常量foo存储的是一个地址，指向一个对象。不可变的只是地址，即不能把foo指向另外一个地址，但是这个对象本身是可变的，所以依然可以添加新的属性。

## 四、跨模块常量

const声明的常量只在当前代码块有效。如果想设置跨模块的常量，可以采用下面的写法：

    // constants.js 模块
    export const A = 1;
    export const B = 2;
    export const C = 3;

    // test1.js 模块
    import * as constants from './constants';
    console.log(constants.A); // 1
    console.log(constants.B); // 2

    // test2.js 模块
    import {A,B} from './constants';
    console.log(A); // 1
    console.log(B); // 2

## 五、全局对象的属性

在ES6中，var和function命令声明的全局变量依旧是全局对象（window）的属性；而let,const,class命令声明的全局变量则不属于全局对象的属性。

    var a = 1;
    window.a // 1

    let b = 2;
    window.b // undefined

**ES5中，只有两种声明变量的方法：var和function命令。ES6添加了let和const命令外，还有import和class命令，所以ES6中一共有6中声明变量的方法**








