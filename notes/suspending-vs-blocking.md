Here are the detailed notes generated from the video **"Suspending vs Blocking in Kotlin Coroutines"** by Amit Shekhar, outlining the conceptual differences, code implementations, and log results.

---

### **1. Core Concepts & Context**

The video aims to answer a common Android/Kotlin interview question: **Suspending vs. Blocking in Kotlin Coroutines**.

To demonstrate the difference, a custom dispatcher with **exactly one thread** is created:

```kotlin
val customDispatcher = Executors.newSingleThreadExecutor().asCoroutineDispatcher()

```

Using a single-threaded dispatcher highlights how efficiently the thread resource is used (or wasted) under suspending versus blocking operations [[02:21](http://www.youtube.com/watch?v=V2lL_aJp17I&t=141)].

---

### **2. Setup: Simulated Time-Taking Task**

Both examples utilize a simulated asynchronous heavy task that takes 5 seconds to run, explicitly switching to `Dispatchers.IO` using `withContext` [[00:44](http://www.youtube.com/watch?v=V2lL_aJp17I&t=44)].

```kotlin
suspend fun timeTakingTask() {
    withContext(Dispatchers.IO) {
        // Simulates a 5-second long-running task
        Thread.sleep(5000) 
    }
}

```

---

### **3. The Suspending Example**

In a standard suspending coroutine flow, when a coroutine encounters a suspend function (like a context switch to `Dispatchers.IO`), it yields control. The single thread belonging to `customDispatcher` is unblocked and made free to pick up other tasks instead of sitting idle [[02:59](http://www.youtube.com/watch?v=V2lL_aJp17I&t=179)].

#### **Code Sample:**

```kotlin
fun testSuspending(dispatcher: CoroutineDispatcher) {
    CoroutineScope(dispatcher).launch {
        Log.d("Test", "before task 1")
        timeTakingTask()
        Log.d("Test", "after task 1")
    }

    CoroutineScope(dispatcher).launch {
        Log.d("Test", "before task 2")
        timeTakingTask()
        Log.d("Test", "after task 2")
    }
}

```

#### **Execution Results / Logs:**

```text
before task 1
before task 2
after task 1
after task 2

```

* **Why this happens:** When `coroutine 1` executes `timeTakingTask()`, the single thread of `customDispatcher` transfers the 5-second workload onto `Dispatchers.IO` [[02:55](http://www.youtube.com/watch?v=V2lL_aJp17I&t=175)]. Since it is a *suspending* operation, `customDispatcher` suspends its execution for coroutine 1 and is freed up immediately to execute `coroutine 2` [[03:29](http://www.youtube.com/watch?v=V2lL_aJp17I&t=209)], printing "before task 2" before the first task completes. This represents optimal resource utilization [[04:12](http://www.youtube.com/watch?v=V2lL_aJp17I&t=252)].

---

### **4. The Blocking Example (`runBlocking`)**

When `runBlocking` is wrapped inside the routine execution, it forces the current thread to pause and wait until the code inside the block finishes entirely. It strips away the optimization benefits of coroutines [[05:23](http://www.youtube.com/watch?v=V2lL_aJp17I&t=323)].

#### **Code Sample:**

```kotlin
fun testBlocking(dispatcher: CoroutineDispatcher) {
    CoroutineScope(dispatcher).launch {
        runBlocking {
            Log.d("Test", "before task 1")
            timeTakingTask()
            Log.d("Test", "after task 1")
        }
    }

    CoroutineScope(dispatcher).launch {
        runBlocking {
            Log.d("Test", "before task 2")
            timeTakingTask()
            Log.d("Test", "after task 2")
        }
    }
}

```

#### **Execution Results / Logs:**

```text
before task 1
after task 1
before task 2
after task 2

```

* **Why this happens:** Even though `timeTakingTask()` tries to run on a different thread (`Dispatchers.IO`), `runBlocking` locks the executing thread [[05:30](http://www.youtube.com/watch?v=V2lL_aJp17I&t=330)]. The single thread of `customDispatcher` is forced to sit completely idle and block until Task 1 fully completes after 5 seconds [[05:44](http://www.youtube.com/watch?v=V2lL_aJp17I&t=344)]. Only then does it proceed to print "after task 1" and finally begin processing `coroutine 2` [[05:14](http://www.youtube.com/watch?v=V2lL_aJp17I&t=314)].

---

### **Summary Table**

| Feature | Suspending | Blocking (`runBlocking`) |
| --- | --- | --- |
| **Thread Behavior** | The thread is released to do other work [[02:59](http://www.youtube.com/watch?v=V2lL_aJp17I&t=179)]. | The thread is tied up and forced to wait [[05:30](http://www.youtube.com/watch?v=V2lL_aJp17I&t=330)]. |
| **Resource Efficiency** | **Highly Optimized:** Maximum resource utilization [[04:12](http://www.youtube.com/watch?v=V2lL_aJp17I&t=252)]. | **Inefficient:** The thread sits idle without doing work [[05:44](http://www.youtube.com/watch?v=V2lL_aJp17I&t=344)]. |
| **Execution Order** | Interleaved logs (`before 1` $\rightarrow$ `before 2`) [[04:18](http://www.youtube.com/watch?v=V2lL_aJp17I&t=258)]. | Strictly sequential logs (`1 finishes` $\rightarrow$ `2 starts`) [[05:06](http://www.youtube.com/watch?v=V2lL_aJp17I&t=306)]. |

For more interview-focused videos, visit the [Amit Shekhar YouTube Channel](http://www.youtube.com/watch?v=V2lL_aJp17I).
