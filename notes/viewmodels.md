# What is a ViewModel and how is it useful? | Amit Shekhar

## Overview

* **What the video teaches:** This guide covers the definition, purpose, and lifecycle behavior of the Android Architecture Component's `ViewModel` class. It addresses why it is essential for handling UI state preservation, how it differs from traditional saved instances, and how to discuss it effectively in technical interviews.
* **Who it is for:** Android developers, architecture enthusiasts, and candidates preparing for technical Android engineering interviews.
* **Prerequisites:** A foundational understanding of Android application components (Activities and Fragments), the Android activity lifecycle, and the concept of configuration changes.
* **Expected learning outcomes:** By the end of this study guide, you will understand how a `ViewModel` retains state across screen rotations, when to use it for encapsulating business logic, and its scale advantages over standard system Bundles.

---

## Detailed Notes

### Topic: The Android ViewModel Class

#### What it is

A `ViewModel` is a architectural component class responsible for preparing, holding, and managing data for a UI controller (an Activity or a Fragment) `[00:00:26]`. It exposes the UI state to the view layer and acts as a barrier between your visual components and your data layers `[00:00:26]`.

#### Why it exists

In Android, UI controllers like Activities are managed by the operating system. The system can destroy or recreate them at any time in response to specific user actions or system events completely outside of your control (such as device rotations) `[00:01:12]`. If an Activity is recreated, any transient UI data it holds in memory is lost. The `ViewModel` was introduced to safely cache and persist UI data through these lifecycle events so that data does not need to be repeatedly re-fetched `[00:01:12]`.

#### When to use it

* When your UI relies on data that is expensive to load (e.g., database queries, network requests) and you want to prevent redundant loads on screen orientation changes `[00:01:30]`.
* When implementing structural design patterns like MVVM (Model-View-ViewModel) to achieve a separation of concerns.
* When you need to process, filter, or combine data from multiple backend repositories before passing the final state to the UI components `[00:00:42]`.

#### How it works

The `ViewModel` is scoped to the lifecycle of its designated lifecycle owner (an Activity or Fragment) `[00:02:21]`. When a configuration change occurs, the owner instance is destroyed and a new instance is created. However, the exact same instance of the `ViewModel` is retained in memory and re-attached to the new Activity instance `[00:02:37]`.

#### Advantages

* **Configuration Change Survival:** Seamlessly preserves state during device rotations or window resizing events without requiring complex manual storage mechanisms `[00:01:12]`.
* **Encapsulates Business Logic:** Serves as an ideal location to merge or slice data coming from multiple repositories before displaying it `[00:00:42]`.
* **Resource Efficiency:** Prevents waste of network bandwidth and processor cycles by eliminating duplicate network transactions upon rotation `[00:01:45]`.

#### Disadvantages

* *Not explicitly detailed in the video.* * **Additional Context:** Storing references to long-lived Android UI elements or standard context objects (`View`, `Activity`, etc.) inside a `ViewModel` can lead to severe memory leaks since the `ViewModel` outlives the hosting Activity instance.

#### Common Mistakes

* Relying heavily on the standard system Bundle (`onSaveInstanceState`) to cache massive data models or lists, which breaks the platform's IPC limits `[00:03:49]`.

#### Best Practices

* Keep UI controllers (Activities/Fragments) completely lean; they should only handle rendering layouts and capturing user interactions. Move data aggregation and preparation rules directly inside the `ViewModel` `[00:00:34]`.

---

## Code Examples

*No explicit code listings or development environments were displayed by the instructor during this theoretical interview discussion.*

### Additional Context: Basic Implementation of a ViewModel

To assist you in practicing the concepts discussed by the instructor, review this standard structure of an Android ViewModel implementation using Kotlin:

#### Initial Version: Creating the ViewModel Class

```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.LiveData

class UserViewModel : ViewModel() {
    // Exposing the state to the UI via LiveData
    private val _userData = MutableLiveData<String>()
    val userData: LiveData<String> get() = _userData

    fun loadUserData() {
        // Simulating data fetch from a data source or multiple repositories
        _userData.value = "Loaded User Profile Data"
    }
}

```

#### Updated Version: Consuming the ViewModel in an Activity

```kotlin
import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import androidx.activity.viewModels
import com.example.databinding.ActivityUserBinding

class UserActivity : AppCompatActivity() {

    // Lazily initiating the ViewModel scoped to this Activity instance
    private val viewModel: UserViewModel by viewModels()
    private lateinit var binding: ActivityUserBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityUserBinding.inflate(layoutInflater)
        setContentView(binding.root)

        // Observing the exposed UI state
        viewModel.userData.observe(this) { data ->
            binding.textView.text = data
        }

        // Trigger loading logic; survives screen rotations perfectly
        if (savedInstanceState == null) {
            viewModel.loadUserData()
        }
    }
}

```

---

## Outputs / Results

*No compiler logs, runtime output streams, or direct screen demonstrations occurred within this clip.*

---

## Examples Used

### Real-World App Network Fetch Example

The instructor outlines a typical mobile app scenario `[00:01:30]`:

1. An application initiates an asynchronous network call to fetch a resource.
2. The network responds successfully, and the app displays the payload on screen.
3. The user rotates their phone horizontally.
4. Without a `ViewModel`, the app destroys the current screen layout, instantiates a brand new layout, and has to rerun the entire network transaction or utilize heavy caching subsystems to recover `[00:01:52]`. With a `ViewModel`, the data stays safely cached in memory, and the newly rotated screen immediately renders the pre-existing data `[00:01:45]`.

---

## Analogies

