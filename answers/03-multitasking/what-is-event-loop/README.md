# What is the event loop in Javascript?

The entire process of reading code, queueing up tasks, and executing tasks is the event loop.

The __Event Loop__ is a queue of callback functions. When an async function executes, the callback function is pushed into the queue. 
The JavaScript engine doesn't start processing the event loop until the code after an async function has executed. 
This means that JavaScript code is not multi-threaded even though it appears to be so. 
The event loop is a __first-in-first-out (FIFO) queue__, meaning that callbacks execute in the order they were added onto the queue.

__Javascript is single threaded__. This means Javascript will process each line of code after the previous line of code.

__Event Table (Hash) and Event Queue.__
 
When the Javascript engine comes across a line of code that needs to be run, it places the ‘trigger’ of that execution and the action of it in an event table. Later on, when the engine checks the event table, any events that have transpired, the corresponding action is moved into an event queue. 
Later on, when the engine checks the event queue, if there is an event there to execute it will do so. After executing, it will check the event table again to see if any new events have transpired. Javascript is looping through this process over and over.

In the case of a callback, when the initial function is invoke (let’s say setTimeout), the trigger-action pair of ‘1000ms in the future’ and callback are added to the event table. Later on, when the Javascript engine checks the event table and ‘1000ms in the future’ has transpired, it will add the callback to the event queue.
In reality, the callback is executed the next time the Javascript engine checks the event queue. In fact it is possible this may not be right away, and so the actual timing of the callback being executed is not 1000ms.

```javascript
console.log( "a" );
setTimeout(function() {
    console.log( "c" )
}, 500 );
setTimeout(function() {
    console.log( "d" )
}, 500 );
setTimeout(function() {
    console.log( "e" )
}, 500 );
console.log( "b" );
```

As expected, the console outputs "a", "b", and then 500 ms(ish) later, we see "c", "d", and "e". I use "ish" because setTimeout is actually unpredictable. In fact, even the HTML5 spec talks about this issue:

> "This API does not guarantee that timers will run exactly on schedule. Delays due to CPU load, other tasks, etc, are to be expected."

# Reference
* [__MDN:__ Concurrency model and Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)
* [__Medium:__ Event Loop Horizon — Javascript Event Loop](https://abc.danch.me/event-loop-horizon-javascript-event-loop-637c1932985)
* []()
