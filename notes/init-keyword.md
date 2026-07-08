# `init` block in Kotlin | Amit Shekhar | @OutcomeSchool

## Overview

* **What the Video Teaches:** This lesson breaks down the concept, purpose, behavior, execution order, and real-world implementation of the `init` block in Kotlin, positioning it in the context of Android technical interview preparation `[00:00:10]`.
* **Who It Is For:** Android developers, Kotlin programmers, and technical job candidates looking to master core object initialization mechanics.
* **Prerequisites:** Basic knowledge of object-oriented programming (OOP) and simple class architecture in Kotlin.
* **Expected Learning Outcomes:** Understand the lifecycle of object creation in Kotlin, how multiple `init` blocks interact with property initializers, and how to apply them inside Android `ViewModel` structures.

---

## Detailed Notes

### Topic: Primary vs. Secondary Constructors

* **What It Is:** Kotlin segregates object construction into a singular **Primary Constructor** (declared directly in the class header) and optional **Secondary Constructors** (declared inside the class body) `[00:00:42]`.
* **Why It Exists:** To provide a highly concise class definition syntax for common single-constructor scenarios, while still offering flexible constructor overloading for complex initialization paths.
* **How It Works:** Properties declared in the primary constructor are initialized instantly when the class header is invoked `[00:00:55]`. Secondary constructors are declared using the `constructor` keyword and must explicitly delegate back to the primary constructor if one exists `[00:01:07]`.
* **Core Difference:** The primary constructor is strictly a declarative header and **cannot contain execution code blocks** `[00:01:16]`. Conversely, secondary constructors **can** contain functional execution blocks `[00:01:24]`.

### Topic: The `init` Block

* **What It Is:** An initializer block enclosed by the `init` keyword that acts as the functional code container for class setup tasks `[00:01:42]`.
* **Why It Exists:** Because primary constructors cannot house procedural code, Kotlin introduces the `init` block to run side effects, parameter checks, or setup scripts immediately during primary object creation `[00:01:42]`.
* **When to Use It:** Use it whenever you rely on a primary constructor but require code execution (such as parameter verification or initial function triggers) during object instantiation `[00:04:10]`.
* **How It Works / Execution Order:** 1. The primary constructor values are bound `[00:02:23]`.
2. The `init` block runs immediately after the primary constructor completes `[00:02:23]`.
3. The secondary constructor body runs *after* all primary initialization is completed `[00:02:32]`.
* **Key Rules & Behaviors:**
* It does not accept parameters or arguments `[00:03:18]`.
* It can read properties and variables passed down by the primary constructor `[00:02:57]`.
* A single class can feature multiple `init` blocks `[00:03:05]`.



---

## Code Examples

### Example 1: Comprehensive Initialization & Ordering Execution

This snippet demonstrates how multiple `init` blocks, interleaved member properties, and secondary constructors behave at runtime `[00:03:25]`.

```kotlin
class Mentor(val firstName: String, val lastName: String) {
    
    // First initialization block
    init {
        println("First init block: $firstName")
    }
    
    // Interleaved property initialization
    val fullName = "$firstName $lastName".also { 
        println("Full name property: $it") 
    }
    
    // Second initialization block
    init {
        println("Second init block")
    }
    
    // Secondary Constructor
    constructor(firstName: String, lastName: String, technology: String) : this(firstName, lastName) {
        println("Secondary constructor: $technology")
    }
}

```

#### Line-by-Line Breakdown:

* `class Mentor(...)`: Declares the class alongside its primary constructor, immediately generating fields for `firstName` and `lastName`.
* `init { println(...) }` (First Block): Evaluates immediately after primary constructor values are passed. It accesses `firstName` from the primary parameter scope `[00:02:57]`.
* `val fullName = ... .also { ... }`: Evaluated next because it is placed directly beneath the first `init` block. Initialization reads sequentially from top to bottom `[00:03:05]`.
* `init { ... }` (Second Block): Executes right after the `fullName` property evaluation concludes.
* `constructor(...) : this(...)`: Declares a secondary constructor. The `: this(...)` tells Kotlin to pass inputs up to the primary constructor first. Its internal code block is executed last `[00:02:32]`.

### Example 2: Real-World Android Application (`ViewModel` Pattern)

This pattern showcases how developers use `init` blocks to fetch remote data immediately when a screen's business logic layer is built `[00:04:23]`.

```kotlin
class NewsListViewModel(private val repository: NewsRepository) : ViewModel() {
    
    init {
        // Automatically start fetching data on object creation
        fetchNews()
    }
    
    private fun fetchNews() {
        // Private network or database operations using the injected repository
    }
}

```

#### Why It Works:

When an Android Jetpack Architecture Component spins up `NewsListViewModel`, the constructor injects a `NewsRepository` instance. The `init` block instantly fires `fetchNews()`, eliminating the need to write manual lifecycle hooks outside the class to load the initial UI dataset `[00:04:30]`.

---

## Outputs / Results

When you invoke the `Mentor` class constructor using a secondary constructor statement `[00:03:37]`:

```kotlin
val mentor = Mentor("Amit", "Shekhar", "Android")

```

### Console Output:

```text
First init block: Amit
Full name property: Amit Shekhar
Second init block
Secondary constructor: Android

```

### Explanation of Results:

1. **`First init block: Amit`**: Confirms that execution starts at the top of the body immediately after the primary constructor parameters bind `[00:03:46]`.
2. **`Full name property: Amit Shekhar`**: Demonstrates that properties interleaved between `init` blocks evaluate smoothly in textual order `[00:03:54]`.
3. **`Second init block`**: Runs seamlessly as the top-to-bottom sequence finishes checking the body initializers `[00:03:54]`.
4. **`Secondary constructor: Android`**: Prints last, proving that a secondary constructor block is forced to wait until all primary body setup routines finish `[00:03:54]`.

