# Long running operation : how to handle it?

By default the browser uses a single thread to run all the JavaScript in your page as well as to perform layout, 
reflows, and garbage collection. This means that long-running JavaScript functions can block the thread, 
leading to an unresponsive page and a bad user experience.

[An example site](http://mdn.github.io/performance-scenarios/js-worker/index.html) whose long-running JavaScript causes responsiveness problems. 
We can apply two different approaches to fixing them. The first is to split long-running functions 
into pieces and use `requestAnimationFrame` to schedule each piece, 
and the second is to run the whole function in a separate thread using a `web worker`.

__`Using requestAnimationFrame`__

In the first attempt at fixing this, we'll split up the function into a number of much smaller self-contained functions, 
and schedule each one using `requestAnimationFrame()`.

`requestAnimationFrame()` tells the browser to run the given function in each frame, just before it performs a repaint. 
As long as each function is reasonably small, the browser should be able to keep inside its frame budget.

![Using requestAnimationFrame](https://mdn.mozillademos.org/files/10897/perf-js-raf-overview.png)

Each function should be short enough that the browser can handle it without the overall frame rate dropping.

Using requestAnimationFrame worked to solve the responsiveness problem here, 
but there are a couple of potential problems with it:

* it can be difficult to split up a long-running function into separate self-contained functions. 
Even this very simple case produced more complex code.
* the split-up version takes much longer to run. In fact we can be quite precise about how long it takes: 
there are 50 iterations, and the browser is producing about 60 frames per second. So it will take almost 
a second to run all the computations, and that's visible in both the user experience and in the profile.

__`Using web workers`__

Web workers enable you to run JavaScript in a separate thread. 
The main thread and the worker thread can't call each other directly, but communicate using an asynchronous messaging API.

```javascript
const iterations = 50;
const multiplier = 1000000000;

var worker = new Worker("js/calculate.js");

function doPointlessComputationsInWorker() {

  function handleWorkerCompletion(message) {
    if (message.data.command == "done") {
      pointlessComputationsButton.disabled = false;
      console.log(message.data.primes);
      worker.removeEventListener("message", handleWorkerCompletion);
    }
  }

  worker.addEventListener("message", handleWorkerCompletion, false);

  worker.postMessage({
    "multiplier": multiplier,
    "iterations": iterations
  });
}
```
The main difference here, compared with the original, is that we need to:

* create a worker
* send it a message when we are ready to calculate
* listen for a message called "done", which indicates that the worker has finished.

Then we need a new file `"calculate.js"`, that looks like this:

```javascript
self.addEventListener("message", go);

function go(message) {
  var iterations = message.data.iterations;
  var multiplier = message.data.multiplier;
  primes = calculatePrimes(iterations, multiplier);

  self.postMessage({
    "command":"done",
    "primes": primes
  });
}

function calculatePrimes(iterations, multiplier) {
 //do something very long
  return primes;
}
```
In the worker, we have to listen for a message telling us to start, and send a "done" message back when we are done.

![Using web workers](https://mdn.mozillademos.org/files/10899/perf-js-worker-overview.png)

Each button-press is visible in the overview as two very short orange markers:

1. the `doPointlessComputationsInWorker()` function that handles the click event and starts the worker's processing
2. the `handleWorkerCompletion()` function that runs when the worker calls "done".

In between, the worker runs all the primality tests, and it doesn't seem to have any effect at all 
on the responsiveness of the main thread. This might seem unlikely, but because workers run in a separate thread, 
they can take advantage of multi-core processors, which a single-threaded web site can't.

**_The main limitation of web workers is that DOM APIs are not available to code running in a worker._**

# Reference
* [__MDN:__ Intensive JavaScript](https://developer.mozilla.org/en-US/docs/Tools/Performance/Scenarios/Intensive_JavaScript)
* [Updating DOM during long running operation](http://blogs.microsoft.co.il/oric/2015/03/21/updating-dom-during-long-running-operation)
