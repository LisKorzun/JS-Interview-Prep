# What is the event loop in Javascript?

The entire process of reading code, queueing up tasks, and executing tasks is the event loop.

The __Event Loop__ is a queue of callback functions. When an async function executes, the callback function is pushed into the queue. 
The JavaScript engine doesn't start processing the event loop until the code after an async function has executed. 
This means that JavaScript code is not multi-threaded even though it appears to be so. 
(__Javascript is single threaded.__ This means Javascript will process each line of code after the previous line of code.)
The event loop is a __first-in-first-out (FIFO) queue__, meaning that callbacks execute in the order they were added onto the queue.

### Concurrency model based on an "event loop"

``:grey_exclamation:   QUEUE   :grey_exclamation:``

A JavaScript runtime uses a message queue, which is a list of messages to be processed. 
Each message has an associated function which gets called in order to handle the message.

At some point during the event loop, the runtime starts handling the messages on the queue, starting with the oldest one. 
To do so, the message is removed from the queue and its corresponding function is called with the message as an input parameter. 
As always, calling a function creates a new stack frame for that function's use.

![Concurrency model based on an "event loop"](https://developer.mozilla.org/files/4617/default.svg)

The processing of functions continues until the stack is once again empty; then the event loop will process the next message in the queue (if there is one).

__Event Table (Hash) and Event Queue:__ _When the Javascript engine comes across a line of code that needs to be run, it places the ‘trigger’ of that execution and the action of it in an event table. Later on, when the engine checks the event table, any events that have transpired, the corresponding action is moved into an event queue. 
Later on, when the engine checks the event queue, if there is an event there to execute it will do so. After executing, it will check the event table again to see if any new events have transpired. Javascript is looping through this process over and over._

In the case of a callback, when the initial function is invoke (let’s say ``setTimeout``), the trigger-action pair of ‘1000ms in the future’ and callback are added to the event table. Later on, when the Javascript engine checks the event table and ‘1000ms in the future’ has transpired, it will add the callback to the event queue.
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

As expected, the console outputs "a", "b", and then 500 ms(ish) later, we see "c", "d", and "e". I use "ish" because ``setTimeout`` is actually unpredictable. In fact, even the HTML5 spec talks about this issue:

> "This API does not guarantee that timers will run exactly on schedule. Delays due to CPU load, other tasks, etc, are to be expected."

The function ``setTimeout`` is called with 2 arguments: a message to add to the queue, and a time value (optional; defaults to 0). The time value represents the (minimum) delay after which the message will actually be pushed into the queue. If there is no other message in the queue, the message is processed right after the delay; however, if there are messages, the ``setTimeout`` message will have to wait for other messages to be processed. For that reason, the second argument indicates a minimum time and not a guaranteed time.

# Reference
* [__MDN:__ Concurrency model and Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)
* [__Medium:__ Event Loop Horizon — Javascript Event Loop](https://abc.danch.me/event-loop-horizon-javascript-event-loop-637c1932985)
* [__TutsPlus:__ Event-Based Programming: What Async Has Over Sync](https://code.tutsplus.com/tutorials/event-based-programming-what-async-has-over-sync--net-30027)
