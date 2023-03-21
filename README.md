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

### 10. How to check if any widget is placed in widget tree?

The `mounted` property can be used to check if a `State` object is currently in the widget tree, but it is specific to stateful widgets. When a `State` object is created by the framework, `mounted` is false. After calling `createState`, the framework calls build for the first time, and then it sets `mounted` to true. When the framework removes the `State` object from the tree, it sets `mounted` to false. Consequently, you can use the `mounted` property to check if a stateful widget is currently in the widget tree.

Here's a simple example demonstrating how to use the mounted property to check if a StatefulWidget is present in the widget tree:

```
import 'package:flutter/material.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Mounted Property Example')),
      body: Center(
        child: RaisedButton(
          onPressed: () {
            // Check if the current StatefulWidget is in the widget tree
            final isMounted = mounted;

            // Display message based on whether the widget is in the widget tree
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(
                content: Text(
                  isMounted ? 'The widget is in the widget tree' : 'The widget is not in the widget tree',
                ),
              ),
            );
          },
          child: Text('Check if Widget is in the Widget Tree'),
        ),
      ),
    );
  }
}
```

To check for the presence of ancestor widgets, you can use the `BuildContext` and the `findAncestorWidgetOfExactType` method.

### 11. What's the difference between mounted and post-frame callback ?

mounted and past-frame callback (usually achieved using `WidgetsBinding.instance.addPostFrameCallback`) are different concepts in Flutter, serving different purposes.

**1. mounted:** The `mounted` property is specific to `State` objects of stateful widgets. It is a boolean flag that indicates whether a `State` object is currently in the widget tree. When a `State` object is created by the framework, `mounted` is false. After calling `createState`, the framework calls build for the first time, and then it sets `mounted` to true. When the framework removes the `State` object from the tree, it sets `mounted` to false. You can use the `mounted` property to check if a stateful widget is currently in the widget tree or to prevent calling `setState` when the widget is not in the widget tree.

**2. (Post-Frame Callback)**: A post-frame callback is a function that you can register to be called after the current frame has been rendered. You can use `WidgetsBinding.instance.addPostFrameCallback` to register a callback that will be executed after the current frame completes. This is useful when you want to perform some action after the framework has finished rendering the current frame, for example, showing a `SnackBar` or navigating to a new route immediately after building the widget.

Here's an example of using a post-frame callback:

```
import 'package:flutter/material.dart';

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(home: MyHomePage());
  }
}

class MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  void initState() {
    super.initState();

    WidgetsBinding.instance.addPostFrameCallback((_) {
      // Perform an action after the frame has been rendered, e.g., show a SnackBar
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Welcome to MyHomePage')),
      );
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Post-Frame Callback Example')),
      body: Center(child: Text('Hello, Flutter!')),
    );
  }
}

```


### 12. When and why we should use `WidgetsBindingObserver`?

`WidgetsBindingObserver` is an interface in Flutter that allows you to listen to various application-level events, such as when the app is paused, resumed, or the system informs the app about changes in text scale factors or accessibility features. You might use a `WidgetsBindingObserver` when you want to perform specific actions based on these events.

Here are some common use cases for WidgetsBindingObserver:

- **Managing resources:** If your app uses resources like network connections or sensors that should be released when the app is paused (sent to the background) and reacquired when the app is resumed, you can use `WidgetsBindingObserver` to listen for these lifecycle events and manage resources accordingly.

- **Updating UI based on system changes**: When the system's text scale factor or accessibility features change, you may need to update your app's UI accordingly. `WidgetsBindingObserver` allows you to listen for these changes and perform necessary UI updates.

- **Saving app state**: You can use `WidgetsBindingObserver` to save the app state when it is paused or terminated by the system, ensuring that important data is preserved.

Here's an example of using `WidgetsBindingObserver` to listen for app lifecycle events:

