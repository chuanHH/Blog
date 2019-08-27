
前面已经讲过如何实现双向数据绑定了

Vue.js 是一款 MVVM 框架，数据模型仅仅是普通的 JavaScript 对象，但是对这些对象进行操作时，却能影响对应视图，它的核心实现就是「响应式系统」。

尽管我们在使用 Vue.js 进行开发时不会直接修改「响应式系统」，但是理解它的实现有助于避开一些常见的「坑」，也有助于在遇见一些琢磨不透的问题时可以深入其原理来解决它。

[vue官网深入响应式原理] (https://cn.vuejs.org/v2/guide/reactivity.html)

响应式的核心还是Object.defineProperty这个前面双向数据绑定已经使用过啦

前面的例子只是单个数据的双向绑定（可观察） 响应式是把整个object都变成可观察模式

我们先来简单实现一下
    
   
    ```js
    function cb(val){
        console.log(val + '更新啦...')
    }

    function defineReactive (obj, key, val) {
            Object.defineProperty(obj, key, {
                enumerable: true,       /* 属性可枚举 */
                configurable: true,     /* 属性可被修改或删除 */
                get: function reactiveGetter () {
                    return val;         /* 实际上会依赖收集，下一小节会讲 */
                },
            set: function reactiveSetter (newVal) {
                if (newVal === val) return;
                cb(newVal);
            }
        });
    }
    ```

    我们来测试一下

    ```js
        let testObj = {
            name: 'hhc',
            age: '27',
            face: 'handsome'
        }
        defineReactive(testObj, 'name')
        testObj.name = 'zj' //zj更新啦
        defineReactive(testObj, 'age')
        testObj.age = '25' //25更新啦 没有赋值成功是因为我们在set里面没有去做赋值操作

    ```

    上面这样我们就监听了对象里面某个key值的变化啦

    但是这样一个个写很麻烦呀 所以我们再封装一下直接监听整个对象的变化

    ```js
    function observer (obj) {
    if (!obj || (typeof obj !== 'object')) {
        return;
    }
    
    Object.keys(obj).forEach((key) => {
        defineReactive(obj, key, obj[key]); //defineReactive跟上面一样
    });
   }
    ```
    我们再来试一下

    ```js
        let testObj1= {
            name: 'zj',
            age: 'z5',
            face: 'beatiful'
        }
        observer(testObj1)

        testObj1.name = 'hhc' // hhc更新啦 
        testObj1.age= '27' // 27更新啦  现在就整个对象的变化都是可监听的啦
    ```

    ok，那我们回到vue里面，自己动手写一个vue

    ```js
    class Vue {
    constructor(options) {
        this.data = options.data;
        observer(this.data); // observer跟上面一样
    }
   }
   let vue1 = new Vue({
       data:{
         test:'i am test'
       }
   })
   vue1.data.test = 'i am vue' // i am vue更新啦
    ```


ok到这里我们实现了一个最初的vue雏形，后面我们慢慢来增加。。。