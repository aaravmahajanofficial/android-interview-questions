# Launch vs Async in Kotlin Coroutines | Amit Shekhar | @OutcomeSchool

---

## Overview

* **What the Video Teaches**: This guide details the explicit technical differences between two foundational coroutine builders in Kotlin: `launch` and `async`. It focuses on their distinct return behaviors, real-world use cases, and their diverging exception propagation models.
* **Who It Is For**: Android developers, Kotlin backend engineers, and candidates preparing for technical software engineering interviews.
* **Prerequisites**: A basic comprehension of asynchronous programming concepts, Kotlin language fundamentals, and a high-level acquaintance with coroutines.
* **Expected Learning Outcomes**:
* Debunk common industry misconceptions about coroutine return values.
* Master the structural and practical differences between `Job` and `Deferred` instances.
* Appropriately structure exception-handling blocks for asynchronous workflows to avoid application crashes or silent resource failures.
* Understand why top-level scopes like `GlobalScope` should be avoided in production environments.



---

## Detailed Notes

### Topic 1: Core Mechanics of Coroutine Builders

* **What It Is**: Both `launch` and `async` are extension functions belonging to `CoroutineScope` used to initialize and execute concurrent processing blocks [[00:31](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=31)].
* **Why It Exists**: To abstract thread management. Just as we call standard methods to spin up underlying native system threads, Kotlin uses these builders to launch lightweight coroutines over a designated dispatcher framework [[00:39](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=39)].
* **When to Use It**: Use them whenever an asynchronous or non-blocking operation must be scheduled concurrently (e.g., executing network requests, disk operations, or heavy mathematical computations).
* **How It Works**: They intercept a block of functional code, append a coroutine context, and hand execution over to a coroutine dispatcher to process logic cleanly across thread pools.
* **Advantages**: Avoids manual thread configuration, reduces resource overhead via a lightweight green-threading architecture, and standardizes asynchronous sequential-looking code patterns.
* **Disadvantages**: Misuse of scopes or bad choices between builders can introduce hidden runtime crashes or silent processing drops.
* **Common Mistakes**: Believing the widespread industry rumor that the `launch` builder does not return anything [[00:46](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=46)].

### Topic 2: Return Behaviors (`Job` vs `Deferred`)

* **What It Is**: `launch` yields a `Job` object, whereas `async` yields a `Deferred<T>` object [[01:08](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=68)].
* **Why It Exists**: Distinct return instances specify whether a computational workflow is intended to return a data payload back to the calling scope upon completion.
* **When to Use It**:
* **`launch`**: Use when the asynchronous workflow is a "fire-and-forget" operation where the final evaluated data result is not needed by the parent caller [[01:43](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=103)].
* **`async`**: Use when the asynchronous workflow evaluates a result payload that must be fetched later via suspending coordination [[01:43](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=103)].


* **How It Works**:
* `Job` monitors background status but holds no computational data placeholder [[01:36](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=96)].
* `Deferred<T>` extends `Job` and introduces the `.await()` suspending function to fetch the underlying value `T` once resolved [[01:19](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=79)].


* **Best Practices**: Use `launch` for operations where side effects are the only goal. Use `async` when you explicitly require data back from a background task.

### Topic 3: Exception Handling Mechanisms

* **What It Is**: The underlying strategy used by Kotlin Coroutines to propagate unhandled exceptions thrown inside respective builder blocks [[05:23](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=323)].
* **Why It Exists**: To maintain structural concurrency guarantees. Unhandled errors must either bubble up to notify the runtime environment or be encapsulated securely within the asynchronous task handle.
* **When to Use It**: Must be understood thoroughly to prevent application crashes on production setups [[05:29](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=329)].
* **How It Works**:
* **`launch`**: Unhandled exceptions immediately propagate upward to the parent context. If no coroutine exception handler is specified, it will instantly crash the application [[05:29](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=329)].
* **`async`**: Unhandled exceptions are safely trapped and stored within the resulting `Deferred` instance [[05:43](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=343)]. The exception is not propagated outwards automatically and will be silently ignored unless `.await()` is triggered, at which point the exception re-throws [[05:53](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=353)].



---

## Code Examples

### 1. Basic Launch Mechanics