```
import 'package:flutter/material.dart';
import 'package:flutter/widgets.dart';

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(home: MyHomePage());
  }
}

class MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> with WidgetsBindingObserver {
  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addObserver(this);
  }

  @override
  void dispose() {
    WidgetsBinding.instance.removeObserver(this);
    super.dispose();
  }

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    super.didChangeAppLifecycleState(state);

    switch (state) {
      case AppLifecycleState.inactive:
        print("App is inactive");
        break;
      case AppLifecycleState.paused:
        print("App is paused");
        break;
      case AppLifecycleState.resumed:
        print("App is resumed");
        break;
      case AppLifecycleState.detached:
        print("App is detached");
        break;
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('WidgetsBindingObserver Example')),
      body: Center(child: Text('Hello, Flutter!')),
    );
  }
}

```


### 13. What is InheritedWidget? Why/when do we use it? How to pass data through InheritedWidget and access it from other widgets?

`InheritedWidget` is a special type of widget in Flutter that can efficiently propagate data down the widget tree. It's designed to allow child widgets to access shared data without having to pass it down explicitly through the constructor of each intermediate widget. `InheritedWidget` is particularly useful when you need to share data across multiple widgets, and passing it down via constructors becomes cumbersome.

Here's a step-by-step guide to create, pass data through, and access an `InheritedWidget`:

1. **Create an InheritedWidget subclass**:
To create an InheritedWidget, you need to subclass it and implement the updateShouldNotify method. This method is called when the widget is rebuilt, and it determines whether the updated data should be propagated down the widget tree.

```
class MyInheritedData extends InheritedWidget {
  final int counter;

  MyInheritedData({Key? key, required this.counter, required Widget child})
      : super(key: key, child: child);

  @override
  bool updateShouldNotify(MyInheritedData oldWidget) {
    return oldWidget.counter != counter;
  }

  static MyInheritedData of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<MyInheritedData>()!;
  }
}
```

2. **Pass data through the InheritedWidget**:
To pass data through the `InheritedWidget`, you need to wrap the part of the widget tree that needs access to the data with the InheritedWidget subclass. This makes the data available to all the child widgets.

```
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: MyInheritedData(
        counter: 42,
        child: MyHomePage(),
      ),
    );
  }
}
```
In this example, we wrap `MyHomePage` with `MyInheritedData` and set the `counter` value to 42. Now, any child widgets of `MyHomePage` can access the `counter `value.

3. **Access data from the InheritedWidget**:

To access the data from the `InheritedWidget`, you can use the of method we defined in the `MyInheritedData` subclass. This method takes a `BuildContext` and returns the `InheritedWidget` instance, allowing you to access the shared data.

```
class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    int counter = MyInheritedData.of(context).counter;

    return Scaffold(
      appBar: AppBar(title: Text('InheritedWidget Example')),
      body: Center(child: Text('Counter value: $counter')),
    );
  }
}
```


### 14. AOT vs JIT compiler? In Flutter which compiler gets used in which cases?

AOT (Ahead-of-Time) and JIT (Just-in-Time) are two different compilation techniques used in programming languages, including Dart, the language used to develop Flutter applications. They serve different purposes and are used in different scenarios.

**AOT (Ahead-of-Time) Compilation:**
AOT compilation involves converting the source code into a native machine code or an intermediate bytecode before the program is executed. This process happens during the build stage. When the application is launched, the native code is executed directly by the hardware without any further compilation. AOT has several benefits:

- Faster startup time since the code is already compiled to native code.
- Better performance and optimizations as the compiler can perform complex and time-consuming optimizations during the build process.
- Improved security and code obfuscation, making it harder to reverse-engineer the compiled code.

**JIT (Just-in-Time) Compilation:**
JIT compilation converts the source code into native machine code during the execution of the program, at runtime. This means that the code is compiled and executed in the same process. JIT compilation allows for faster development cycles and enables features like hot reloading in Flutter. However, it may result in slower startup times and increased memory usage due to the runtime compilation overhead. JIT also enables better runtime optimizations, as the compiler can optimize code based on actual usage patterns.

