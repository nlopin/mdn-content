---
title: Using Web Workers
slug: Web/API/Web_Workers_API/Using_web_workers
tags:
  - Advanced
  - Firefox
  - Guide
  - HTML5
  - JavaScript
  - WebWorkers
  - Workers
---
<div>{{DefaultAPISidebar("Web Workers API")}}</div>

<p>Web Workers are a simple means for web content to run scripts in background threads. The worker thread can perform tasks without interfering with the user interface. In addition, they can perform I/O using <code><a href="/en-US/docs/Web/API/XMLHttpRequest">XMLHttpRequest</a></code> (although the <code>responseXML</code> and <code>channel</code> attributes are always null) or <code><a href="/en-US/docs/Web/API/Fetch_API">fetch</a></code> (with no such restrictions). Once created, a worker can send messages to the JavaScript code that created it by posting messages to an event handler specified by that code (and vice versa).</p>

<p>This article provides a detailed introduction to using web workers.</p>

<h2 id="Web_Workers_API">Web Workers API</h2>

<p>A worker is an object created using a constructor (e.g. {{domxref("Worker.Worker", "Worker()")}}) that runs a named JavaScript file — this file contains the code that will run in the worker thread; workers run in another global context that is different from the current {{domxref("window")}}. Thus, using the {{domxref("window")}} shortcut to get the current global scope (instead of {{domxref("window.self","self")}}) within a {{domxref("Worker")}} will return an error.</p>

<p>The worker context is represented by a {{domxref("DedicatedWorkerGlobalScope")}} object in the case of dedicated workers (standard workers that are utilized by a single script; shared workers use {{domxref("SharedWorkerGlobalScope")}}). A dedicated worker is only accessible from the script that first spawned it, whereas shared workers can be accessed from multiple scripts.</p>

<div class="note">
<p><strong>Note:</strong> See <a href="/en-US/docs/Web/API/Web_Workers_API">The Web Workers API landing page</a> for reference documentation on workers and additional guides.</p>
</div>

<p>You can run whatever code you like inside the worker thread, with some exceptions. For example, you can't directly manipulate the DOM from inside a worker, or use some default methods and properties of the {{domxref("window")}} object. But you can use a large number of items available under <code>window</code>, including <a href="/en-US/docs/Web/API/WebSockets_API">WebSockets</a>, and data storage mechanisms like <a href="/en-US/docs/Web/API/IndexedDB_API">IndexedDB</a>. See <a href="/en-US/docs/Web/API/Web_Workers_API/Functions_and_classes_available_to_workers">Functions and classes available to workers</a> for more details.</p>

