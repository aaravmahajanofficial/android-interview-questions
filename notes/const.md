This video explains the advantage of using the `const` keyword in *Kotlin*, a common topic in *Android* developer interviews. By decompiling *Kotlin* code into *Java*, the video demonstrates how `const` improves application performance through inlining.

### Key Concepts

*   **Without `const` (0:37 - 1:35):** When defining a variable using only `val` in an object, the compiler generates a getter method. Accessing this variable at runtime involves calling this method, which introduces a slight overhead (0:56).
*   **With `const` (1:41 - 2:24):** By adding the `const` keyword to a `val`, the compiler replaces the variable reference directly with its assigned value during the compilation process. 
*   **Performance Benefit:** Because the value is "inlined," there is **no runtime overhead** to access it, which contributes to better overall application performance (2:16 - 2:24).

### Summary Table

| Feature | `val` (without `const`) | `const val` |
| :--- | :--- | :--- |
| **Access** | Via getter method | Direct value inlining |
| **Runtime Overhead** | Yes | No |
| **Performance** | Standard | Optimized |
