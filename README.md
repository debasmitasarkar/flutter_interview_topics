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


### 4. How does isolates talks to each other?

Isolates in Dart communicate with each other through message passing, using SendPort and ReceivePort objects. Since isolates don't share memory, they can't directly access each other's variables or objects. Instead, they send messages containing data between them, allowing them to exchange information or coordinate tasks.

Here's a brief overview of how isolates talk to each other in Dart:

**SendPort and ReceivePort**: To facilitate communication between isolates, each isolate has a SendPort for sending messages and a ReceivePort for receiving messages. When an isolate wants to send a message to another isolate, it uses the target isolate's SendPort. The target isolate, in turn, listens for messages on its ReceivePort.

**Message Passing**: Messages are passed by value, meaning a copy of the data is sent, not a reference to the original data. This ensures that the isolates remain isolated and don't share memory. You can send primitive data types (numbers, strings, booleans) as well as serializable objects between isolates.

**Listening for Messages**: To receive messages from other isolates, you need to listen to the ReceivePort. You can use the listen method on the ReceivePort object and provide a callback function to handle incoming messages. When a message is received, the callback function is executed with the message as an argument.

Here's a simple example demonstrating how two isolates can communicate in Dart:

```
import 'dart:async';
import 'dart:isolate';

// Function to be executed in the spawned isolate
void isolateFunction(SendPort mainIsolateSendPort) {
  // Create a receive port for the spawned isolate
  ReceivePort spawnedIsolateReceivePort = ReceivePort();

  // Send the send port of the spawned isolate to the main isolate
  mainIsolateSendPort.send(spawnedIsolateReceivePort.sendPort);

  // Listen for messages from the main isolate
  spawnedIsolateReceivePort.listen((message) {
    print('Spawned isolate received: $message');
  });
}

Future<void> main() async {
  // Create a receive port for the main isolate
  ReceivePort mainIsolateReceivePort = ReceivePort();

  // Spawn a new isolate and pass the send port of the main isolate
  await Isolate.spawn(isolateFunction, mainIsolateReceivePort.sendPort);

  // Listen for the send port of the spawned isolate
  SendPort spawnedIsolateSendPort = await mainIsolateReceivePort.first;

  // Send a message to the spawned isolate
  spawnedIsolateSendPort.send('Hello from the main isolate!');
}
```

### 5. What is an event loop? What are micro tasks?

An event loop is a programming construct that continuously processes and executes tasks or events from a queue in a single-threaded, non-blocking manner. In Dart, both the main isolate and other isolates have their own event loops. The event loop's primary function is to manage the execution of tasks, such as event handling, I/O operations, and timers, allowing asynchronous programming without blocking the thread.

An event loop generally consists of the following components:

**Task Queue**: A queue that stores tasks or events to be processed. When a new task is scheduled, it is added to the task queue.

**Microtask Queue**: A separate queue that holds microtasks, which are small, short-lived tasks that need to be executed before the event loop processes the next task in the task queue. Microtasks are typically generated as a result of scheduling callbacks using `Future` or `Promise` objects.

**Event Loop Cycle**: The event loop repeatedly processes tasks from the task queue and microtasks from the microtask queue. In each iteration of the loop, it first checks if there are any microtasks in the microtask queue. If there are, the event loop processes all the microtasks in the queue before moving on to the task queue. Once the microtask queue is empty, the event loop processes the next task in the task queue.

Microtasks are small, quick tasks that are executed in between the processing of regular tasks in the event loop. They are typically used for operations that need to be completed before the event loop continues processing other tasks. In Dart, you can create a microtask using the `scheduleMicrotask` function, or they can be generated implicitly when you use `async` functions and `Future` objects.

The primary difference between tasks and microtasks is their priority in the event loop. Microtasks have a higher priority and are executed before the event loop moves on to the next task in the task queue. This ensures that microtasks are executed as soon as possible, allowing you to efficiently handle quick, short-lived operations that should be completed before the event loop processes other tasks.

In summary, the event loop is a core concept in asynchronous programming that allows tasks to be executed in a non-blocking manner. Microtasks are short-lived tasks with higher priority than regular tasks, ensuring they are executed quickly before the event loop moves on to other tasks in the queue.


### 6. How does obfuscation work in Flutter? What's the need for it?

Obfuscation in Flutter is a process that transforms your app's Dart code into an equivalent, but harder-to-understand version, by replacing meaningful names of classes, methods, and variables with shorter, less descriptive names (such as random characters). This is done to make it more difficult for others to reverse-engineer or analyze your app's source code, protecting your intellectual property and making it harder for potential attackers to identify vulnerabilities.

To enable obfuscation in Flutter, you need to pass certain flags when building your app in release mode. For example, when building an Android app with Flutter, you would use the following command:

```flutter build apk --obfuscate --split-debug-info=<output-directory>```

For an iOS app, the command would be:

```flutter build ios --obfuscate --split-debug-info=<output-directory>```

These flags tell the Dart compiler to obfuscate the code and to store the debugging information separately in the specified output directory. The --split-debug-info flag is necessary because obfuscation makes debugging more difficult, so storing the debug information separately allows you to debug your app if needed while keeping the release binary obfuscated.

