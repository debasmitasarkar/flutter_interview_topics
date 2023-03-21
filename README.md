# Flutter Interview Questions
These questions are not sorted and listed randomly. Most of these questions were encountered in various interviews.
I plan to make this a segmented guide with links and answers to questions, so that, before any Flutter interview, you take a glance or prepare throughly just these topics to ace the interview. Any PRs to make this better are welcome.

### 1. is Dart multithreaded? If not how does it process Future calls?`

  Dart, the programming language used in Flutter, is single-threaded, meaning it runs on a single thread of execution by default. However, Dart provides support for   asynchronous programming through the use of Future, async, and await keywords, as well as the Isolate API, which allows you to perform parallel computations.

  When you use Future in Dart, you're actually working with asynchronous programming constructs. Futures represent a potential value or an error that will be         available at some point in the future. They allow you to write non-blocking code that can handle time-consuming operations, such as network requests or file I/O,   without freezing the UI.

 When you make a Future call, Dart's event loop schedules the operation to run in the background. Once the operation is complete, the event loop processes the        result and either resolves the Future with a value or rejects it with an error. During this time, the main thread can continue executing other tasks.

  If you need true parallelism, you can use Dart's Isolate API. Isolates are separate execution threads that run in parallel with the main thread. Each isolate has   its own memory heap, which ensures that no memory is shared between isolates. This allows you to run CPU-intensive tasks without blocking the main thread.

In summary, while Dart itself is single-threaded, it provides mechanisms for asynchronous programming with Future, async, and await, as well as parallelism with the Isolate API.

### 2. What is an isolate ?

An isolate in Dart is a separate, independent execution unit that runs in parallel with the main thread. Isolates allow you to perform concurrent programming in Dart, which is particularly useful for running CPU-intensive tasks without blocking the main thread or freezing the user interface.

Each isolate has its own memory heap, which means it doesn't share memory with the main thread or other isolates. This helps to prevent common concurrency issues, such as race conditions or deadlocks, that can arise when multiple threads access shared memory.

Isolates communicate with each other through message passing, typically using SendPort and ReceivePort. When you want to send data between isolates, you need to ensure that the data is either primitive (e.g., numbers, booleans, strings) or can be serialized and deserialized, since the data is passed by value, not by reference.

Here's a simple example of how to create an isolate and send a message between isolates in Dart:

```
import 'dart:async';
import 'dart:isolate';

void isolateFunction(SendPort sendPort) {
  sendPort.send('Hello from the isolate!');
}

Future<void> main() async {
  // Create a receive port for the main isolate to receive messages from the spawned isolate
  ReceivePort receivePort = ReceivePort();

  // Spawn a new isolate and pass the send port of the main isolate to it
  await Isolate.spawn(isolateFunction, receivePort.sendPort);

  // Listen for a message from the spawned isolate
  receivePort.listen((message) {
    print('Received message: $message');
  });
}
```

In this example, we create a new isolate by spawning the isolateFunction and passing it the send port of the main isolate. The spawned isolate sends a message to the main isolate, which listens for messages on its receive port and prints the received message.

### 3. How does isolate work?

Isolates work in Dart by providing an isolated execution environment that runs in parallel with the main thread, allowing you to perform concurrent programming. Each isolate runs in its own memory heap and has its own event queue and event loop, enabling them to execute tasks independently without sharing memory or state.

Here's a step-by-step explanation of how isolates work in Dart:

**Creation**: You create a new isolate using the `Isolate.spawn()` function, passing it a top-level function or a static method as an entry point, along with an initial message, typically a `SendPort` to establish communication between the isolates.

Message Passing: Since isolates don't share memory, they communicate using message passing. To send and receive messages, you use `SendPort` and `ReceivePort` objects. A `SendPort` is used to send messages to a receiving isolate, while a ReceivePort is used to listen for incoming messages. Messages are passed by value, meaning a copy of the data is sent, not a reference to the original data.

**Execution**: Each isolate has its own event loop and event queue. When an isolate receives a message, the message is added to the event queue. The event loop processes messages in the queue one by one, executing the associated task or function. The isolate continues processing messages until the event queue is empty or the isolate is terminated.

**Termination**: An isolate can be terminated either programmatically or when it finishes executing all the tasks in its event queue. To programmatically terminate an isolate, you can use the `Isolate.kill()` method or send a specific message to the isolate, indicating it should terminate itself.

In summary, isolates work in Dart by providing a separate execution environment for parallel computation, with their own memory heap, event loop, and event queue. They communicate with other isolates and the main thread through message passing, enabling concurrent programming without the complexities of shared memory and synchronization.


4. How does isolates talks to each other?
5. What is an event loop? What are micro tasks?
6. How does obfuscation work in Flutter? What's the need for it?
7. Const vs final
8. Difference between Dev dependencies vs regular dependency
9. Should we declare a variable to allocate size from media query in build method? why?
10. How to check if any widget is placed in widget tree?
11. Difference between mounted vs past-frame callback
12. When and why should we use widget binding observer?
13. What is InheritedWidget? Why/when do we use it? How to pass data through InheritedWidget and access it from other widgets?
14. AOT vs JIT compiler? In Flutter which compiler gets used in which cases?
15. AOT vs JIT combiler advantages and disadvantages
16. BehaviourSubject and RxDart 
17. Inherited widget vs Provider which is is better and why?
18. what is vsync?
19. what are mixins? Give an example of mixin.
20. Do mixins solve the diamond problem?
21. What is tree shaking ? Disadvantages of tree shaking
22. Dependency Injection and it's disadvantages
23. How to overcome problems in low latency network? what should be taken care in this case?
24. What are the drawbacks of Singleton network?
25. SendPort.send() vs Isolate.exit() , what's the difference?
30. Immulibility related questions in Flutter
31. SizedBox vs Container
32. Should we do Future calls inside build method? If not why?
33. factory constructor vs const constructor
34. What is BuildContext?
35. How does build method work? behind the scene what happens?
36. How to ensure security in mobile apps?
37. What is man in the middle attack? how to prevent that?