**AOT and JIT Compilers in Flutter:**

In Flutter, both AOT and JIT compilers are used in different scenarios:

- **Development**: During development, Flutter uses the JIT compiler. This enables fast development cycles and hot reloading, allowing developers to see the changes in the code without having to rebuild the entire application. JIT compilation makes it easier to iterate quickly and experiment with the UI and functionality.

- **Production**: In production builds, Flutter uses the AOT compiler. AOT compilation results in faster startup times, better performance, and smaller application sizes. The AOT compiler optimizes the code for the target platform, ensuring that the application runs smoothly on end-user devices.

In conclusion, AOT and JIT compilers serve different purposes in Flutter development. The JIT compiler is used during development to enable fast development cycles and hot reloading, while the AOT compiler is used in production builds to achieve better performance, faster startup times, and smaller application sizes.


### 15. Explain AOT vs JIT combiler advantages and disadvantages in Flutter

In Flutter, both AOT (Ahead-of-Time) and JIT (Just-in-Time) compilers serve different purposes and have their own advantages and disadvantages.

**AOT (Ahead-of-Time) Compilation:**

Advantages:

1. **Faster startup time**: Since the code is compiled to native code before execution, the app launches faster as there is no need for runtime compilation.
2. **Better performance**: The AOT compiler has more time to perform complex optimizations during the build process, resulting in better overall performance.
3. **Smaller memory footprint**: As the code is compiled ahead of time, there is no need for a runtime compiler, reducing the memory usage of the app.
4. **Improved security**: AOT compiled code is more difficult to reverse-engineer, providing better code protection.

Disadvantages:

1. **Longer build times**: AOT compilation can increase build times, as the entire codebase must be compiled during the build process.
2. **Less flexible**: As the code is compiled ahead of time, features like hot reloading are not possible with AOT compilation.

**JIT (Just-in-Time) Compilation:**

Advantages:

1. **Faster development cycle**: JIT compilation enables faster development cycles, as the code is compiled and executed on-the-fly, allowing developers to see changes quickly without rebuilding the entire app.
2. **Hot reloading**: JIT compilation supports hot reloading, which allows developers to inject new code into the running app and see the results immediately without losing the current app state.
3. **Runtime optimizations**: JIT compilers can optimize code based on actual usage patterns and runtime information, potentially leading to better-performing code.

Disadvantages:

1. **Slower startup time**: JIT compilation adds overhead during app startup, as the code must be compiled at runtime before execution.
2. **Increased memory usage**: JIT compilers require additional memory to store the runtime compiler and generated native code.
3. **Potentially less optimized code**: JIT compilers have limited time to optimize code during runtime, which may result in less optimized code compared to AOT compilers.


### 16. What's BehaviourSubject in RxDart ?

`BehaviorSubject` is a type of `StreamController` provided by RxDart, which is an extension of the Dart `Stream` system with additional functionality inspired by ReactiveX (Rx). RxDart adds several new classes and operators to work with streams more efficiently, and `BehaviorSubject` is one of them.

BehaviorSubject is a special kind of stream that has the following characteristics:

1. **Latest value**: A BehaviorSubject remembers the latest value that was emitted by the stream. When a new listener subscribes to the BehaviorSubject, it immediately receives the latest value. This is useful in scenarios where you want new listeners to have access to the current value of the stream without waiting for the next event.

2. **Broadcast stream**: A BehaviorSubject is a broadcast stream, meaning it can have multiple listeners at the same time. When an event is added to the BehaviorSubject, all active listeners receive that event.

3. **Synchronous emission**: BehaviorSubject can emit events synchronously, which means listeners receive events as soon as they are added to the stream. This can be useful for situations where you need to ensure that events are processed in a specific order.

Here's a simple example of using BehaviorSubject in RxDart:

