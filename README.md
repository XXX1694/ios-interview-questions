# iOS Interview Questions and Answers

[Русская версия](README.ru.md)

685 iOS, Swift, SwiftUI and UIKit interview questions across 31 topics, from junior to senior, each with a short answer. Every question links to its full answer and code on **[irbisa.com](https://irbisa.com/ios/)**, where you can also practice them with spaced repetition, timed mock interviews and a 5-minute diagnostic that finds your weak topics.

## Contents

- [Most asked questions](#most-asked-questions)
- [All topics](#all-topics)
- [Practice](#practice)

## Most asked questions

The iOS questions that come up most often in real interviews.

### 1. What is the difference between `let` and `var` in Swift?

A `let` constant can be assigned exactly once, while a `var` variable can be reassigned any number of times; use `let` by default and switch to `var` only when the value really has to change.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-2401) · [Swift Basics](topics/swift-basics.md)

### 2. What is an optional in Swift, how is it implemented, and how do you unwrap it safely?

An optional either holds a value or holds `nil`, so "this may be missing" is checked by the compiler instead of crashing at runtime; `T?` is shorthand for the generic enum `Optional<T>` with the cases `.some(T)` and `.none`.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-2402) · [Swift Basics](topics/swift-basics.md)

### 3. What is the difference between value types and reference types in Swift?

A value type is copied when you assign it or pass it to a function, so every variable owns its own data; a reference type is shared, so several variables point at one instance and see each other's changes.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-2403) · [Swift Basics](topics/swift-basics.md)

### 4. What is a closure in Swift, and is it a value type or a reference type?

A closure is a block of code you can store in a variable and pass around, and it remembers the variables of the scope where it was created; closures, like functions, are reference types.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-2404) · [Swift Basics](topics/swift-basics.md)

### 5. What is the difference between `weak` and `unowned`, and when do you need `[weak self]` in a closure?

Neither keeps the object alive: `weak` is an optional that becomes `nil` when the object is deallocated, while `unowned` is non-optional and crashes if you touch it after the object is gone. Use `[weak self]` in an escaping closure that `self` itself owns.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62415) · [Swift Basics](topics/swift-basics.md)

### 6. How does error handling work in Swift: `throws`, `try`, `try?`, `try!`, `do`/`catch` and `Result`?

A function that can fail is marked `throws`, every call to it must be marked with `try`, and the error is either handled in `do`/`catch` or passed up to the caller, so failure is visible in the function's signature.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-2406) · [Swift Basics](topics/swift-basics.md)

### 7. What is the difference between a struct and a class in Swift?

Beyond copy vs share, a struct has no inheritance, `deinit` or identity, and a class has no memberwise initializer, `mutating` or automatic `Sendable`.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-2501) · [Swift OOP & Protocols](topics/swift-oop-protocols.md)

### 8. What is the difference between raw values and associated values in a Swift enum?

A raw value is one fixed constant of the same type for every case, known at compile time; an associated value is a payload attached to a case when you create it, and each case can carry different types.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-2503) · [Swift OOP & Protocols](topics/swift-oop-protocols.md)

### 9. What is SwiftUI, and what are its advantages and drawbacks compared with UIKit?

SwiftUI is Apple's declarative UI framework: you describe what the screen looks like for the current state, and the framework works out which changes to apply. UIKit is imperative; you create views and then mutate them yourself.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-2801) · [SwiftUI Fundamentals](topics/swiftui-fundamentals.md)

### 10. How does `NavigationStack` work, and how do you navigate programmatically or from a deep link?

`NavigationStack` keeps its screens as a path of values: `NavigationLink(value:)` appends a value, `.navigationDestination(for:)` turns a value of that type into a screen, and Back removes the last one. Bind the path to state and you navigate by editing an array.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-2804) · [SwiftUI Fundamentals](topics/swiftui-fundamentals.md)

### 11. Walk through the `UIViewController` lifecycle: which callbacks run once, which run on every appearance, and where do you start loading data?

`loadView` and `viewDidLoad` run once per controller instance; `viewWillAppear`, `viewIsAppearing`, `viewDidAppear` and the two disappear callbacks run every time the screen is shown or hidden.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-3002) · [UIKit Fundamentals](topics/uikit-fundamentals.md)

### 12. What is the difference between `frame` and `bounds`, and when is `bounds.origin` not zero?

`frame` is the view's position and size in its superview's coordinate system; `bounds` is the same rectangle in the view's own coordinates. `bounds.origin` is non-zero when the view shows a shifted part of its content: a scroll view's `bounds.origin` is its `contentOffset`.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-3003) · [UIKit Fundamentals](topics/uikit-fundamentals.md)

### 13. What is the responder chain, and how does `hitTest(_:with:)` decide which view receives a touch?

