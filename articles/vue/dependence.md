首先为什么要依赖收集 

我们先看一下观察者模式吧 先理解这个设计模式比较好理解这里

什么是观察者模式呢  有点复杂后面我单独拿出来讲一下设计模式吧

那我们先跳过设计模式（vue的响应式原理用到的就是观察者模式）来看一下依赖收

集干了什么吧，例如：

```js
  new Vue({
    template: 
        `<div>
            <span>{{test1}}</span> 
            <span>{{test2}}</span> 
      <div>`,
    data:{
      test1: 'i am test1',
      test2: 'i am test2',
      test3: 'i am test3'  
    }  
  })
```
按照上一讲我们说的这里的直接去observer data的变化去同步更新test1和test2

的变化,发现问题没有？这里的test3其实是没有用到的，那我observer整个对象的变化

是不是有点不太好？ 

再举个栗子

假设我们定义了一个全局变量（先挂载到Vue.prototype上吧）,

然后我们在不同的组件内部用到这个变量

```js
Vue.prototype.test1 = 'test1' //全部是伪代码方便理解用的
new vue({
   data:{
      test1: this.test1
   } 
})
new vue({
   data:{
      test1: this.test1
   } 
})
```
这个时候如果test1修改了，怎么通知到我依赖test1的这两个组件去更新视图，

而其他组件不更新呢？

ok，大概思路是不是有了，就是监听对象只有当属性做set操作的时候我才去改变它


那么怎么确保所有依赖的属性都变化呢 当然就是在get的时候去收集起来（这里可以看一下

之前手写promise的时候,then方法没有直接去执行方法而是push,一样的道理
）

那我们来改造一下之前的代码

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
    } //呃 没错 前面复制粘贴过来的
```
我们修改一下这个defineReactive

```js
  const depObj = {
      deps:[],
      update: function(value){
         this.deps.forEach(dep=>{
             console.log(value + '更新啦...')
         })
      }
  } //定义一个全局变量来依赖收集 ==> 就是那些地方引用了我这个对象的属性

 function defineReactive (obj, key, val) {
           depObj.deps = []
            Object.defineProperty(obj, key, {
                enumerable: true,       
                configurable: true,  
                get: function reactiveGetter () {
                    depObj.deps.push(this)
                    return val;         
                },
            set: function reactiveSetter (newVal) {
                if (newVal === val) return;
                depObj.update(newVal);
            }
        });
    }    
// observer还是用昨天那个

function observer (obj) {
   if (!obj || (typeof obj !== 'object')) {
        return;
    }
    
  Object.keys(obj).forEach((key) => {
        defineReactive(obj, key, obj[key]); //defineReactive跟上面一样
    });
  }

let testObj1= {
        name: 'zj',
         age: '25',
        face: 'beatiful'
    }
    observer(testObj1)
    testObj1.name  // 这时候会触发依赖收集 可以打印depObj里面的deps会有值
    testObj1.name = 'hhc' // hhc更新啦 
```

我们可不可以用类去改造一下

```js
class Dep {
    constructor () {
        this.subs = [];
    }
    addSub (sub) {
        this.subs.push(sub);
    }
    notify () {
        this.subs.forEach((sub) => {
            sub.update();
        })
    }
}

class Watcher {
    constructor () {
        /* 在new一个Watcher对象时将该对象赋值给Dep.target，在get中会用到 */
        Dep.target = this;
    }

    /* 更新视图的方法 */
    update () {
        console.log("视图更新啦～");
    }
}

Dep.target = null;

function defineReactive (obj, key, val) {
    /* 一个Dep类对象 */
    const dep = new Dep();
    
    Object.defineProperty(obj, key, {
        enumerable: true,
        configurable: true,
        get: function reactiveGetter () {
            /* 将Dep.target（即当前的Watcher对象存入dep的subs中） */
            dep.addSub(Dep.target);
            return val;         
        },
        set: function reactiveSetter (newVal) {
            if (newVal === val) return;
            /* 在set的时候触发dep的notify来通知所有的Watcher对象更新视图 */
            dep.notify();
        }
    });
}
```

我们来试一下


```js
class Vue {
    constructor(options) {
        this.data = options.data;
        observer(this.data);
        /* 新建一个Watcher观察者对象，这时候Dep.target会指向这个Watcher对象 */
        new Watcher();
        /* 在这里模拟render的过程，为了触发test属性的get函数 */
        console.log('render~', this.data.test);
    }
}
```