```kotlin
val job = GlobalScope.launch(Dispatchers.Default) {
    // Perform fire-and-forget task
}
// You can cancel the job
job.cancel()

```

* **Line-by-Line Breakdown**:
* `GlobalScope.launch(...)`: Invokes the launch builder on the broad global scope.
* `Dispatchers.Default`: Directs execution to a backing thread pool optimized for CPU-intensive work.
* `val job`: Captures the returned `Job` instance (proving it returns an object) [[02:12](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=132)].
* `job.cancel()`: Terminates the running coroutine prematurely [[02:29](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=149)].



### 2. Basic Async Mechanics

```kotlin
val deferred = GlobalScope.async(Dispatchers.Default) {
    // Perform task and return result
    "Result Data" 
}
// Retrieve the result
val result = deferred.await()

```

* **Line-by-Line Breakdown**:
* `GlobalScope.async(...)`: Initiates a coroutine optimized for fetching a result.
* `"Result Data"`: The last expression evaluated in the lambda block serves as the return payload.
* `deferred.await()`: A suspending invocation that pauses the current coroutine until the result is ready [[03:53](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=233)].



### 3. Exception Handling in Launch

#### Initial Version (App Crash Encountered)

```kotlin
GlobalScope.launch(Dispatchers.Default) {
    throw RuntimeException("Error in launch")
}

```

#### Updated Version (Robust/Handled)

```kotlin
GlobalScope.launch(Dispatchers.Default) {
    try {
        throw RuntimeException("Error in launch")
    } catch (e: Exception) {
        // Exception caught and managed cleanly inside the coroutine
    }
}

```

* **Line-by-Line Breakdown**:
* `try { ... } catch`: Encapsulates the dangerous execution paths directly inside the coroutine block so the underlying exception never leaks to the parent scope to crash the application [[06:25](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=385)].



### 4. Exception Handling in Async

#### Initial Version (Silent Failure/Dropped Exception)

```kotlin
val deferred = GlobalScope.async(Dispatchers.Default) {
    throw RuntimeException("Error in async")
}

```

#### Updated/Final Version (Catching via .await())

```kotlin
val deferred = GlobalScope.async(Dispatchers.Default) {
    throw RuntimeException("Error in async")
}

try {
    val result = deferred.await()
} catch (e: Exception) {
    // Exception thrown by await() is intercepted here
}

```

* **Line-by-Line Breakdown**:
* `deferred.await()`: Triggers execution observation. Since the async block failed, the wrapped exception is re-thrown right at this line, which allows the outer `try-catch` wrapper to handle it smoothly [[06:52](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=412)].



---

## Outputs / Results

* **Launch Unhandled Crash Output**: When running an unhandled exception inside a `launch` context, the console/runtime logs will print a stack trace followed by a terminated process message:
```text
Exception in thread "DefaultDispatcher-worker-1" java.lang.RuntimeException: Error in launch
Process finished with exit code 1 (or Application Crash on Android UI)

```


*Meaning*: The exception escaped the coroutine bounds, causing an unhandled crash because `launch` propagates errors to the scope directly [[05:29](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=329)].
* **Async Unhandled Silent Output**: When running an unhandled exception inside an `async` context without calling `.await()`, the log console remains entirely clear.
*Meaning*: The error is held quietly inside the `Deferred` instance, escaping observation unless specifically invoked via an awaited checkpoint [[05:43](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=343)].

---

## Examples Used

* **Moving a File (Real-world Example)**: Moving a file from one physical storage folder location to another [[03:11](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=191)].
* *Utility*: Demonstrates when to choose `launch`. You care whether the task finishes its systemic side effect successfully or fails, but there is no meaningful computation return object generated by moving the file [[03:17](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=197)].


* **Interview QA Practice**: The general framework of asking "Launch vs Async" directly addresses common technical evaluation criteria used to judge a developer's true proficiency with concurrency details [[00:11](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=11)].

---

## Analogies

* **Thread Comparison**: The instructor compares coroutine builders to starting threads (`Thread.start()`) [[00:39](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=39)].
* *What it represents*: A conceptual baseline for launching async processes.
* *Why it helps*: Developers already familiar with multi-threaded environments immediately recognize that calling `launch` or `async` kicks off a parallel operational block.
* *Technical Concept*: Under the hood, Kotlin coroutines are multiplexed over a shared Java thread pool via dispatchers rather than map 1-to-1 to expensive system threads.



