##JS MarcoTasks MicroTasks

在JS的`event loop`中，有两种任务队列`microtasks`和`macrotasks`

**microtasks**

- process.nextTick
- Promise
- Object.observe
- MutationObserver

**macrotasks**

- setTimeout
- setInterval
- setImmediate
- I/O(Ajax, fs)
- UI渲染

在一个事件循环event loop的周期中，一个task应该从macrotask队列开始执行。当这个macrotask结束后，所有的microtasks将在同一个cycle中执行。    
且在microtasks执行时还可以加入更多的microtask，然后一个一个的执行，直到microtask队列清空。   

```js
console.log('start')

const interval = setInterval(() => {  
  console.log('setInterval')
}, 0)

setTimeout(() => {  
  console.log('setTimeout 1')
  Promise.resolve()
      .then(() => {
        console.log('promise 3')
      })
      .then(() => {
        console.log('promise 4')
      })
      .then(() => {
        setTimeout(() => {
          console.log('setTimeout 2')
          Promise.resolve()
              .then(() => {
                console.log('promise 5')
              })
              .then(() => {
                console.log('promise 6')
              })
              .then(() => {
                clearInterval(interval)
              })
        }, 0)
      })
}, 0)

Promise.resolve()
    .then(() => {  
        console.log('promise 1')
    })
    .then(() => {
        console.log('promise 2')
    })
```

event loop1:    
macrotasks: [主程序代码]   
microtasks: []    

执行macrotasks队列，也即执行主程序代码，收集macro或micro的tasks    
输出: 'start'   
收集的macrotasks（下次循环的）: [setInterval, setTimeout]    
收集的microtasks（当前循环的）: [Promise]    

macrotasks队列执行完毕，这时候microtasks: [Promise]不为空，执行microtasks队列     
输出： 'promise1'    
输出 ： 'promise2'   
这时候microtasks为空

下次循环开始之前的队列状态     
macrotasks: [setInterval, setTimeout]   
microtasks: []    

event loop2:    
执行macrotasks队列      
执行setInterval      
输出：'setInterval'，且收集setInterval到下次循环的macrotasks中    
执行setTimeout    
输出：'setTimeout 1'，且收集Promise到当前循环的microtasks: [Promise]   

由于当前循环的microtasks不为空，执行队列中的任务Promise     
输出：'promise3'    
输出: 'promise4'    
收集setTimeout到下次循环的macrotasks中
这时候microtasks为空

下次循环开始之前的队列状态   
macrotasks: [setInterval, setTimeout]   
microtasks: []

event loop3:    
执行macrotasks队列    
执行setInterval   
输出: 'setInterval'，且收集setInterval到下次循环的macrotasks中   
执行setTimeout    
输出: 'setTimeout2'，且收集Promise到当前的microtasks: [Promise]

由于当前循环的microtasks不为空，执行队列中的任务Promise
输出: 'promise5'  
输出: 'promise6'    
清楚定时器clearInterval，所以下次循环的macrotasks的setInterval被清除

下次循环开始之前的队列状态   
macrotasks: []    
microtasks: []

event loop4:    
由于macrotasks为空，和microtasks为空，程序处于等待状态。

上面程序总的输出接口是
```js
// event loop1
start

promise 1
promise 2

// event loop2
setInterval
setTimeout 1

promise 3
promise 4

// event loop3
setInterval
setTimeout 2

promise 5
promise 6
```

## 总结

- 一个循环开始的时候microtasks是空的，或者说当前循环的microtasks一开始是空的，在macrotasks执行完后可能不为空
- microtasks要等到macrotasks队列执行完毕才会开始执行，且microtasks的任务在执行的过程中，是可以添加任务的，只要当前循环还未结束
- 在当前循环中收集的macro任务是收集到下一个循环的macrotasks，而当前循环收集的micro任务是收集到当前microtasks中





