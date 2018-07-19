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


# Reference
* [Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)
* [Microtasks & Macrotasks  —  More On The Event Loop](https://abc.danch.me/microtasks-macrotasks-more-on-the-event-loop-881557d7af6f)
* [A deeper look at event loop (micro/macro tasks)](https://vcfvct.wordpress.com/2017/05/08/a-deeper-look-at-event-loop-micromacro-tasks/)
* [JavaScript Async - Microtasks](https://www.i-programmer.info/programming/javascript/11337-javascript-async-microtasks.html)