<p>Data is sent between workers and the main thread via a system of messages — both sides send their messages using the <code>postMessage()</code> method, and respond to messages via the <code>onmessage</code> event handler (the message is contained within the {{event("Message")}} event's data attribute.) The data is copied rather than shared.</p>

<p>Workers may, in turn, spawn new workers, as long as those workers are hosted within the same origin as the parent page. In addition, workers may use <a href="/en-US/docs/Web/API/XMLHttpRequest"><code>XMLHttpRequest</code></a> for network I/O, with the exception that the <code>responseXML</code> and <code>channel</code> attributes on <code>XMLHttpRequest</code> always return <code>null</code>.</p>

<h2 id="Dedicated_workers">Dedicated workers</h2>

<p>As mentioned above, a dedicated worker is only accessible by the script that called it. In this section we'll discuss the JavaScript found in our <a class="external external-icon" href="https://github.com/mdn/simple-web-worker">Basic dedicated worker example</a> (<a class="external external-icon" href="https://mdn.github.io/simple-web-worker/">run dedicated worker</a>): This allows you to enter two numbers to be multiplied together. The numbers are sent to a dedicated worker, multiplied together, and the result is returned to the page and displayed.</p>

<p>This example is rather trivial, but we decided to keep it simple while introducing you to basic worker concepts. More advanced details are covered later on in the article.</p>

<h3 id="Worker_feature_detection">Worker feature detection</h3>

<p>For slightly more controlled error handling and backwards compatibility, it is a good idea to wrap your worker accessing code in the following (<a href="https://github.com/mdn/simple-web-worker/blob/gh-pages/main.js">main.js</a>):</p>

<pre class="brush: js">if (window.Worker) {

  ...

}</pre>

<h3 id="Spawning_a_dedicated_worker">Spawning a dedicated worker</h3>

<p>Creating a new worker is simple. All you need to do is call the {{domxref("Worker.Worker", "Worker()")}} constructor, specifying the URI of a script to execute in the worker thread (<a href="https://github.com/mdn/simple-web-worker/blob/gh-pages/main.js">main.js</a>):</p>

<pre class="brush: js">var myWorker = new Worker('worker.js');</pre>

<h3 id="Sending_messages_to_and_from_a_dedicated_worker">Sending messages to and from a dedicated worker</h3>

<p>The magic of workers happens via the {{domxref("Worker.postMessage", "postMessage()")}} method and the {{domxref("Worker.onmessage", "onmessage")}} event handler. When you want to send a message to the worker, you post messages to it like this (<a href="https://github.com/mdn/simple-web-worker/blob/gh-pages/main.js">main.js</a>):</p>

<pre class="brush: js">first.onchange = function() {
  myWorker.postMessage([first.value, second.value]);
  console.log('Message posted to worker');
}

second.onchange = function() {
  myWorker.postMessage([first.value, second.value]);
  console.log('Message posted to worker');
}</pre>

<p>So here we have two {{htmlelement("input")}} elements represented by the variables <code>first</code> and <code>second</code>; when the value of either is changed, <code>myWorker.postMessage([first.value,second.value])</code> is used to send the value inside both to the worker, as an array. You can send pretty much anything you like in the message.</p>

<p>In the worker, we can respond when the message is received by writing an event handler block like this (<a href="https://github.com/mdn/simple-web-worker/blob/gh-pages/worker.js">worker.js</a>):</p>

<pre class="brush: js">onmessage = function(e) {
  console.log('Message received from main script');
  var workerResult = 'Result: ' + (e.data[0] * e.data[1]);
  console.log('Posting message back to main script');
  postMessage(workerResult);
}</pre>

<p>The <code>onmessage</code> handler allows us to run some code whenever a message is received, with the message itself being available in the <code>message</code> event's <code>data</code> attribute. Here we multiply together the two numbers then use <code>postMessage()</code> again, to post the result back to the main thread.</p>

<p>Back in the main thread, we use <code>onmessage</code> again, to respond to the message sent back from the worker:</p>

<pre class="brush: js">myWorker.onmessage = function(e) {
  result.textContent = e.data;
  console.log('Message received from worker');
}</pre>

<p>Here we grab the message event data and set it as the <code>textContent</code> of the result paragraph, so the user can see the result of the calculation.</p>

<div class="note"><p><strong>Note:</strong> Notice that <code>onmessage</code> and <code>postMessage()</code> need to be hung off the <code>Worker</code> object when used in the main script thread, but not when used in the worker. This is because, inside the worker, the worker is effectively the global scope.</p></div>

<div class="note"><p><strong>Note:</strong> When a message is passed between the main thread and worker, it is copied or "transferred" (moved), not shared. Read {{anch("Transferring data to and from workers further details", "Transferring data to and from workers: further details")}} for a much more thorough explanation.</p></div>

<h3 id="Terminating_a_worker">Terminating a worker</h3>

<p>If you need to immediately terminate a running worker from the main thread, you can do so by calling the worker's {{domxref("Worker", "terminate")}} method:</p>

<pre class="brush: js">myWorker.terminate();</pre>

<p>The worker thread is killed immediately.</p>

<h3 id="Handling_errors">Handling errors</h3>

<p>When a runtime error occurs in the worker, its <code>onerror</code> event handler is called. It receives an event named <code>error</code> which implements the <code>ErrorEvent</code> interface.</p>

<p>The event doesn't bubble and is cancelable; to prevent the default action from taking place, the worker can call the error event's <a href="/en-US/docs/Web/API/Event/preventDefault"><code>preventDefault()</code></a> method.</p>

<p>The error event has the following three fields that are of interest:</p>

<dl>
 <dt><code>message</code></dt>
 <dd>A human-readable error message.</dd>
 <dt><code>filename</code></dt>
 <dd>The name of the script file in which the error occurred.</dd>
 <dt><code>lineno</code></dt>
 <dd>The line number of the script file on which the error occurred.</dd>
</dl>

<h3 id="Spawning_subworkers">Spawning subworkers</h3>

<p>Workers may spawn more workers if they wish. So-called sub-workers must be hosted within the same origin as the parent page. Also, the URIs for subworkers are resolved relative to the parent worker's location rather than that of the owning page. This makes it easier for workers to keep track of where their dependencies are.</p>

<h3 id="Importing_scripts_and_libraries">Importing scripts and libraries</h3>

<p>Worker threads have access to a global function, <code>importScripts()</code>, which lets them import scripts. It accepts zero or more URIs as parameters to resources to import; all of the following examples are valid:</p>

<pre class="brush: js">importScripts();                         /* imports nothing */
importScripts('foo.js');                 /* imports just "foo.js" */
importScripts('foo.js', 'bar.js');       /* imports two scripts */
importScripts('//example.com/hello.js'); /* You can import scripts from other origins */</pre>

<p>The browser loads each listed script and executes it. Any global objects from each script may then be used by the worker. If the script can't be loaded, <code>NETWORK_ERROR</code> is thrown, and subsequent code will not be executed. Previously executed code (including code deferred using {{domxref("setTimeout()")}}) will still be functional though. Function declarations <strong>after</strong> the <code>importScripts()</code> method are also kept, since these are always evaluated before the rest of the code.</p>

<div class="note"><p><strong>Note:</strong> Scripts may be downloaded in any order, but will be executed in the order in which you pass the filenames into <code>importScripts()</code> . This is done synchronously; <code>importScripts()</code> does not return until all the scripts have been loaded and executed.</p></div>

<h2 id="Shared_workers">Shared workers</h2>

<p>A shared worker is accessible by multiple scripts — even if they are being accessed by different windows, iframes or even workers. In this section we'll discuss the JavaScript found in our <a class="external external-icon" href="https://github.com/mdn/simple-shared-worker">Basic shared worker example</a> (<a class="external external-icon" href="https://mdn.github.io/simple-shared-worker/">run shared worker</a>): This is very similar to the basic dedicated worker example, except that it has two functions available handled by different script files: <em>multiplying two numbers</em>, or <em>squaring a number</em>. Both scripts use the same worker to do the actual calculation required.</p>

<p>Here we'll concentrate on the differences between dedicated and shared workers. Note that in this example we have two HTML pages, each with JavaScript applied that uses the same single worker file.</p>

<div class="note">
<p><strong>Note:</strong> If SharedWorker can be accessed from several browsing contexts, all those browsing contexts must share the exact same origin (same protocol, host, and port).</p>
</div>

<div class="note">
<p><strong>Note:</strong> In Firefox, shared workers cannot be shared between documents loaded in private and non-private windows ({{bug(1177621)}}).</p>
</div>

<h3 id="Spawning_a_shared_worker">Spawning a shared worker</h3>

<p>Spawning a new shared worker is pretty much the same as with a dedicated worker, but with a different constructor name (see <a href="https://github.com/mdn/simple-shared-worker/blob/gh-pages/index.html">index.html</a> and <a href="https://github.com/mdn/simple-shared-worker/blob/gh-pages/index2.html">index2.html</a>) — each one has to spin up the worker using code like the following:</p>

<pre class="brush: js">var myWorker = new SharedWorker('worker.js');</pre>

<p>One big difference is that with a shared worker you have to communicate via a <code>port</code> object — an explicit port is opened that the scripts can use to communicate with the worker (this is done implicitly in the case of dedicated workers).</p>

<p>The port connection needs to be started either implicitly by use of the <code>onmessage</code> event handler or explicitly with the <code>start()</code> method before any messages can be posted. Calling <code>start()</code> is only needed if the <code>message</code> event is wired up via the <code>addEventListener()</code> method.</p>

<div class="note">
<p><strong>Note:</strong> When using the <code>start()</code> method to open the port connection, it needs to be called from both the parent thread and the worker thread if two-way communication is needed.</p>
</div>

<h3 id="Sending_messages_to_and_from_a_shared_worker">Sending messages to and from a shared worker</h3>

<p>Now messages can be sent to the worker as before, but the <code>postMessage()</code> method has to be invoked through the port object (again, you'll see similar constructs in both <a href="https://github.com/mdn/simple-shared-worker/blob/gh-pages/multiply.js">multiply.js</a> and <a href="https://github.com/mdn/simple-shared-worker/blob/gh-pages/square.js">square.js</a>):</p>

<pre class="brush: js">squareNumber.onchange = function() {
  myWorker.port.postMessage([squareNumber.value,squareNumber.value]);
  console.log('Message posted to worker');
}</pre>

<p>Now, on to the worker. There is a bit more complexity here as well (<a href="https://github.com/mdn/simple-shared-worker/blob/gh-pages/worker.js">worker.js</a>):</p>

<pre class="brush: js">onconnect = function(e) {
  var port = e.ports[0];

  port.onmessage = function(e) {
    var workerResult = 'Result: ' + (e.data[0] * e.data[1]);
    port.postMessage(workerResult);
  }
}</pre>

<p>First, we use an <code>onconnect</code> handler to fire code when a connection to the port happens (i.e. when the <code>onmessage</code> event handler in the parent thread is setup, or when the <code>start()</code> method is explicitly called in the parent thread).</p>

<p>We use the <code>ports</code> attribute of this event object to grab the port and store it in a variable.</p>

<p>Next, we add a <code>message</code> handler on the port to do the calculation and return the result to the main thread. Setting up this <code>message</code> handler in the worker thread also implicitly opens the port connection back to the parent thread, so the call to <code>port.start()</code> is not actually needed, as noted above.</p>

<p>Finally, back in the main script, we deal with the message (again, you'll see similar constructs in both <a href="https://github.com/mdn/simple-shared-worker/blob/gh-pages/multiply.js">multiply.js</a> and <a href="https://github.com/mdn/simple-shared-worker/blob/gh-pages/square.js">square.js</a>):</p>

<pre class="brush: js">myWorker.port.onmessage = function(e) {
  result2.textContent = e.data;
  console.log('Message received from worker');
}</pre>

<p>When a message comes back through the port from the worker, we insert the calculation result inside the appropriate result paragraph.</p>

<h2 id="About_thread_safety">About thread safety</h2>

<p>The {{domxref("Worker")}} interface spawns real OS-level threads, and mindful programmers may be concerned that concurrency can cause “interesting” effects in your code if you aren't careful.</p>

<p>However, since web workers have carefully controlled communication points with other threads, it's actually very hard to cause concurrency problems. There's no access to non-threadsafe components or the DOM. And you have to pass specific data in and out of a thread through serialized objects. So you have to work really hard to cause problems in your code.</p>

<h2 id="Content_security_policy">Content security policy</h2>

<p>Workers are considered to have their own execution context, distinct from the document that created them. For this reason they are, in general, not governed by the <a href="/en-US/docs/Mozilla/Add-ons/WebExtensions/Content_Security_Policy">content security policy</a> of the document (or parent worker) that created them. So for example, suppose a document is served with the following header:</p>

<pre class="brush: plain">Content-Security-Policy: script-src 'self'</pre>

<p>Among other things, this will prevent any scripts it includes from using <code><a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/eval">eval()</a></code>. However, if the script constructs a worker, code running in the worker's context <em>will</em> be allowed to use <code>eval()</code>.</p>

<p>To specify a content security policy for the worker, set a <a href="/en-US/docs/Web/HTTP/Headers/Content-Security-Policy">Content-Security-Policy</a> response header for the request which delivered the worker script itself.</p>

<p>The exception to this is if the worker script's origin is a globally unique identifier (for example, if its URL has a scheme of data or blob). In this case, the worker does inherit the CSP of the document or worker that created it.</p>

<h2 id="Transferring_data_to_and_from_workers_further_details">Transferring data to and from workers: further details</h2>

<p>Data passed between the main page and workers is <strong>copied</strong>, not shared. Objects are serialized as they're handed to the worker, and subsequently, de-serialized on the other end. The page and worker <strong>do not share the same instance</strong>, so the end result is that <strong>a duplicate</strong> is created on each end. Most browsers implement this feature as <a href="/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm">structured cloning</a>.</p>

<p>To illustrate this, let's create for didactical purpose a function named <code>emulateMessage()</code>, which will simulate the behavior of a value that is <em>cloned and not shared</em> during the passage from a <code>worker</code> to the main page or vice versa:</p>

<pre class="brush: js">function emulateMessage(vVal) {
    return eval('(' + JSON.stringify(vVal) + ')');
}

// Tests

// test #1
var example1 = new Number(3);
console.log(typeof example1); // object
console.log(typeof emulateMessage(example1)); // number

// test #2
var example2 = true;
console.log(typeof example2); // boolean
console.log(typeof emulateMessage(example2)); // boolean

// test #3
var example3 = new String('Hello World');
console.log(typeof example3); // object
console.log(typeof emulateMessage(example3)); // string

// test #4
var example4 = {
    'name': 'John Smith',
    "age": 43
};
console.log(typeof example4); // object
console.log(typeof emulateMessage(example4)); // object

// test #5
function Animal(sType, nAge) {
    this.type = sType;
    this.age = nAge;
}
var example5 = new Animal('Cat', 3);
alert(example5.constructor); // Animal
alert(emulateMessage(example5).constructor); // Object</pre>

<p>A value that is cloned and not shared is called <em>message</em>. As you will probably know by now, <em>messages</em> can be sent to and from the main thread by using <code>postMessage()</code>, and the <code>message</code> event's {{domxref("MessageEvent.data", "data")}} attribute contains data passed back from the worker.</p>

<p><strong>example.html</strong>: (the main page):</p>

<pre class="brush: js">var myWorker = new Worker('my_task.js');

myWorker.onmessage = function(oEvent) {
  console.log('Worker said : ' + oEvent.data);
};

myWorker.postMessage('ali');</pre>

<p><strong>my_task.js</strong> (the worker):</p>

<pre class="brush: js">postMessage("I\'m working before postMessage(\'ali\').");

onmessage = function(oEvent) {
  postMessage('Hi ' + oEvent.data);
};</pre>

<p>The <a href="/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm">structured cloning</a> algorithm can accept JSON and a few things that JSON can't — like circular references.</p>

<h3 id="Passing_data_examples">Passing data examples</h3>

<h4 id="Example_1_Advanced_passing_JSON_Data_and_creating_a_switching_system">Example #1: Advanced passing JSON Data and creating a switching system</h4>

<p>If you have to pass some complex data and have to call many different functions both on the main page and in the Worker, you can create a system which groups everything together.</p>

<p>First, we create a <code>QueryableWorker</code> class that takes the URL of the worker, a default listener, and an error handler, and this class is going to keep track of a list of listeners and help us communicate with the worker:</p>

<pre class="brush: js">function QueryableWorker(url, defaultListener, onError) {
    var instance = this,
        worker = new Worker(url),
        listeners = {};

    this.defaultListener = defaultListener || function() {};

    if (onError) {worker.onerror = onError;}

    this.postMessage = function(message) {
        worker.postMessage(message);
    }

    this.terminate = function() {
        worker.terminate();
    }
}</pre>

<p>Then we add the methods of adding/removing listeners:</p>

<pre class="brush: js">this.addListeners = function(name, listener) {
    listeners[name] = listener;
}

this.removeListeners = function(name) {
    delete listeners[name];
}</pre>

<p>Here we let the worker handle two simple operations for illustration: getting the difference of two numbers and making an alert after three seconds. In order to achieve that we first implement a <code>sendQuery</code> method which queries if the worker actually has the corresponding methods to do what we want.</p>

<pre class="brush: js">/*
  This functions takes at least one argument, the method name we want to query.
  Then we can pass in the arguments that the method needs.
 */
this.sendQuery = function() {
    if (arguments.length &lt; 1) {
         throw new TypeError('QueryableWorker.sendQuery takes at least one argument');
         return;
    }
    worker.postMessage({
        'queryMethod': arguments[0],
        'queryArguments': Array.prototype.slice.call(arguments, 1)
    });
}</pre>

<p>We finish QueryableWorker with the <code>onmessage</code> method. If the worker has the corresponding methods we queried, it should return the name of the corresponding listener and the arguments it needs, we just need to find it in <code>listeners</code>.:</p>

<pre class="brush: js">worker.onmessage = function(event) {
    if (event.data instanceof Object &amp;&amp;
        event.data.hasOwnProperty('queryMethodListener') &amp;&amp;
        event.data.hasOwnProperty('queryMethodArguments')) {
        listeners[event.data.queryMethodListener].apply(instance, event.data.queryMethodArguments);
    } else {
        this.defaultListener.call(instance, event.data);
    }
}
</pre>

<p>Now onto the worker. First we need to have the methods to handle the two simple operations:</p>

<pre class="brush: js">var queryableFunctions = {
    getDifference: function(a, b) {
        reply('printStuff', a - b);
    },
    waitSomeTime: function() {
        setTimeout(function() {
            reply('doAlert', 3, 'seconds');
        }, 3000);
    }
}

function reply() {
    if (arguments.length &lt; 1) {
        throw new TypeError('reply - takes at least one argument');
        return;
    }
    postMessage({
        queryMethodListener: arguments[0],
        queryMethodArguments: Array.prototype.slice.call(arguments, 1)
    });
}

/* This method is called when main page calls QueryWorker's postMessage method directly*/
function defaultReply(message) {
    // do something
}
</pre>

<p>And the <code>onmessage</code> method is now trivial:</p>

<pre class="brush: js">onmessage = function(event) {
    if (event.data instanceof Object &amp;&amp;
        event.data.hasOwnProperty('queryMethod') &amp;&amp;
        event.data.hasOwnProperty('queryMethodArguments')) {
        queryableFunctions[event.data.queryMethod]
            .apply(self, event.data.queryMethodArguments);
    } else {
        defaultReply(event.data);
    }
}
</pre>

<p>Here are the full implementation:</p>

<p><strong>example.html</strong> (the main page):</p>

<pre class="brush: html">&lt;!doctype html&gt;
  &lt;html&gt;
    &lt;head&gt;
      &lt;meta charset="UTF-8"  /&gt;
      &lt;title&gt;MDN Example - Queryable worker&lt;/title&gt;
    &lt;script type="text/javascript"&gt;
    /*
      QueryableWorker instances methods:
        * sendQuery(queryable function name, argument to pass 1, argument to pass 2, etc. etc): calls a Worker's queryable function
        * postMessage(string or JSON Data): see Worker.prototype.postMessage()
        * terminate(): terminates the Worker
        * addListener(name, function): adds a listener
        * removeListener(name): removes a listener
      QueryableWorker instances properties:
        * defaultListener: the default listener executed only when the Worker calls the postMessage() function directly
     */
    function QueryableWorker(url, defaultListener, onError) {
      var instance = this,
      worker = new Worker(url),
      listeners = {};

      this.defaultListener = defaultListener || function() {};

      if (onError) {worker.onerror = onError;}

      this.postMessage = function(message) {
        worker.postMessage(message);
      }

      this.terminate = function() {
        worker.terminate();
      }

      this.addListener = function(name, listener) {
        listeners[name] = listener;
      }

      this.removeListener = function(name) {
        delete listeners[name];
      }

      /*
        This functions takes at least one argument, the method name we want to query.
        Then we can pass in the arguments that the method needs.
      */
      this.sendQuery = function() {
        if (arguments.length &lt; 1) {
          throw new TypeError('QueryableWorker.sendQuery takes at least one argument');
          return;
        }
        worker.postMessage({
          'queryMethod': arguments[0],
          'queryMethodArguments': Array.prototype.slice.call(arguments, 1)
        });
      }

      worker.onmessage = function(event) {
        if (event.data instanceof Object &amp;&amp;
          event.data.hasOwnProperty('queryMethodListener') &amp;&amp;
          event.data.hasOwnProperty('queryMethodArguments')) {
          listeners[event.data.queryMethodListener].apply(instance, event.data.queryMethodArguments);
        } else {
          this.defaultListener.call(instance, event.data);
        }
      }
    }

    // your custom "queryable" worker
    var myTask = new QueryableWorker('my_task.js');

    // your custom "listeners"
    myTask.addListener('printStuff', function (result) {
      document.getElementById('firstLink').parentNode.appendChild(document.createTextNode('The difference is ' + result + '!'));
    });

    myTask.addListener('doAlert', function (time, unit) {
      alert('Worker waited for ' + time + ' ' + unit + ' :-)');
    });
&lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
  &lt;ul&gt;
    &lt;li&gt;&lt;a id="firstLink" href="javascript:myTask.sendQuery('getDifference', 5, 3);"&gt;What is the difference between 5 and 3?&lt;/a&gt;&lt;/li&gt;
    &lt;li&gt;&lt;a href="javascript:myTask.sendQuery('waitSomeTime');"&gt;Wait 3 seconds&lt;/a&gt;&lt;/li&gt;
    &lt;li&gt;&lt;a href="javascript:myTask.terminate();"&gt;terminate() the Worker&lt;/a&gt;&lt;/li&gt;
  &lt;/ul&gt;
&lt;/body&gt;
&lt;/html&gt;</pre>

<p><strong>my_task.js</strong> (the worker):</p>

<pre class="brush: js">var queryableFunctions = {
  // example #1: get the difference between two numbers:
  getDifference: function(nMinuend, nSubtrahend) {
      reply('printStuff', nMinuend - nSubtrahend);
  },
  // example #2: wait three seconds
  waitSomeTime: function() {
      setTimeout(function() { reply('doAlert', 3, 'seconds'); }, 3000);
  }
};

// system functions

function defaultReply(message) {
  // your default PUBLIC function executed only when main page calls the queryableWorker.postMessage() method directly
  // do something
}

function reply() {
  if (arguments.length &lt; 1) { throw new TypeError('reply - not enough arguments'); return; }
  postMessage({ 'queryMethodListener': arguments[0], 'queryMethodArguments': Array.prototype.slice.call(arguments, 1) });
}

onmessage = function(oEvent) {
  if (oEvent.data instanceof Object &amp;&amp; oEvent.data.hasOwnProperty('queryMethod') &amp;&amp; oEvent.data.hasOwnProperty('queryMethodArguments')) {
    queryableFunctions[oEvent.data.queryMethod].apply(self, oEvent.data.queryMethodArguments);
  } else {
    defaultReply(oEvent.data);
  }
};</pre>

<p>It is possible to switch the content of each mainpage -&gt; worker and worker -&gt; mainpage message. And the property names "queryMethod", "queryMethodListeners", "queryMethodArguments" can be anything as long as they are consistent in <code>QueryableWorker</code> and the <code>worker</code>.</p>

<h3 id="Passing_data_by_transferring_ownership_transferable_objects">Passing data by transferring ownership (transferable objects)</h3>

<p>Google Chrome 17+ and Firefox 18+ contain an additional way to pass certain types of objects (transferable objects, that is objects implementing the {{domxref("Transferable")}} interface) to or from a worker with high performance. Transferable objects are transferred from one context to another with a zero-copy operation, which results in a vast performance improvement when sending large data sets. Think of it as pass-by-reference if you're from the C/C++ world. However, unlike pass-by-reference, the 'version' from the calling context is no longer available once transferred. Its ownership is transferred to the new context. For example, when transferring an {{jsxref("ArrayBuffer")}} from your main app to a worker script, the original {{jsxref("ArrayBuffer")}} is cleared and no longer usable. Its content is (quite literally) transferred to the worker context.</p>

<pre class="brush: js">// Create a 32MB "file" and fill it.
var uInt8Array = new Uint8Array(1024 * 1024 * 32); // 32MB
for (var i = 0; i &lt; uInt8Array.length; ++i) {
  uInt8Array[i] = i;
}

worker.postMessage(uInt8Array.buffer, [uInt8Array.buffer]);
</pre>

<div class="note">
<p><strong>Note:</strong> For more information on transferable objects, performance, and feature-detection for this method, read <a href="https://updates.html5rocks.com/2011/12/Transferable-Objects-Lightning-Fast">Transferable Objects: Lightning Fast!</a> on HTML5 Rocks.</p>
</div>

<h2 id="Embedded_workers">Embedded workers</h2>

<p>There is not an "official" way to embed the code of a worker within a web page, like {{HTMLElement("script")}} elements do for normal scripts. But a {{HTMLElement("script")}} element that does not have a <code>src</code> attribute and has a <code>type</code> attribute that does not identify an executable MIME type can be considered a data block element that JavaScript could use. "Data blocks" is a more general feature of HTML5 that can carry almost any textual data. So, a worker could be embedded in this way:</p>

<pre class="brush: html">&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;
&lt;meta charset="UTF-8" /&gt;
&lt;title&gt;MDN Example - Embedded worker&lt;/title&gt;
&lt;script type="text/js-worker"&gt;
  // This script WON'T be parsed by JS engines because its MIME type is text/js-worker.
  var myVar = 'Hello World!';
  // Rest of your worker code goes here.
&lt;/script&gt;
&lt;script type="text/javascript"&gt;
  // This script WILL be parsed by JS engines because its MIME type is text/javascript.
  function pageLog(sMsg) {
    // Use a fragment: browser will only render/reflow once.
    var oFragm = document.createDocumentFragment();
    oFragm.appendChild(document.createTextNode(sMsg));
    oFragm.appendChild(document.createElement('br'));
    document.querySelector('#logDisplay').appendChild(oFragm);
  }
&lt;/script&gt;
&lt;script type="text/js-worker"&gt;
  // This script WON'T be parsed by JS engines because its MIME type is text/js-worker.
  onmessage = function(oEvent) {
    postMessage(myVar);
  };
  // Rest of your worker code goes here.
&lt;/script&gt;
&lt;script type="text/javascript"&gt;
  // This script WILL be parsed by JS engines because its MIME type is text/javascript.

  // In the past...:
  // blob builder existed
  // ...but now we use Blob...:
  var blob = new Blob(Array.prototype.map.call(document.querySelectorAll('script[type=\'text\/js-worker\']'), function (oScript) { return oScript.textContent; }),{type: 'text/javascript'});

  // Creating a new document.worker property containing all our "text/js-worker" scripts.
  document.worker = new Worker(window.URL.createObjectURL(blob));

  document.worker.onmessage = function(oEvent) {
    pageLog('Received: ' + oEvent.data);
  };

  // Start the worker.
  window.onload = function() { document.worker.postMessage(''); };
&lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;&lt;div id="logDisplay"&gt;&lt;/div&gt;&lt;/body&gt;
&lt;/html&gt;</pre>

<p>The embedded worker is now nested into a new custom <code>document.worker</code> property.</p>

<p>It is also worth noting that you can also convert a function into a Blob, then generate an object URL from that blob. For example:</p>

<pre class="brush: js">function fn2workerURL(fn) {
  var blob = new Blob(['('+fn.toString()+')()'], {type: 'text/javascript'})
  return URL.createObjectURL(blob)
}</pre>

<h2 id="Further_examples">Further examples</h2>

<p>This section provides further examples of how to use web workers.</p>

<h3 id="Performing_computations_in_the_background">Performing computations in the background</h3>

<p>Workers are mainly useful for allowing your code to perform processor-intensive calculations without blocking the user interface thread. In this example, a worker is used to calculate Fibonacci numbers.</p>

<h4 id="The_JavaScript_code">The JavaScript code</h4>

<p>The following JavaScript code is stored in the "fibonacci.js" file referenced by the HTML in the next section.</p>

<pre class="brush: js">var results = [];

function resultReceiver(event) {
  results.push(parseInt(event.data));
  if (results.length == 2) {
    postMessage(results[0] + results[1]);
  }
}

function errorReceiver(event) {
  throw event.data;
}

onmessage = function(event) {
  var n = parseInt(event.data);

  if (n == 0 || n == 1) {
    postMessage(n);
    return;
  }

  for (var i = 1; i &lt;= 2; i++) {
    var worker = new Worker('fibonacci.js');
    worker.onmessage = resultReceiver;
    worker.onerror = errorReceiver;
    worker.postMessage(n - i);
  }
 };</pre>

<p>The worker sets the property <code>onmessage</code> to a function which will receive messages sent when the worker object's <code>postMessage()</code> is called (note that this differs from defining a global <em>variable</em> of that name, or defining a <em>function</em> with that name. <code>var onmessage</code> and <code>function onmessage</code> will define global properties with those names, but they will not register the function to receive messages sent by the web page that created the worker). This starts the recursion, spawning new copies of itself to handle each iteration of the calculation.</p>

<h4 id="The_HTML_code">The HTML code</h4>

<pre class="brush: html">&lt;!DOCTYPE html&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;meta charset="UTF-8"  /&gt;
    &lt;title&gt;Test threads fibonacci&lt;/title&gt;
  &lt;/head&gt;
  &lt;body&gt;

  &lt;div id="result"&gt;&lt;/div&gt;

  &lt;script language="javascript"&gt;

    var worker = new Worker('fibonacci.js');

    worker.onmessage = function(event) {
      document.getElementById('result').textContent = event.data;
      console.log('Got: ' + event.data + '\n');
    };

    worker.onerror = function(error) {
      console.log('Worker error: ' + error.message + '\n');
      throw error;
    };

    worker.postMessage('5');

  &lt;/script&gt;
  &lt;/body&gt;
&lt;/html&gt;
</pre>

<p>The web page creates a <code>div</code> element with the ID <code>result</code> , which gets used to display the result, then spawns the worker. After spawning the worker, the <code>onmessage</code> handler is configured to display the results by setting the contents of the <code>div</code> element, and the <code>onerror</code> handler is set to log the error message to the devtools console.</p>

<p>Finally, a message is sent to the worker to start it.</p>

<p><a href="https://mdn.github.io/fibonacci-worker/">Try this example live</a>.</p>

<h3 id="Dividing_tasks_among_multiple_workers">Dividing tasks among multiple workers</h3>

<p>As multi-core computers become increasingly common, it's often useful to divide computationally complex tasks among multiple workers, which may then perform those tasks on multiple-processor cores.</p>

<h2 id="Other_types_of_worker">Other types of worker</h2>

<p>In addition to dedicated and shared web workers, there are other types of worker available:</p>

<ul>
 <li><a href="/en-US/docs/Web/API/Service_Worker_API">ServiceWorkers</a> essentially act as proxy servers that sit between web applications, and the browser and network (when available). They are intended to (amongst other things) enable the creation of effective offline experiences, intercepting network requests and taking appropriate action based on whether the network is available and updated assets reside on the server. They will also allow access to push notifications and background sync APIs.</li>

 <li><a href="/en-US/docs/Web/API/Web_Audio_API#audio_processing_in_javascript">Audio Worklet</a> provide the ability for direct scripted audio processing to be done in a worklet (a lightweight version of worker) context.</li>
</ul>

<h2 id="Debugging_worker_threads">Debugging worker threads</h2>

<p>Most browsers support debugging of worker threads in their JavaScript debuggers in <em>exactly the same way</em> as debugging the main thread! For example, both Firefox and Chrome list JavaScript source files for both the main thread and active worker threads, and all of these files can be opened to set breakpoints and logpoints.</p>

<p>The screenshot below shows this on Firefox. The <em>sources list</em> shows <code>worker.js</code> running in a separate worker thread. When selected this file is opened in the <a href="/en-US/docs/Tools/Debugger/UI_Tour#source_pane">source pane</a>, just like code running in the main thread.</p>

<p><img src="worker-source.png"></p>

<div class="notecard note">
<p><strong>Note:</strong> Worker scripts are loaded when needed, and hence may not be present in the sources list when a page is first loaded.</p>
</div>

<p>In the source pane you can <a href="/en-US/docs/Tools/Debugger/How_to/Set_a_breakpoint">set a breakpoint</a> (or <a href="/en-US/docs/Tools/Debugger/Set_a_logpoint">logpoint</a>) in a worker thread in the normal way. When execution is paused, the context of the debugger is updated to show correct <a href="/en-US/docs/Tools/Debugger/How_to/Set_a_breakpoint">breakpoints</a>, <a href="/en-US/docs/Tools/Debugger/How_to/Set_a_breakpoint#inline_variable_preview">inline variable preview</a>, <a href="/en-US/docs/Tools/Debugger/UI_Tour#call_stack">call stack</a>, etc., just as you'd expect.</p>

<p><img src="worker-breakpoints-callstack.png"></p>

<div class="notecard note">
<p><strong>Note:</strong> For more information see <a href="/en-US/docs/Tools/Debugger">Firefox JavaScript Debugger</a>.</p>
</div>

<h2 id="Functions_and_interfaces_available_in_workers">Functions and interfaces available in workers</h2>

<p>You can use most standard JavaScript features inside a web worker, including:</p>

<ul>
 <li>{{domxref("Navigator")}}</li>
 <li>{{domxref("XMLHttpRequest")}}</li>
 <li>{{jsxref("Global_Objects/Array", "Array")}}, {{jsxref("Global_Objects/Date", "Date")}}, {{jsxref("Global_Objects/Math", "Math")}}, and {{jsxref("Global_Objects/String", "String")}}</li>
 <li>{{domxref("setTimeout()")}} and {{domxref("setInterval()")}}</li>
</ul>

<p>The main thing you <em>can't</em> do in a Worker is directly affect the parent page. This includes manipulating the DOM and using that page's objects. You have to do it indirectly, by sending a message back to the main script via {{domxref("DedicatedWorkerGlobalScope.postMessage")}}, then actioning the changes from there.</p>

<div class="notecard note">
<p><strong>Note:</strong> You can test whether a method is available to workers using the site: <a href="https://worker-playground.glitch.me/">https://worker-playground.glitch.me/</a> . For example, if you enter <a href="/en-US/docs/Web/API/EventSource">EventSource</a> into the site on Firefox 84 you'll see that this is not supported in service workers, but is in dedicated and shared workers.</p>
</div>

<div class="note">
<p><strong>Note:</strong> For a complete list of functions available to workers, see <a href="/en-US/docs/Web/API/Web_Workers_API/Functions_and_classes_available_to_workers">Functions and interfaces available to workers</a>.</p>
</div>

<h2 id="Specifications">Specifications</h2>

<table class="standard-table">
 <tbody>
  <tr>
   <th scope="col">Specification</th>
   <th scope="col">Status</th>
   <th scope="col">Comment</th>
  </tr>
  <tr>
   <td>{{SpecName('HTML WHATWG', '#workers', 'Web workers')}}</td>
   <td>{{Spec2('HTML WHATWG')}}</td>
   <td></td>
  </tr>
 </tbody>
</table>

<h2 id="See_also">See also</h2>

<ul>
 <li><code><a href="/en-US/docs/Web/API/Worker">Worker</a></code> interface</li>
 <li><code><a href="/en-US/docs/Web/API/SharedWorker">SharedWorker</a></code> interface</li>
 <li><a href="/en-US/docs/Web/API/Web_Workers_API/Functions_and_classes_available_to_workers">Functions available to workers</a></li>
</ul>