```
import 'package:rxdart/rxdart.dart';

void main() {
  final subject = BehaviorSubject<int>();

  // Listener 1
  subject.stream.listen((value) => print('Listener 1: $value'));

  subject.add(1);
  subject.add(2);

  // Listener 2
  subject.stream.listen((value) => print('Listener 2: $value'));

  subject.add(3);

  subject.close();
}
```
 
### 17. Inherited widget vs Provider which is is better and why?

`InheritedWidget` and `Provider` both serve the purpose of sharing data and managing state across different widgets in a Flutter application, but they have different levels of abstraction and features. Choosing between them largely depends on your specific use case, complexity, and requirements.

**InheritedWidget:**

`InheritedWidget` is a built-in Flutter widget designed to propagate data down the widget tree. It allows child widgets to access shared data without having to pass it down explicitly through the constructor of each intermediate widget. It's a simple and efficient way to share data across multiple widgets.

Advantages:

- Built into the Flutter framework, no additional dependencies required.
- Lightweight and easy to understand for simple use cases.

Disadvantages:

- Lacks advanced features and abstractions provided by Provider.
- Boilerplate code is required to create custom InheritedWidget subclasses.
- Less flexible in managing complex state management scenarios.

**Provider:**

Provider is a popular third-party package built on top of `InheritedWidget`. It provides a more advanced, flexible, and easy-to-use approach to state management in Flutter. `Provider` offers different types of providers for various use cases, such as `ChangeNotifierProvider`, `ValueListenableProvider`, and `StreamProvider`.

Advantages:

- Higher-level abstractions, making it easier to manage complex state scenarios.
- Reduces boilerplate code compared to using InheritedWidget directly.
- Offers built-in support for common state management patterns like ChangeNotifier.
- Easily composable, allowing you to use multiple providers together to manage different parts of the application state.
- Actively maintained and widely used in the Flutter community.

Disadvantages:

- Requires an additional dependency.
- Might be overkill for very simple use cases.

**Conclusion:**

Choosing between `InheritedWidget` and `Provider` depends on your specific use case and requirements:

If you have a simple use case, limited state management needs, and want to avoid adding an additional dependency, using InheritedWidget might be sufficient.
If you require a more advanced, flexible, and easy-to-use solution for state management, especially in complex applications, Provider is the better choice. It reduces boilerplate code, offers built-in support for common state management patterns, and is widely used in the Flutter community.

Overall, `Provider` is generally considered to be the better option for most use cases due to its advanced features, flexibility, and ease of use. However, it's essential to evaluate your specific needs and requirements to make the best decision for your application.

### 18. what is vsync?

`vsync`, short for "vertical synchronization", is a term used in computer graphics and animation to refer to the synchronization of frame updates with the display's refresh rate. In the context of Flutter, `vsync` is often used in relation to animations to ensure they are smooth and consistent with the device's screen refresh rate.

When creating animations in Flutter, you typically use the `Ticker` class, which generates a stream of ticks at a regular interval. These ticks are used to update the animation's state and render a new frame. The `vsync` parameter is provided when creating a `TickerProvider` to help synchronize the ticks with the device's screen refresh rate.

By synchronizing the animation updates with the screen refresh rate, `vsync` helps prevent visual artifacts like screen tearing, where parts of the screen update at different times, resulting in a misaligned or torn appearance. It also ensures that the animation does not update too frequently or too rarely, providing a smoother and more visually pleasing experience for the user.

In Flutter, you often encounter `vsync` when working with `AnimationController`:

```
class _MyAnimatedWidgetState extends State<MyAnimatedWidget> with SingleTickerProviderStateMixin {
  AnimationController _animationController;

  @override
  void initState() {
    super.initState();
    _animationController = AnimationController(
      vsync: this, // Providing the vsync parameter.
      duration: const Duration(seconds: 2),
    );
  }

  @override
  void dispose() {
    _animationController.dispose();
    super.dispose();
  }

  // ...
}
```