UIKit first hit-tests from the window down to find the deepest view under the finger and delivers the touch there; whatever that view does not handle travels up the responder chain: superview, view controller, window, application.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-3004) · [UIKit Fundamentals](topics/uikit-fundamentals.md)

### 14. Walk me through your iOS background: which apps have you shipped, and what do you do in Xcode day to day?

Open with the **app you ship now** and your role in it, add the one earlier step that explains your path, and close with why this iOS team fits what you want next.

[Full answer →](https://irbisa.com/topic/ios-behavioral-interview/#q-76401) · [Behavioral Interview](topics/ios-behavioral-interview.md)

### 15. What is making you look beyond your current iOS job, and why did this company catch your eye?

Give one **honest reason** phrased as what you are moving toward, then connect it to something concrete you noticed in this company's iOS product.

[Full answer →](https://irbisa.com/topic/ios-behavioral-interview/#q-76402) · [Behavioral Interview](topics/ios-behavioral-interview.md)

### 16. Tell me about an iOS feature you shipped that you are proud of. What did you own?

Choose a feature that used **a platform capability** well, such as a widget, a Live Activity or App Intents, and explain why it mattered to users, what you decided and what moved afterwards.

[Full answer →](https://irbisa.com/topic/ios-behavioral-interview/#q-76403) · [Behavioral Interview](topics/ios-behavioral-interview.md)

### 17. What compensation do you expect for this iOS position?

Quote a **range from research**, not from your current pay, and make clear whether it is gross per year or net per month and that the final figure hinges on the whole package.

[Full answer →](https://irbisa.com/topic/ios-behavioral-interview/#q-76405) · [Behavioral Interview](topics/ios-behavioral-interview.md)

### 18. What is the difference between `some` and `any` in Swift, and how do both relate to generics?

`some Shape` is an opaque type: one specific concrete type that the compiler knows and hides from the reader. `any Shape` is an existential: a box that can hold a value of any conforming type, and a different one each time.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-2602) · [Swift Generics & Advanced](topics/swift-advanced.md)

### 19. Passing a big `Array` around in Swift looks free. How does copy-on-write work, and how would you implement it for your own type?

The underlying buffer is shared between copies until someone writes; before the write, a uniqueness check decides whether the buffer has to be cloned first.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62604) · [Swift Generics & Advanced](topics/swift-advanced.md)

### 20. What is the difference between concurrency and parallelism, and between synchronous and asynchronous execution?

Concurrency means several tasks are in progress at the same time and may interleave; parallelism means they literally execute at the same instant on different CPU cores. Synchronous code makes the caller wait for the result, asynchronous code lets the caller continue and delivers the result later.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62715) · [Swift Concurrency](topics/swift-concurrency.md)

### 21. What is GCD, and what is the difference between serial and concurrent queues and between `sync` and `async` dispatch?

GCD (Grand Central Dispatch) is Apple's library for submitting closures to queues that the system runs on a thread pool it manages. A serial queue runs one item at a time in FIFO order, a concurrent queue may run several at once; `async` returns immediately, `sync` blocks until the closure finishes.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62716) · [Swift Concurrency](topics/swift-concurrency.md)

### 22. How does `async`/`await` work in Swift, and how does it differ from completion handlers?

An `async` function can suspend at each `await` and resume later, and while suspended it holds no thread. Compared with completion handlers, the code reads top to bottom, errors go through `try`/`catch`, and the compiler guarantees the result arrives exactly once.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-2701) · [Swift Concurrency](topics/swift-concurrency.md)

### 23. What is an actor, how does it prevent data races, and why do you `await` its methods from outside?

An actor is a reference type that protects its mutable state by running only one piece of its code at a time on its own serial executor. Code outside the actor cannot touch that state directly; it calls the actor's methods with `await`, because the call may have to wait until the actor is free.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-2703) · [Swift Concurrency](topics/swift-concurrency.md)

### 24. What is the difference between `@State` and `@Binding` in SwiftUI?

`@State` is storage a view owns: SwiftUI keeps the value outside the view struct and re-renders the view when it changes. `@Binding` is a read-write reference to state owned by someone else; the child can change the value but does not store it.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-2901) · [SwiftUI State Management](topics/swiftui-state.md)

### 25. What is the difference between `@Environment`, `@EnvironmentObject` and `@ObservedObject`, and when do you use the environment instead of `init`?

`@ObservedObject` receives an object explicitly through the initializer, while `@EnvironmentObject` and `@Environment` read something injected higher up the tree, so the views in between do not have to pass it along (prop drilling).

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-2903) · [SwiftUI State Management](topics/swiftui-state.md)

### 26. What is MVC in UIKit, and why does it so often turn into a Massive View Controller?

