# 手写Promise
## PromiseA规范
>Promise对象代表一个异步操作，有三种状态：pending（进行中）、fulfilled（已成功）和       rejected（已失败）。一旦成功就不允许失败，一旦失败就不允许成功。
>Promise接收一个函数作为参数，该函数有两个参数，一个是resolve，表示成功时执行的函数，一个是reject，表示失败失败时执行的函数。resolve执行时传入的参数会作为then方法中第一个回调函数的参数，reject执行传入的参数会作为then方法中第二函数回调的参数
>then方法接收的两个函数中，可以通过return把值传给下一个步，也可以返回一个新的Promise把值传给下一步，then方法执行的时候有个特点，就是为了保证链式调用，上一次then中不管你是成功还是失败都会把参数作为下一个then中成功时回调的参数
先来实现一个es5的同步版本：

```js
function Promise(excutor){
   let self = this
   self.status = 'pending'
   self.value = null
   self.reason = null
function resolve(value){
  if(self,status==='pending'){
    self.status = 'fulfailled'
    self.value= value
  }
}
function reject(reason){
  if(self.status==='pending'){
    self.status = 'rejected'
    self.resaon = reason
  }
}
try{
  excutor(resolve,reject)
   }catch(err){
    reject(err)
   }
}
Promise.prototype.then = function(onFulfailled, onRejected){
    let self = this
    if(self.status==='fulfailled'){
    onFulfailled(self.value)
    }
    if(self.status==='rejected'){
    onRejected(self.reason)
    }
}
```
es5支持异步的版本（订阅者模式）
```js
function Prominse(excutor){
    let self = this
    self.status = 'pending'
    self.value = null
    self.reason = null
    self.onFulfailledCallbacks = []
    self.onRejectedCallbacks = []
    function resolve(value){
   if(self.status==='pending'){
    self.status='fulfailled'
    self.value=value
    slef.onFulfailledCallbacks.forEach(item=>item(self.vslue))
   }
}
function reject(value){
   if(self.status==='pending'){
     self.status='rejected'
     self.value=value
     slef.onRejectedCallbacks.forEach(item=>item(self.vslue))
   }
}
try{
  excutor(resolve,reject)
}catch(err){
  reject(err)
}
}
Promise.prototype.then = function(onFulfailled, onRejected){
    let self = this
    if(self.status==='fulfailled'){
     onFulfailled(self.value)
    }
    if(self.status==='rejected'){
     onRejected(self.reason)
    }
    if(self.staus==='pending'){
      self.onFulfailledCallbacks.push(onFulfailed)
       self.onRejectedCallbacks.push(onRejected)
    }
}
```