Node js is a runtime environment which works well with javascript.

####Libuv :
    Libuv is an open-source library that handles the thread-pool, doing signaling, inter process communications all other magic needed to make the asynchronous
    Libuv implements fully featured event loop.Originally developed as an abstraction to node js .But currently it also can :
    TCP and UDP sockets of the net package
    Async DNS resolutions
    Async file and file system operations (like the one we're doing here)
    File System events
    IPC
    Child processes and shell control


    JS engines actually do not run isolated from everything. They run inside what is called a hosting environment. This environment can be whatever place JS is running into, like a browser, Node.js or, since JS is pretty much everywhere, can be a toaster or a plane. Every environment is different from each other, every one has their own skills and abilities, but they all have an event loop.

    The event loop is what actually takes care of asynchronous code execution for JS Engines, at least of the scheduling part. It is the one who calls the engine and send the commands to be executed, and also is the one who queues response callbacks which the engine returns to be called afterwards. So we're beginning to comprehend that a JS Engine is nothing more than an on-demand execution environment for any JS code, working or not. All that surrounds it, the environment, the event loop, is responsible for scheduling the JS code executions, which are called events.


    Web APIs are, in essence, threads that we cannot access as developers, we can only make calls to them. Generally these are pieces that are built into the environment itself, for instance, in a browser environment, these would be APIs like document, XMLHttpRequest or setTimeout, which are mostly async functions. In Node.js these would be our C++ APIs we saw in the first part of the guide.


    So, in plain words, whenever we call a function like setTimeout on Node.js, this call is sent to a different thread. All of this is controlled and provided by libuv, including the APIs we're using.

    Let's take a simpler example to show how the event loop actually works:
    console.log('Node.js')
    setTimeout(function cb() { console.log(' awesome!') }, 5000)
    console.log(' is')

    ___________      ____________   ________
    |          |     |          |   |       |
    |          |     |          |   |       |
    | Browser  |     | Callstack|   | web   |
    |          |     |          |   | API   |
    |__________|     |__________|   |_______|
                          |
                          |          ___________________
                          |          |  call back QUEUE |
                         (⭕️)        |__________________|
                        event loop

    The event loop has a single task to do: Monitor the call stack and what is called the callback queue. Once the call stack is empty, it'll take the first event from the callback queue and push it into the call stack, which effectively runs it. To this iteration, taking a callback from the queue and executing it into the call stack, we give the name of tick.
    

    Since there are no restrictions of what a Microtask can do to your code, it's possible for a Microtask to add another Microtask in the end of the same queue endlessly, causing what is called a "Microtask loop", which starves the program of the needed resources and prevent it from moving on the the next tick. This is the equivalent of having a while(true) loop running in your code, but asynchronously.

    In order to prevent such starvation, the engine has built-in protection called process.maxTickDepth, which is set to the value of 1000, after 1000 microtasks have been scheduled and ran in the same tick, then the next macrotask is run.