---

## Visual Explanations

*The instructor presents structural rules via slides throughout the video. The slide representations are structured as follows:*

* **The Misconception Slide**: Shows a common technical statement ("launch does not return anything") labeled clearly as a wrong design conclusion [[00:46](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=46)].
* **Return Type Mapping Hierarchy**:
* `launch` $\rightarrow$ points directly to `Job` [[01:08](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=68)].
* `async` $\rightarrow$ points directly to `Deferred Job` [[01:19](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=79)].


* **Exception Flow Representation**:
* Unhandled `launch` error flows straight out to crash the parent system application [[05:29](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=329)].
* Unhandled `async` error routes inwardly into a safe `Deferred` storage capsule, remaining dropped/silent unless unsealed via an await process [[05:43](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=343)].



---

## Step-by-Step Processes

### Basic Workflow for Async Processing & Exception Capture

1. **Define Coroutine Context**: Establish a target execution context using valid scopes (e.g., `viewModelScope`).
2. **Launch via Async Builder**: Initialize processing through `async(Dispatchers.Default) { ... }` [[03:45](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=225)].
3. **Isolate Potential Errors**: If exceptions might manifest inside the async task, wrap the eventual collection step (`.await()`) with defensive blocks.
4. **Invoke Suspension via Await**: Call `deferred.await()` enclosed within a `try-catch` structural component [[06:52](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=412)].
5. **Handle Gracefully**: Intercept the thrown payload inside the `catch` clause to avoid application state disruption [[06:58](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=418)].

---

## Commands

*(No direct terminal or command-line instructions were presented in this video session. All examples are written purely in the Kotlin language syntax).*

---

## APIs / Functions / Methods

### 1. `launch`

* **Syntax**: `fun CoroutineScope.launch(context: CoroutineContext = EmptyCoroutineContext, start: CoroutineStart = CoroutineStart.DEFAULT, block: suspend CoroutineScope.() -> Unit): Job`
* **Parameters**:
* `context`: The backing execution ruleset (e.g., dispatchers) [[02:12](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=132)].
* `start`: The startup layout strategy options.
* `block`: The executable suspension lambda code block.


* **Return Value**: A `Job` instance representing the lifestyle of the coroutine [[01:08](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=68)].
* **Behavior**: Launches an asynchronous operation without expecting a return payload ("fire and forget") [[01:43](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=103)].

### 2. `async`

* **Syntax**: `fun <T> CoroutineScope.async(context: CoroutineContext = EmptyCoroutineContext, start: CoroutineStart = CoroutineStart.DEFAULT, block: suspend CoroutineScope.() -> T): Deferred<T>`
* **Parameters**: Same setup infrastructure as `launch`.
* **Return Value**: A `Deferred<T>` instance [[01:19](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=79)].
* **Behavior**: Runs a task asynchronously and allows its evaluation payload to be requested concurrently [[01:30](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=90)].

### 3. `Deferred.await()`

* **Syntax**: `suspend fun await(): T`
* **Return Value**: The resolved generic value typed dynamically from the execution block.
* **Edge Cases**: If the target coroutine encounters an unhandled runtime error during processing, invoking `await()` will re-throw that exception instantly [[06:52](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=412)].

---

## Definitions

### Glossary

* **Coroutine**: A lightweight, cooperative design sequence multiplexed over system threads allowing non-blocking processing operations.
* **Job**: A controllable execution handle managing the life-cycle states and parental relationships of active coroutines [[02:49](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=169)].
* **Deferred**: A specialized subclass of `Job` that acts as a future container for a computed value that will resolve asynchronously [[04:10](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=250)].
* **GlobalScope**: A highly unrestricted, top-level coroutine scope that persists across the total lifetime of an operating system application framework [[02:19](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=139)].

---

## Best Practices

