# OOP, SOLID & Design Patterns interview questions

OOP and SOLID in Swift, DRY, KISS and YAGNI, coupling and cohesion, dependency injection, GoF patterns, delegate, target-action, observer and coordinator, anti-patterns. Middle · 18 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/ios-design-patterns/)** · [All iOS topics](../README.md#all-topics)

### 1. What do DRY, KISS and YAGNI mean, and when does removing duplication make code worse?

DRY: every piece of knowledge has one authoritative place. KISS: pick the simplest design that solves today's problem. YAGNI: do not build for requirements you do not have yet. DRY backfires when you merge code that only looks alike but changes for different reasons.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75501)

### 2. What are coupling and cohesion, and how do they relate to the Single Responsibility Principle?

Cohesion is how strongly the parts of one type or module belong together; coupling is how much one type depends on another's details. Aim for high cohesion and low coupling, so a change lands in one place instead of rippling through others.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75502)

### 3. How do you implement a singleton in Swift, is it thread-safe, and what is wrong with singletons?

A `static let shared` plus a `private init()`. Swift creates a static property lazily and exactly once, even when several threads reach it first. The problem is the design: a singleton is global state that any line can reach, so dependencies hide and tests leak into each other.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75503)

### 4. How do you implement the delegate pattern in Swift, and why do Cocoa delegate methods pass the sender and use `will`, `did` and `should`?

Declare a class-bound protocol for the events, give the delegating object a `weak var delegate`, and call it at the right moments. Cocoa's naming makes one delegate reusable and expressive: the sender comes first, `should` methods ask permission, `will` and `did` report before and after.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75504)

### 5. Which SOLID principle does `CachedFeedLoader` break, and what happens to the `feed(from:)` caller?

```swift
protocol FeedLoader {
  func load(completion: @escaping (Result<[Post], Error>) -> Void)  // contract: exactly once
}
struct CachedFeedLoader: FeedLoader {
  let cache: FeedCache
  let remote: FeedLoader
  func load(completion: @escaping (Result<[Post], Error>) -> Void) {
    completion(.success(cache.posts))     // cached posts at once
    remote.load(completion: completion)   // then the fresh ones
  }
}
func feed(from loader: FeedLoader) async throws -> [Post] {
  try await withCheckedThrowingContinuation { continuation in
    loader.load { continuation.resume(with: $0) }
  }
}
```

Liskov substitution: callers of `FeedLoader` rely on its promise of exactly one completion, and `CachedFeedLoader` calls it twice. The checked continuation in `feed(from:)` then stops the app with `SWIFT TASK CONTINUATION MISUSE: feed(from:) tried to resume its continuation more than once`.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75505)

### 6. What does Interface Segregation look like in Swift, and how can a protocol's default implementations hide a violation?

No client should depend on methods it does not use. In Swift that means small role protocols named after what one caller needs, combined with `&` where a caller needs two, instead of one `UserService` that every screen and every test double has to carry.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75506)

### 7. What is the difference between dependency injection and dependency inversion, and which module should own the protocol?

Dependency injection is a technique: an object receives its collaborators instead of creating them. Dependency inversion is a design rule: high-level and low-level code both depend on an abstraction that the high-level side owns. You can inject a concrete client and still have no inversion.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75507)

### 8. What is the difference between Factory Method and Abstract Factory, and where do you meet each in iOS code?

A factory method is one overridable method that creates an object, so a subclass or conformer decides the concrete type. An abstract factory is an object with several such methods that build a family of related objects, swapped as a whole: live screens for stub screens, for example.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75508)

### 9. Do you need the Builder pattern in Swift, given default arguments and memberwise initializers?

Rarely in its classic form: default arguments and memberwise initializers already solve the telescoping-constructor problem a Java builder exists for. What remains in Swift is a configuration value you fill in and hand over, a fluent chain that returns modified copies, and result builders for DSLs.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75509)

### 10. What is the difference between Adapter and Facade, and how do you wrap a third-party SDK so it does not leak into your features?

An adapter makes one existing type fit an interface your code expects; a facade puts one simple interface in front of a whole subsystem. Wrapping an SDK is usually both: a facade with the few calls the app needs, and adapter code behind it that converts the vendor's types, callbacks and errors.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75510)

### 11. What is the Decorator pattern in Swift, how is it different from an extension or a subclass, and how does a Proxy differ from it?

A decorator conforms to the same protocol as the object it wraps and adds behaviour around each call, so caching, logging or retry stack per instance. A proxy has the same shape but another job: it controls access to the real object, creating it lazily, guarding it or forwarding across a boundary.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75511)

### 12. A delegate has a single listener. How do you notify several observers without keeping them alive?

Keep the observers in a collection of weak references, such as `NSHashTable.weakObjects()`, and loop over a snapshot when something changes. The subject never keeps a screen alive, and an observer that has been deallocated simply drops out instead of receiving calls.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75512)

### 13. What is Chain of Responsibility, and how can a deeply nested cell reach its view controller through the responder chain instead of a delegate?

A request travels along a chain of handlers until one handles it, and the sender never learns which one did. UIKit's responder chain is the textbook case: a cell sends a request upward, and the first view, controller or app object able to handle it does, with no delegate wiring.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75513)

### 14. What is the Strategy pattern, how do you express it in Swift, and how is it different from the State pattern?

Strategy puts each variant of an algorithm behind one interface, and the caller picks or swaps the variant at runtime, so the code that uses it never switches over the variants. In Swift a strategy is a small protocol, or just a closure when one function is all it is.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75514)

### 15. Find the bug: the user taps Stop while a video is still buffering, and a moment later it starts playing anyway.

```swift
@MainActor final class PlayerModel {
  private(set) var isBuffering = false
  private(set) var isPlaying = false
  private let engine: PlaybackEngine
  init(engine: PlaybackEngine) { self.engine = engine }

  func playTapped(_ url: URL) { isBuffering = true; engine.load(url) }
  func stopTapped() { isPlaying = false; engine.pause() }

  // The engine calls this once enough of the video has arrived
  func bufferingDidFinish() {
    isBuffering = false
    isPlaying = true
    engine.start()
  }
}
```

`stopTapped()` never ends the buffering, and `bufferingDidFinish()` starts playback without checking what the user wants now. When the engine's callback arrives after Stop, it sets `isPlaying = true` and calls `engine.start()`: a late event overrides the user's last action.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75515)

### 16. What is the Command pattern, and how does `UndoManager` use it to give an app undo and redo?

Command turns a request into an object that can be stored, queued, logged or reversed later. `UndoManager` is built on it: each change registers a closure that reverses it, and the undo and redo stacks are simply stored commands.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75516)

### 17. What is the wrong abstraction, how do you recognise one, and how do you back out of it without a rewrite?

An abstraction is wrong when its callers must pass flags or special cases to get their own behaviour out of it. Back out by inlining it into each caller, deleting what that caller never uses, and only then extracting what is genuinely shared: duplication is cheaper than the wrong abstraction.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75517)

### 18. Which anti-patterns do you look for in an inherited iOS codebase, and how do you refactor out of a god object, singletons everywhere and a `BaseViewController`?

God objects, singletons used as hidden dependencies and a deep `BaseViewController` are the first three I look for. Each hides dependencies or couples unrelated features, and each has an incremental way out that needs no rewrite: move one responsibility at a time behind a seam.

[Full answer →](https://irbisa.com/topic/ios-design-patterns/#q-75518)