The need for obfuscation in Flutter (or any other app development framework) stems from the following reasons:

**Protection of Intellectual Property**: Obfuscation helps protect your proprietary algorithms, business logic, or other trade secrets from being easily understood by competitors or malicious actors who may gain access to your app's compiled code.

**Security**: By making the app's code harder to understand, obfuscation can make it more difficult for attackers to analyze the code, identify vulnerabilities, and develop exploits.

**Tampering Prevention**: Obfuscation can make it harder for attackers to modify your app's code for malicious purposes, such as injecting malware or bypassing licensing checks.

It's important to note that obfuscation is not a foolproof method for protecting your app, as determined attackers can still reverse-engineer obfuscated code using advanced tools and techniques. However, it does increase the effort required to understand your app's inner workings and can act as an additional layer of security alongside other best practices.


### 7. What is the difference between Const vs final ?

In Dart, `const` and `final` are both used to create variables with values that cannot be changed after they are assigned. However, there are some differences in their usage and behavior:

**1. const (Compile-Time Constant)**: When you declare a variable as const, it means that the variable is a compile-time constant. The value of a const variable must be determined at compile time, and it cannot be changed after compilation. const variables are implicitly final, meaning they cannot be reassigned.

Example:
```const int myConstValue = 42;```

Here, `myConstValue` is a compile-time constant with a value of 42, which cannot be changed after compilation.

**2.final (Run-Time Constant)**: When you declare a variable as `final`, it means that the variable is a run-time constant. The value of a `final` variable can be determined at runtime, but it can only be assigned once. After the initial assignment, the value of a `final` variable cannot be changed.

Example:

```final int myFinalValue = calculateValue();```

In this example, myFinalValue is a run-time constant that gets its value from the calculateValue() function. The value of myFinalValue cannot be changed after it has been assigned.

Here's a summary of the differences between const and final:

- const variables must have their values assigned at compile time, while final variables can have their values assigned at runtime.
- const variables are implicitly final, meaning they cannot be reassigned, while final variables can only be assigned once.
- const variables can be used in places where the value must be known at compile time, such as when defining the keys of a Map, while final variables can be used in cases where the value can be determined at runtime.

It's important to choose the appropriate keyword based on your requirements. If you need a constant value that must be known at compile time, use const. If you need a constant value that can be determined at runtime, use final.



### 8. Difference between Dev dependencies vs regular dependency

In a Dart or Flutter project, dependencies are libraries or packages that your project relies on for additional functionality. They are defined in the pubspec.yaml file, and there are two types of dependencies: regular dependencies (also called "dependencies") and development dependencies (or "dev_dependencies"). Here are the main differences between the two:

**1. Regular Dependencies (dependencies)**: These are the packages or libraries that your project needs to run in both development and production environments. They are essential for the core functionality of your app, and they will be included when you build or compile your project for release. Regular dependencies are listed under the dependencies section in the `pubspec.yaml` file.

```
dependencies:
  flutter:
    sdk: flutter
  http: ^0.13.3
```

In this example, the `http` package is a regular dependency, as it is required for the app's functionality, such as making network requests.

**2. Development Dependencies (dev_dependencies)**: These are the packages or libraries that are only required during the development of your project. They typically include tools for testing, linting, or generating code, among others. Development dependencies are not included when you build or compile your project for release. They are listed under the `dev_dependencies` section in the `pubspec.yaml` file.

```
dev_dependencies:
  flutter_test:
    sdk: flutter
  pedantic: ^1.11.0
```
In summary, the main differences between regular dependencies and development dependencies are their usage and inclusion in the build process. Regular dependencies are essential for your app's functionality and are included in both development and production builds, while development dependencies are only needed during development and are excluded from production builds.


### 9. Should we declare a variable to allocate size from media query in build method? why?

When using MediaQuery in a Flutter app, it's a common practice to declare a variable to store the size obtained from MediaQuery inside the build method. Although declaring the variable inside the build method means it will be redeclared every time the widget rebuilds, the impact on memory is minimal, especially when compared to the benefits it provides in terms of code readability, maintainability, and ease of access to screen dimensions when laying out your widgets.

Here's an example:
```
import 'package:flutter/material.dart';

class MyHomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Declare a variable to store the size from MediaQuery
    final screenSize = MediaQuery.of(context).size;

    return Scaffold(
      appBar: AppBar(title: Text('Media Query Example')),
      body: Center(
        child: Container(
          width: screenSize.width * 0.8,
          height: screenSize.height * 0.4,
          color: Colors.blue,
          child: Center(
            child: Text(
              'Hello, Flutter!',
              style: TextStyle(fontSize: 24, color: Colors.white),
            ),
          ),
        ),
      ),
    );
  }
}
```

In this example, we declare a `screenSize` variable inside the `build` method and use it to calculate the dimensions of the `Container` widget. This ensures that the container dimensions are proportional to the screen size, making the layout responsive to different screen sizes and orientations.

The `build` method is called whenever the widget rebuilds, which means the `screenSize` variable will be redeclared each time. However, the impact on memory is minimal, as the variable simply holds a reference to a `Size` object, which is relatively lightweight. The benefits of improved code readability, maintainability, and ease of access to screen dimensions outweigh the small memory impact.

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