* **Avoid GlobalScope**: Always refrain from implementing production coroutines using `GlobalScope`. Instead, rely on contextual structural options like `viewModelScope` or `lifecycleScope` to automatically dispose of processes alongside UI closures [[04:48](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=288)].
* **Isolate Async Failures**: Always enclose `.await()` invocations or internal `async` structures inside structured error interceptors to prevent silent technical data drops [[05:53](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=353)].
* **Use Launch for Actions with Side Effects**: Reserve `launch` exclusively for operations that perform a state transition without outputting a calculated entity (e.g., logging, file moving, cache clear) [[03:04](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=184)].

---

## Common Mistakes

* **The "Launch Returns Nothing" Fallacy**: Assuming `launch` doesn't yield a reference handle. It returns a fully functioning `Job` object capable of life-cycle inspection or explicit cancellation [[00:57](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=57)].
* **Ignoring Silent Async Crashes**: Forgetting to safely handle errors generated in an `async` segment, which leaves hidden bugs unlogged or unaddressed because the system drops the exception silently [[05:53](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=353)].

---

## Tips & Tricks

* **Instant Job Cancellation**: Keep track of a returned `Job` handle from your coroutine builders so you can invoke `.cancel()` inside clean-up states (such as a view destruction cycle), avoiding long-running background memory leaks [[02:36](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=156)].

---

## Important Takeaways

* Both `launch` and `async` are coroutine initialization wrappers [[00:31](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=31)].
* `launch` returns a standard `Job` (ideal for Fire and Forget tasks) [[01:08](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=68)].
* `async` returns a `Deferred` handle (ideal for tasks returning a calculated payload via `.await()`) [[01:19](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=79)].
* Unhandled `launch` errors crash applications immediately if left untouched [[05:29](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=329)].
* Unhandled `async` errors wait silently until checked via `.await()` execution boundaries [[05:43](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=343)].

---

## Cheatsheet

| Core Feature | `launch` Builder | `async` Builder |
| --- | --- | --- |
| **Primary Intent** | Fire and forget operations [[01:43](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=103)] | Compute and return result [[01:43](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=103)] |
| **Return Handle** | `Job` [[01:08](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=68)] | `Deferred<T>` [[01:19](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=79)] |
| **Result Retrieval** | N/A (No results carried) [[01:08](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=68)] | suspension via `.await()` [[01:19](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=79)] |
| **Unhandled Exception** | Immediately crashes the application [[05:29](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=329)] | Hidden silently inside `Deferred` [[05:43](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=343)] |
| **Common Use Case** | File relocation, DB entry mutation [[03:11](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=191)] | Network fetch, intensive parallel calculation |

---

## FAQs

**Q: Does calling `launch` return absolute Void / Unit at a compiler level?** **A:** No. `launch` explicitly returns a `Job` handle instance [[01:08](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=68)]. It does not output the internal calculated result of the lambda block, but it does return the control handle itself [[02:12](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=132)].

**Q: Why didn't my application crash when an exception occurred inside an `async` block?** **A:** This is by design. `async` encapsulates failures directly inside its `Deferred` object container [[05:43](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=343)]. The failure remains completely muted until you attempt to read the result by calling `.await()` [[05:53](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=353)].

**Q: Why should we avoid using `GlobalScope` in real applications?** **A:** `GlobalScope` operates without structural concurrency bounds across the absolute runtime life of the app process [[04:42](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=282)]. This makes it highly prone to creating severe background resource drains or memory leaks when components are torn down [[04:48](http://www.youtube.com/watch?v=B4AfTPpCU5o&t=288)].

---

## Summary

In this lesson, instructor Amit Shekhar clarifies the critical differences between `launch` and `async` builders in Kotlin Coroutines. While both serve as mechanics to spin up asynchronous processes, `launch` serves operations built around fire-and-forget goals by returning a `Job` instance, while `async` encapsulates parallel tasks that compute data outputs wrapped in a `Deferred` container. Furthermore, understanding their distinct exception habits—where `launch` errors escalate directly to crash workflows and `async` captures them inside deferred boundaries—is essential for developing highly stable and performant Android or Kotlin applications.

---

*Reference Video:* [Launch vs Async in Kotlin Coroutines | Amit Shekhar | @OutcomeSchool](https://youtu.be/B4AfTPpCU5o?si=mae6rJk0Ka9Kf-zf)
