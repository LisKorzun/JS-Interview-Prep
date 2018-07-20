# Microtasks & Macrotasks : what Does It All Mean?

There are macrotasks (known as tasks) and microtasks. In reality, these two types of tasks occupy different queues, and do they don’t mix.

```javascript
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});

console.log('script end');
```

__Tasks__ are scheduled so the browser can get from its internals into JavaScript/DOM land and ensures these actions happen sequentially. 
Between tasks, the browser may render updates. Getting from a mouse click to an event callback requires scheduling a task, as does parsing HTML, 
and in the above example, `setTimeout`.

`setTimeout` waits for a given delay then schedules a new task for its callback. This is why `setTimeout` is logged after script end, 
as logging script end is part of the first task, and `setTimeout` is logged in a separate task.

__Microtasks__ are usually scheduled for things that should happen straight after the currently executing script, such as reacting to a batch of actions, 
or to make something async without taking the penalty of a whole new task. 
The microtask queue is processed after callbacks as long as no other JavaScript is mid-execution, and at the end of each task. 
Any additional microtasks queued during microtasks are added to the end of the queue and also processed. 
Microtasks include mutation observer callbacks, and as in the above example, promise callbacks.

Once a promise settles, or if it has already settled, it queues a microtask for its reactionary callbacks. 
This ensures promise callbacks are async even if the promise has already settled. 
So calling `.then(yey, nay)` against a settled promise immediately queues a microtask. 
This is why `promise1` and `promise2` are logged after script end, as the currently running script must finish before microtasks are handled. 
`promise1` and `promise2` are logged before `setTimeout`, __as microtasks always happen before the next task__.


> *__Examples of Macro__
  setTimeout, setInternal, setImmediate, I/O tasks*
  
> *__Examples of Micro__
  process.nextTick, Promises, mutation observer callbacks*
  
```javascript
(function test() {
    setTimeout(function() {console.log(4)}, 0);
    new Promise(function executor(resolve) {
        console.log(1);
        for( var i=0 ; i<10000 ; i++ ) {
            i == 9999 && resolve();
        }
        console.log(2);
    }).then(function() {
        console.log(5);
    });
    console.log(3);
})()
```

So why the result is `1,2,3,5,4` rather than `1,2,3,4,5`？

The async of `setTimeout` is different from the async of `Promise.then`, at least they are not in the same async queue.

__`THE EVENT LOOP PROCESS MODEL IS AS FOLLOWS:`__

__*when call stack is empty,*__ do the steps-

1. select the oldest task(task A) in task queues
2. if task A is null(means task queues is empty),jump to step 6
3. set “currently running task” to “task A”
4. run “task A”(means run the callback function)
5. set “currently running task” to null,remove “task A”
6. perform microtask queue
    * (a).select the oldest task(task x) in microtask queue
    * (b).if task x is null(means microtask queues is empty),jump to step (g)
    * (c).set “currently running task” to “task x”
    * (d).run “task x”
    * (e).set “currently running task” to null,remove “task x”
    * (f).select next oldest task in microtask queue,jump to step(b)
    * (g).finish microtask queue;
7. jump to step 1.

__`A SIMPLIFIED PROCESS MODEL IS AS FOLLOWS:`__

1. run the oldest task in macrotask queue,then remove it.
2. run all available tasks in microtask queue,then remove them.
3. next round:run next task in macrotask queue(jump step 2)

__`SOMETHING TO REMEMBER:`__

1. when a task (in macrotask queue) is running,new events may be registered.So new tasks may be created.Below are two new created tasks:
    * `promiseA.then()` is callback is a task
        * promiseA is resolved/rejected:  the task will be pushed into microtask queue __in current round of event loop__.
        * promiseA is pending:  the task will be pushed into microtask queue __in the future round of event loop__ (may be next round)
    * `setTimeout`(callback,n)’s callback is a task,and will be pushed into macrotask queue,even n is 0;
2. __*task in microtask queue will be run in the current round, while task in macrotask queue has to wait for next round of event loop*__.
3. callback of `“click”`, `”scroll”`, `”ajax”`, `”setTimeout”`… are tasks, however we should also remember __js codes as a whole in script tag is a task(a macrotask) too__.

# Reference
* [Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)
* [Microtasks & Macrotasks  —  More On The Event Loop](https://abc.danch.me/microtasks-macrotasks-more-on-the-event-loop-881557d7af6f)
* [A deeper look at event loop (micro/macro tasks)](https://vcfvct.wordpress.com/2017/05/08/a-deeper-look-at-event-loop-micromacro-tasks/)
* [JavaScript Async - Microtasks](https://www.i-programmer.info/programming/javascript/11337-javascript-async-microtasks.html)