---

## Examples Used

* **Toy Example (`Mentor` Class):** Used to trace execution pathways down a source file to definitively verify line-by-line compiler evaluations `[00:03:25]`.
* **Business Architecture Example (`NewsListViewModel`):** Illustrates clean architecture design by automating safe initial network fetch routines during framework injection setups `[00:04:23]`.

---

## Analogies

### Additional Context (Analogy for Clarity)

Think of a primary constructor as the foundational blueprint structure of a home, an `init` block as the integrated wiring and plumbing that must be run through the core framing, and the secondary constructor as the post-construction interior styling. You cannot decorate the living room (secondary constructor) until the basic underlying frame and foundational utilities (`init` blocks) are securely operational.

---

## Visual Explanations

The instructor outlines the processing hierarchy sequentially on slides `[00:02:41]`:

* **Top-Down Sequential Path:** The text highlights that properties and `init` blocks are bound tightly by their structural layout positions.
* **Deferred Constructor Branching:** The layout displays structural blocks indicating that secondary execution branches are held back until body execution steps complete.

---

## Step-by-Step Processes

### Object Instantiation Workflow

1. **Primary Parameter Mapping:** Arguments are passed to the class header and mapped to primary parameters `[00:02:23]`.
2. **Sequential Body Phase:** The compiler traverses down the class body from top to bottom, evaluating property initialization expressions and `init` block scopes concurrently in order of appearance `[00:03:05]`.
3. **Secondary Constructor Execution:** If a secondary constructor initiated the request, control shifts to its block body only after step 2 completes `[00:02:32]`.

---

## Commands

*None mentioned in this theoretical video.*

---

## APIs / Functions / Methods

### The `init` Keyword

* **Syntax:** `init { // initialization tasks }`
* **Parameters:** None. It does not accept any parameters `[00:03:18]`.
* **Return Value:** None (implicitly returns `Unit`).
* **Behavior:** Serves as a code container that runs in step with property construction.
* **Common Mistakes:** Declaring a return statement inside `init` or appending parentheses to the keyword will result in a compiler error.

---

## Definitions

* **Primary Constructor:** A constructor declared in the class header that sets up properties but cannot handle execution logic blocks `[00:00:55]`.
* **Secondary Constructor:** An alternative constructor declared inside a class body using the `constructor` keyword to facilitate extra setup options `[00:01:07]`.
* **Init Block:** A distinct block used to perform block tasks during object initialization when a primary constructor is present `[00:01:42]`.

---

## Best Practices

* **Keep Init Blocks Lightweight:** Avoid doing long-running blocking tasks (like high-latency disk operations) directly inside `init` blocks without spawning asynchronous workers.
* **Organize Properties Cohesively:** Avoid scattering multiple `init` blocks erratically between properties; keep initializations ordered uniformly to prevent forward-reference confusion `[00:03:05]`.
* **Enforce Automated Triggers:** Use `init` blocks to encapsulate validation rules or initial routine tasks so objects cannot exist in an uninitialized state `[00:04:23]`.

---

## Common Mistakes

* **Mistaking the Initialization Order:** Believing secondary constructor logic executes before the class body's `init` block. `init` always runs first `[00:02:32]`.
* **Parameter Errors:** Trying to supply custom argument lists directly to an `init` block block. It can only read from class properties and parameters already existing in the primary constructor `[00:02:57]`.
* **Forward Reference Compilation Errors:** Attempting to use a property inside an `init` block when that property is defined lower down in the file body layout `[00:03:05]`.

---

## Tips & Tricks

* **Trace Initialization with `.also`:** When debugging tricky class configurations, use Kotlin's scoped `.also { println(...) }` block functions inside property declarations to clearly log the exact order of object creation `[00:03:25]`.

---

## Important Takeaways

1. **Primary Limitation:** The primary constructor cannot feature standard code blocks `[00:01:16]`.
2. **Execution Timing:** `init` blocks evaluate immediately after primary parameter assignment but always before secondary constructor bodies fire `[00:02:23]`.
3. **Interleaved Flow:** Multiple `init` blocks run in top-down sequential order, interleaved with property initializers `[00:03:05]`.

---

## Cheatsheet

```kotlin
class Reference(val baseParam: String) {
    // 1. Primary parameter runs first
    
    init {
        // 2. First init block runs next
    }
    
    val state = "Ready" 
    // 3. Property assignment runs sequentially
    
    init {
        // 4. Second init block follows
    }
    
    constructor(extra: Int) : this("Go") {
        // 5. Secondary constructor runs LAST!
    }
}

```

---

## FAQs

**Q: Can a class have multiple `init` blocks?** **A:** Yes. A class can declare multiple `init` blocks. They will execute sequentially from top to bottom as they appear in the class code `[00:03:05]`.

**Q: Can `init` blocks access parameters from the primary constructor?** **A:** Yes. The parameters of the primary constructor are completely accessible inside the initializer blocks `[00:02:57]`.

### Additional Context FAQ

**Q: What happens if a class does not declare a primary constructor?** **A:** If no primary constructor is declared, `init` blocks still execute. They run automatically right before any chosen secondary constructor body executes.

---

## Summary

The `init` block acts as the functional engine room for Kotlin’s primary constructor, which is otherwise unable to host procedural code blocks `[00:01:42]`. Object creation follows a strict lifecycle timeline: primary parameters are assigned, class body declarations and `init` blocks run in a top-down structural order, and finally, secondary constructor bodies are executed `[00:02:41]`. Mastering this sequence is highly valuable for writing clean code and answering core Android technical interview questions `[00:00:10]`.