*The instructor did not employ figurative analogies in this specific lesson, relying instead on concrete platform runtime scenarios.*

---

## Visual Explanations

*No structural whiteboards, slide presentations, or architectural tree layouts were used visually in this verbal presentation.*

---

## Step-by-Step Processes

### Configuration Change State Resolution Workflow

When a configuration shift happens on an Android device, the internal lifecycle operations run as follows `[00:01:30]`:

```
User rotates screen
   ↓
System triggers configuration change event
   ↓
Host UI Controller (Activity) enters destruction phase
   ↓
ViewModel remains preserved in memory (does not clear)
   ↓
New UI Controller (Activity) instance gets instantiated by the OS
   ↓
New Activity re-requests/binds to the active ViewModel instance
   ↓
UI immediately renders existing data without a duplicate network call

```

---

## Commands

*No terminal, command-line interfaces, or build commands were utilized in this lecture.*

---

## APIs / Functions / Methods

### `onSaveInstanceState(outState: Bundle)`

* **Syntax:** `override fun onSaveInstanceState(outState: Bundle)`
* **Parameters:** `outState` (A `Bundle` map object in which to place your saved state).
* **Return Value:** `Unit`
* **Behavior:** A lifecycle hook method that writes minimal primitive data to a system container intended to persist across configuration changes and system-initiated process death `[00:03:36]`.
* **Edge Cases & Limitations:** This method is restricted by hardware binder transaction limits. It cannot handle substantial object graphs or long collections `[00:03:56]`.
* **Common Mistakes:** Passing heavy database models, network lists, or bitmaps inside this bundle parameter, which inevitably results in a runtime crash `[00:03:56]`.

---

## Definitions

### ViewModel

An architectural abstraction layer class that contains the temporary configurations and state details displayed to a view, ensuring state survival across runtime orientation alterations and separating user interface elements from base business operations `[00:00:26]`.

### Configuration Change

An event that occurs during runtime—such as a language change, keyboard availability change, or physical device rotation—that completely restarts an active Android Activity structure `[00:01:12]`.

### Lifecycle Awareness

The structural capacity of a software element to track the lifecycle status of adjacent components (such as an Activity or Fragment) and modify its execution boundaries depending on those shifts `[00:02:21]`.

---

## Best Practices

* **Encapsulate Cross-Repository Calculations:** When combining separate payloads from different entities or filtering properties down for display, place that logic cleanly inside your `ViewModel` instead of clogging up the view controllers `[00:00:42]`.
* **De-clutter the View Layer:** Restrict your Fragment and Activity classes strictly to standard event handling and direct UI bindings `[00:00:34]`.

---

## Common Mistakes

* **Treating Bundles as Bulk Storage:** Using `SavedInstance` storage states for deep structural data models `[00:03:49]`. A bundle is designed only to store shallow navigation ids, flags, or configuration keys; any data collections belong inside a dedicated memory container like a `ViewModel` `[00:03:56]`.

---

## Tips & Tricks

* **Reducing Redundant Operations:** Leverage the persistence of the `ViewModel` layer to store complex, computationally heavy database transforms or server requests, ensuring that they only occur once throughout the lifecycle scope of a user's target task `[00:01:52]`.

---

## Important Takeaways

* A **ViewModel** stores UI states and exposes them, isolating core app routines from immediate display logic `[00:00:26]`.
* It provides robust architecture support by surviving common runtime **configuration changes** such as rotating a screen `[00:01:12]`.
* It possesses native **lifecycle awareness**, meaning it remains alive right up until its owning window element is permanently removed or finished by the system `[00:02:21]`.
* Unlike `onSaveInstanceState` which handles minimal key-value stores, a `ViewModel` easily accommodates larger records, objects, and lists `[00:03:56]`.

---

## Cheatsheet

### Architectural Purpose

| Objective | Component Target |
| --- | --- |
| **UI Presentation Layer** | Activities / Fragments |
| **State Retention & Prep** | ViewModel |
| **Persisted/Data Logic** | Repositories / Data Sources |

### Lifecycle Bounds

* **Survives:** Screen rotations, language alterations, display scale adjustments `[00:01:12]`.
* **Terminated On:** Explicit invocation of `finish()` on an Activity, navigating permanently backwards off a view, or clearing the host transaction scope entirely `[00:03:02]`.

---

## FAQs

### Q: How does a ViewModel differ from Saved Instance State?

**A:** `onSaveInstanceState` utilizes a system-managed serialized `Bundle` that writes data onto the disk or system server processes `[00:03:36]`. Because of this IPC pipeline, it can only support minimal datasets. A `ViewModel` resides natively within app memory space, allowing it to easily manage massive data sets, nested structures, and reference lists without serialization overhead `[00:03:56]`.

### Q: When exactly does a ViewModel get removed from memory?

**A:** A `ViewModel` is explicitly cleared and destroyed only when its assigned lifecycle owner is destroyed permanently `[00:02:27]`. If an Activity is undergoing a temporary rotation tear-down, the ViewModel remains unaffected `[00:02:45]`. If the user manually finishes the activity or navigates away permanently, the instance clears `[00:03:02]`.

---

## Summary

The `ViewModel` class holds, isolates, and manages visual states while insulating data processing structures away from transient UI layers `[00:00:26]`. By outlasting temporary screen destruction cycles, it prevents unnecessary device re-fetching, preserves heavy runtime information models, and simplifies building cohesive, testable Android applications `[00:01:45]`.

*For more details, check out the original video: [What is a ViewModel and how is it useful?*](http://www.youtube.com/watch?v=ORtieK5f_zg)
