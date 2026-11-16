# Swift 6 Strict Concurrency interview questions

Isolation domains, Sendable, global actors, region isolation, sending, Swift 5 to 6 migration. Senior · 19 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/swift-strict-concurrency/)** · [All iOS topics](../README.md#all-topics)

### 1. What does Swift 6 strict concurrency checking verify, and how is the language mode different from the compiler version?

It proves at compile time that no mutable state can be reached from two isolation domains at once: every value crossing a boundary is `Sendable` or provably handed off. The compiler version decides which checks exist; each target's language mode decides whether they are errors.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67017)

### 2. What is an isolation domain in Swift, and how can a value move from one domain to another?

An isolation domain is a scope whose mutable state only one task touches at a time: each actor instance is one, each global actor such as `@MainActor` is one, and `nonisolated` code belongs to none. A value crosses between them only if it is `Sendable` or handed off with nothing left behind.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67018)

### 3. Why does a class with only `let` properties still fail to be `Sendable`, and when is `@unchecked Sendable` the honest fix?

A class never gets `Sendable` inferred: you write the conformance, and then the class must be `final`, have only immutable stored properties of Sendable types, and no superclass except `NSObject`. `@unchecked Sendable` waives all of that and is honest only when a lock you wrote guards every access.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67003)

### 4. Swift 6 rejects both declarations in `Services`. What is each error saying, and what are the legal shapes for shared mutable state?

```swift
final class Analytics {
  var queue: [String] = []
}

enum Services {
  static var currentUserID: String?     // (1)
  static let analytics = Analytics()    // (2)
}
```

Line (1) is a global variable any thread can write with nothing serialising it. Line (2) is immutable, but it points at a non-Sendable class with `var` storage, so the state behind it is still shared and mutable. Every fix makes the state immutable, isolated or guarded.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67004)

### 5. What does `nonisolated(unsafe)` promise the compiler, and when is it an honest answer rather than a way to silence an error?

It switches off concurrency checking for one declaration and promises that you serialise every access yourself; it adds no lock and no runtime protection. It is honest only when you can write "this is safe because…" next to it and point at the mechanism.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67002)

### 6. Why can a class holding a `Mutex` be `Sendable` without `@unchecked`, and what are the restrictions of `Mutex`?

`Mutex<Value>` (iOS 18+) is a lock that owns the value it protects: the value is reachable only inside `withLock`, and the mutex itself is `Sendable`, so a class that holds one conforms to `Sendable` without `@unchecked`.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67019)

### 7. Find the bug: this `Sendable` stats class compiles clean in Swift 6, yet under load the counts come out lower than the number of `record` calls.

```swift
final class RequestStats: Sendable {
  private let storage = Mutex<[String: Int]>([:])

  var counts: [String: Int] {
    get { storage.withLock { $0 } }
    set { storage.withLock { $0 = newValue } }
  }

  func record(_ host: String) {
    counts[host, default: 0] += 1
  }

  func reset() { counts = [:] }
}
```

`counts[host, default: 0] += 1` goes through the getter and then the setter, so it takes the lock twice: it copies the dictionary under one lock, increments the copy unlocked and writes it back under another. Concurrent calls overwrite each other and increments are lost.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67021)

### 8. What is a global actor, and when would you define your own instead of using `@MainActor` or a plain `actor`?

A global actor is one shared actor instance named by an attribute, so declarations spread across many types share one isolation domain; `@MainActor` is the built-in one. Define your own only for process-wide state touched from several types, like a database.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67009)

### 9. What does `nonisolated` mean on a stored property, a synchronous method and an `async` method, and does it move work off the actor?

`nonisolated` takes a declaration out of its actor's isolation domain: callers reach it without `await`, and it may not touch isolated state. It is a statement about checking, not scheduling; none of the three forms moves work to a background thread by itself.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67012)

### 10. Does this compile in Swift 6 language mode, and what changes if you delete the `print` line?

```swift
final class Report {            // not Sendable
  var title = ""
}

actor Archive {
  private var reports: [Report] = []
  func store(_ r: Report) { reports.append(r) }
}

func archiveWeekly(_ archive: Archive) async {
  let report = Report()
  report.title = "Weekly"
  await archive.store(report)
  print(report.title)
}
```