### 19. What are mixins? Give an real world example and usecase of mixin.

Mixins are a way to reuse a class's code in multiple class hierarchies. In Dart, mixins allow you to add functionality from one class to another without using inheritance. Instead of extending a class, you can include a mixin to incorporate its properties and methods into your own class. Mixins are useful when you want to share code among multiple unrelated classes or when you want to extend the functionality of a class without actually inheriting from it.

A real-world example of a mixin is Flutter's `SingleTickerProviderStateMixin`. This mixin is used to create a single `Ticker` for animations in a `StatefulWidget`. The `Ticker` is responsible for generating a stream of ticks at regular intervals, which are used to drive animations.

Let's consider a use case where you want to create a simple custom animation in a Flutter app:

```
import 'package:flutter/material.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: Text('Mixin Example')),
        body: Center(child: MyAnimatedWidget()),
      ),
    );
  }
}

class MyAnimatedWidget extends StatefulWidget {
  @override
  _MyAnimatedWidgetState createState() => _MyAnimatedWidgetState();
}

// Using the SingleTickerProviderStateMixin to include the required functionality.
class _MyAnimatedWidgetState extends State<MyAnimatedWidget>
    with SingleTickerProviderStateMixin {
  AnimationController _animationController;
  Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _animationController = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    );
    _animation = Tween<double>(begin: 0, end: 1).animate(_animationController);

    _animationController.repeat(reverse: true);
  }

  @override
  void dispose() {
    _animationController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return FadeTransition(
      opacity: _animation,
      child: Text(
        'Hello, Mixin!',
        style: TextStyle(fontSize: 48),
      ),
    );
  }
}
```

In this example, the `_MyAnimatedWidgetState` class uses the `SingleTickerProviderStateMixin` mixin to include the required functionality for creating a single Ticker. The mixin provides the TickerProvider required by the AnimationController and ensures that the animation updates are synchronized with the device's screen refresh rate, resulting in a smooth and consistent animation.

By using the mixin, you can reuse the `SingleTickerProviderStateMixin` functionality in multiple classes without inheritance, allowing you to create flexible and modular code.

### 20. Do mixins solve the diamond problem?

Yes, mixins can help solve the diamond problem in programming languages that support multiple inheritance, such as Dart. The diamond problem arises when a class inherits from two or more classes that have a common ancestor. In such cases, there is ambiguity regarding which ancestor's methods should be used, and in what order they should be called.

Mixins provide a mechanism for code reuse without relying on multiple inheritance, thus avoiding the diamond problem. When a class includes a mixin, it incorporates the mixin's properties and methods directly, rather than inheriting them from another class. This ensures a linear inheritance hierarchy and eliminates the ambiguity that causes the diamond problem.

Here's an example to illustrate how mixins help avoid the diamond problem:

```
class A {
  void method() {
    print('A\'s implementation of method');
  }
}

class B extends A {
  @override
  void method() {
    print('B\'s implementation of method');
  }
}

class C extends A {
  @override
  void method() {
    print('C\'s implementation of method');
  }
}

mixin MixinD on A {
  @override
  void method() {
    super.method();
    print('MixinD\'s implementation of method');
  }
}

class E extends B with MixinD {} // Multiple inheritance with mixin

void main() {
  E e = E();
  e.method();
}
```
In this example, class `A` has a method called `method()`. Classes `B` and `C` both extend `A` and override `method()`. `MixinD` is a mixin that overrides `method()` as well, but it requires a superclass of type `A` (indicated by on `A`). Class `E` extends `B` and includes `MixinD`.

When we create an instance of `E` and call `method()`, the output is:

```
B's implementation of method
MixinD's implementation of method
```

As you can see, the mixin allows us to combine the functionality of class `B` and `MixinD` without ambiguity or the diamond problem. The linear order of the method resolution is preserved, ensuring a predictable and consistent behavior.


### 21. What is tree shaking ? What are the disadvantages of tree shaking?


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