In Apple's MVC the controller mediates between model and view, but `UIViewController` also owns the view's lifecycle, so view code, state, networking and navigation all settle in one class that cannot be tested without a view.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-3202) · [iOS Architecture](topics/ios-architecture.md)

### 27. What is MVVM, what problem does it solve, and how does it look in UIKit and in SwiftUI?

MVVM moves presentation logic and screen state out of the view into a view model that exposes ready-to-show values; the view only binds to them and forwards user events, so the logic can be unit-tested without any UI.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63203) · [iOS Architecture](topics/ios-architecture.md)

### 28. What are the parts of the Core Data stack, and what does each one do?

Four pieces: `NSManagedObjectModel` is the schema, `NSPersistentStoreCoordinator` connects that model to the store files, `NSManagedObjectContext` is the scratchpad where you fetch and change objects, and `NSPersistentContainer` builds and owns all of them.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63418) · [iOS Persistence](topics/ios-persistence.md)

### 29. How do you implement a singleton in Swift, is it thread-safe, and what is wrong with singletons?

A `static let shared` plus a `private init()`. Swift creates a static property lazily and exactly once, even when several threads reach it first. The problem is the design: a singleton is global state that any line can reach, so dependencies hide and tests leak into each other.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75503) · [OOP, SOLID & Design Patterns](topics/ios-design-patterns.md)

### 30. How does ARC work in Swift, and how is it different from a garbage collector?

ARC frees a class instance the moment its strong reference count drops to zero, using retain and release calls the compiler inserts. A garbage collector finds unreachable objects at runtime instead: it costs pauses or background work, but it also frees reference cycles, which ARC never does.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-3501) · [iOS Performance & Memory](topics/ios-performance.md)

## All topics

| Topic | Level | Questions |
|---|---|---:|
| [Swift Basics](topics/swift-basics.md) | Junior | 26 |
| [Swift OOP & Protocols](topics/swift-oop-protocols.md) | Junior | 26 |
| [SwiftUI Fundamentals](topics/swiftui-fundamentals.md) | Junior | 25 |
| [UIKit Fundamentals](topics/uikit-fundamentals.md) | Junior | 25 |
| [Auto Layout & UIKit Layout](topics/uikit-layout.md) | Junior | 25 |
| [Git & Team Workflow](topics/ios-git-workflow.md) | Junior | 18 |
| [Behavioral Interview](topics/ios-behavioral-interview.md) | Junior | 17 |
| [Swift Generics & Advanced](topics/swift-advanced.md) | Middle | 25 |
| [Swift Concurrency](topics/swift-concurrency.md) | Middle | 25 |
| [SwiftUI State Management](topics/swiftui-state.md) | Middle | 24 |
| [iOS Architecture](topics/ios-architecture.md) | Middle | 24 |
| [iOS Networking](topics/ios-networking.md) | Middle | 25 |
| [iOS Persistence](topics/ios-persistence.md) | Middle | 25 |
| [iOS Testing](topics/ios-testing.md) | Middle | 25 |
| [App Lifecycle, Background & Push](topics/ios-lifecycle-background.md) | Middle | 24 |
| [App Extensions & WidgetKit](topics/ios-extensions-widgets.md) | Middle | 20 |
| [Accessibility & Localization](topics/ios-accessibility-localization.md) | Middle | 25 |
| [Collection Views & Diffable Data](topics/uikit-collections.md) | Middle | 23 |
| [GCD & Threading](topics/gcd-threading.md) | Middle | 18 |
| [Security, Privacy & Keychain](topics/ios-security-keychain.md) | Middle | 20 |
| [Live Coding Round in Swift](topics/swift-coding-round.md) | Middle | 18 |
| [StoreKit & App Store Release](topics/ios-storekit-release.md) | Middle | 19 |
| [OOP, SOLID & Design Patterns](topics/ios-design-patterns.md) | Middle | 18 |
| [iOS Performance & Memory](topics/ios-performance.md) | Senior | 25 |
| [Combine](topics/combine.md) | Senior | 24 |
| [Swift 6 Strict Concurrency](topics/swift-strict-concurrency.md) | Senior | 19 |
| [Animations & Core Animation](topics/ios-animations.md) | Senior | 20 |
| [Xcode Build & Modularization](topics/ios-build-modularization.md) | Senior | 22 |
| [SwiftUI Layout, Identity & Rendering](topics/swiftui-layout-rendering.md) | Senior | 21 |
| [Objective-C Runtime & Interop](topics/objc-runtime-interop.md) | Senior | 18 |
| [iOS System Design](topics/ios-system-design.md) | Senior | 16 |

## Practice

Reading answers is not the same as giving one. On [irbisa.com](https://irbisa.com/ios/) you answer first, then check yourself against the answer; the questions you miss come back until they stick.

---

Questions and answers are written and kept up to date at [irbisa.com](https://irbisa.com/). This list is generated from the same source.