As written it fails with *"sending 'report' risks causing data races"*, because `report` is read after being handed to the actor. Delete the `print` and it compiles: region-based isolation (SE-0414) proves nothing else can reach `report` after the call.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67005)

### 11. What can a `sending` parameter express that `@Sendable` cannot, and why do `Task.init` and `addTask` take `sending` closures?

`@Sendable` restricts a closure's captures to Sendable values, because it may run anywhere any number of times. `sending` says nothing about the type: it transfers one value, so the caller gives it up and the callee receives it disconnected.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67006)

### 12. After the target turned on Approachable Concurrency, scrolling stutters while this model decodes images. Why, and how do you fix it?

```swift
@MainActor
final class GalleryModel {
  var thumbnails: [UIImage] = []

  func load(_ files: [Data]) async {
    for data in files {
      if let image = await Self.decode(data) { thumbnails.append(image) }
    }
  }

  nonisolated static func decode(_ data: Data) async -> UIImage? {
    UIImage(data: data)?.preparingForDisplay()
  }
}
```

With `NonisolatedNonsendingByDefault` (SE-0461, Swift 6.2) a `nonisolated async` function runs on its caller's actor, so `decode` now runs on the main actor and decodes every image on the main thread. Mark it `@concurrent` to send it back to the global pool.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67020)

### 13. How do you migrate a large app from Swift 5 to the Swift 6 language mode without freezing feature work?

Target by target, leaves of the dependency graph first: in Swift 5 mode turn on complete checking so every violation is a warning, drive the warnings to zero, then flip that target to Swift 6. Every step ends in a green build you can ship.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67001)

### 14. What do Approachable Concurrency and default `MainActor` isolation change in a target, and why keep them off in a shared framework?

Default `MainActor` isolation (SE-0466) makes every unannotated declaration `@MainActor`; Approachable Concurrency turns on a bundle of upcoming features that suit that model. In an app that removes most Sendable errors; in a framework it changes your public API's isolation for every client.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67013)

### 15. Your `@MainActor` view model must conform to `Equatable` and the compiler rejects the conformance. Why, and what are your options?

`Equatable.==` is a nonisolated requirement, so any code may call it synchronously from any thread, and a main-actor witness cannot honour that. The fixes: a `nonisolated` witness, an isolated conformance (Swift 6.2), or a protocol that fits the isolation.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67007)

### 16. What does `@preconcurrency` do on an import and on a protocol conformance, and what risk remains after the build goes green?

On an import it stops diagnostics about missing `Sendable` on that module's types; on a conformance it lets an isolated type satisfy a nonisolated requirement, swapping the compile error for a runtime trap. Neither removes a race: one hides it, the other makes it a crash.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67008)

### 17. A vendor callback is documented to arrive on the main thread but the method is `nonisolated`. When is `MainActor.assumeIsolated` right, and when does it crash?

`assumeIsolated` checks at runtime that you are already on the main actor, then runs its closure synchronously with main-actor isolation; it never switches threads. It is right when main-thread delivery is guaranteed, and it traps, in release builds too, when it is not.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67011)

### 18. A vendor's Objective-C SDK calls your delegate on arbitrary threads. How do you bring it into a Swift 6 module without `@unchecked Sendable` everywhere?

Put one small nonisolated object between the SDK and your code: it is the only thing the vendor's threads touch, it owns no mutable state, and it forwards each event into an `AsyncStream` that an isolated model consumes. The boundary is crossed in one place, with Sendable values.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67014)

### 19. Your target compiles clean in Swift 6 mode, yet Thread Sanitizer reports a data race. What can the compiler not see, and how do you track the race down?

Both are right: Swift 6 proves that no unprotected Swift value crosses an isolation boundary in code it was allowed to check, while TSan watches real memory accesses at runtime, including code the compiler was told to skip or never saw. The race lives in that blind spot.

[Full answer →](https://irbisa.com/topic/swift-strict-concurrency/#q-67015)
