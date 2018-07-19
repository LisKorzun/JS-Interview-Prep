# What is the event loop in Javascript?

The entire process of reading code, queueing up tasks, and executing tasks is the event loop.

The __Event Loop__ is a queue of callback functions. When an async function executes, the callback function is pushed into the queue. 
The JavaScript engine doesn't start processing the event loop until the code after an async function has executed. 
This means that *JavaScript code is not multi-threaded* even though it appears to be so. 
(__Javascript is single threaded.__ This means Javascript will process each line of code after the previous line of code.)
The event loop is a __first-in-first-out (FIFO) queue__, meaning that callbacks execute in the order they were added onto the queue.

### Concurrency model based on an "event loop"

__`QUEUE`__

A JavaScript runtime uses an event queue, which is __a list of events to be processed. 
Each event has an associated function__ which gets called in order to handle the event.

At some point during the event loop, the runtime starts handling the events on the queue, __starting with the oldest one__. 
To do so, the event is removed from the queue and its corresponding function is called with the event as an input parameter. 
As always, calling a function creates a new stack frame for that function's use.

![Concurrency model based on an "event loop"](https://developer.mozilla.org/files/4617/default.svg)

The processing of functions continues until the stack is once again empty; then the event loop will process the next message in the queue (if there is one).

__`STACK`__

Function calls form a stack of frames (context).

```javascript
function foo(b) {
  var a = 10;
  return a + b + 11;
}

function bar(x) {
  var y = 3;
  return foo(x * y);
}

console.log(bar(7)); //returns 42
```
When calling bar, a first frame is created containing bar's arguments and local variables. 
When bar calls foo, a second frame is created and pushed on top of the first one containing foo's arguments and local variables. 
When foo returns, the top frame element is popped out of the stack (leaving only bar's call frame). When bar returns, the stack is empty.

__`HEAP`__

Objects are allocated in a heap which is just a name to denote a large mostly unstructured region of memory.

_In other words:_
> *__Event Table (Hash) and Event Queue:__ When the Javascript engine comes across a line of code that needs to be run, 
it places the ‘trigger’ of that execution and the action of it in an event table. 
Later on, when the engine checks the event table, any events that have transpired, the corresponding action is moved into an event queue. 
Later on, when the engine checks the event queue, if there is an event there to execute it will do so. 
After executing, it will check the event table again to see if any new events have transpired. Javascript is looping through this process over and over.*

### Relating to callbacks

In the case of a callback, when the initial function is invoke (let’s say `setTimeout`), 
the trigger-action pair of ‘1000ms in the future’ and callback are added to the event table. 
Later on, when the Javascript engine checks the event table and ‘1000ms in the future’ has transpired, it will add the callback to the event queue.
In reality, the callback is executed the next time the Javascript engine checks the event queue. 
In fact it is possible this may not be right away, and so the actual timing of the callback being executed is not 1000ms.

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

As expected, the console outputs "a", "b", and then 500 ms(ish) later, we see "c", "d", and "e". I use "ish" because `setTimeout` is actually unpredictable. 
In fact, even the HTML5 spec talks about this issue:

> "This API does not guarantee that timers will run exactly on schedule. Delays due to CPU load, other tasks, etc, are to be expected."

The function `setTimeout` is called with 2 arguments: an message (event, callback) to add to the queue, and a time value (optional; defaults to 0). 
The time value represents the (minimum) delay after which the message will actually be pushed into the queue. 
If there is no other message in the queue, the message is processed right after the delay; 
however, if there are messages, the ``setTimeout`` message will have to wait for other messages to be processed. 
For that reason, __the second argument indicates a minimum time and not a guaranteed time__.

__`ZERO DELAYS`__

Zero delay doesn't actually mean the call back will fire-off after zero milliseconds. 
Calling `setTimeout` with a delay of 0 (zero) milliseconds doesn't execute the callback function after the given interval.

The execution depends on the number of waiting tasks in the queue. 
In the example below, the message `'this is just a message'` will be written to the console before the message in the callback gets processed, 
because the delay is the minimum time required for the runtime to process the request, but not a guaranteed time.

Basically, the `setTimeout` needs to wait for all the code for queued messages to complete even though you specified a particular time limit for your `setTimeout`.
```javascript
(function() {

  console.log('this is the start');

  setTimeout(function cb() {
    console.log('this is a msg from call back');
  });

  console.log('this is just a message');

  setTimeout(function cb1() {
    console.log('this is a msg from call back1');
  }, 0);

  console.log('this is the end');

})();
```

```javascript
// "this is the start"
// "this is just a message"
// "this is the end"
// note that function return, which is undefined, happens here 
// "this is a msg from call back"
// "this is a msg from call back1"
```

### Several runtimes communicating together
A web worker or a cross-origin iframe has its own stack, heap, and message queue. 
Two distinct runtimes can only communicate through sending messages via the postMessage method. 
This method adds a message to the other runtime if the latter listens to message events.

### Never blocking
A very interesting property of the event loop model is that JavaScript, unlike a lot of other languages, never blocks.
So when the application is waiting for an IndexedDB query to return or an XHR request to return, it can still process other things like user input. 

Legacy exceptions exist like alert or synchronous XHR, but it is considered as a good practice to avoid them.

# Reference
* [__MDN:__ Concurrency model and Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)
* [__Medium:__ Event Loop Horizon — Javascript Event Loop](https://abc.danch.me/event-loop-horizon-javascript-event-loop-637c1932985)
* [__TutsPlus:__ Event-Based Programming: What Async Has Over Sync](https://code.tutsplus.com/tutorials/event-based-programming-what-async-has-over-sync--net-30027)
