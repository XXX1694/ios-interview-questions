# GCD & Threading interview questions

Queues, QoS, DispatchGroup and semaphores, the main.sync deadlock, Operation, bridging to async/await. Middle · 18 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/gcd-threading/)** · [All iOS topics](../README.md#all-topics)

### 1. What is `Thread` (`NSThread`), and why do iOS apps almost never create threads by hand?

`Thread` is Foundation's wrapper around a POSIX thread: you create it, start it and own its lifetime. Apps rarely do that because GCD and Swift Concurrency keep a pool of reusable threads and decide how many to run, which is cheaper and harder to get wrong.

[Full answer →](https://irbisa.com/topic/gcd-threading/#q-71517)

### 2. On which thread does a `URLSession` completion handler run, and how do you update the UI from it?

On a background thread: the session calls completion handlers on its delegate queue, which for `URLSession.shared` is a private serial `OperationQueue`, not the main queue. Parse the data there and hop to the main queue with `DispatchQueue.main.async` only to touch the UI.

[Full answer →](https://irbisa.com/topic/gcd-threading/#q-71518)

### 3. How do you run code after a delay with GCD, and how do you cancel it before it runs?

`DispatchQueue.main.asyncAfter(deadline: .now() + 0.3) { … }` schedules a closure, but a plain closure cannot be cancelled. Wrap the work in a `DispatchWorkItem`, schedule the item, and call `cancel()` on it when it is no longer needed.

[Full answer →](https://irbisa.com/topic/gcd-threading/#q-71519)

### 4. What is Thread Sanitizer, and how is it different from the Main Thread Checker?

Thread Sanitizer (TSan) is a runtime tool that reports data races: two threads touching the same memory without synchronisation, at least one of them writing. The Main Thread Checker reports only UIKit and AppKit calls made off the main thread.

[Full answer →](https://irbisa.com/topic/gcd-threading/#q-71520)

### 5. What does this code print when `run()` is called on the main thread, and on which thread does each block run?

```swift
// run() is called on the main thread, for example from viewDidLoad
func run() {
  let serial = DispatchQueue(label: "com.app.serial")
  DispatchQueue.main.async { print("A", Thread.isMainThread) }
  serial.sync { print("B", Thread.isMainThread) }
  DispatchQueue.global().sync { print("C", Thread.isMainThread) }
  serial.async { print("D", Thread.isMainThread) }
  serial.sync { print("E", Thread.isMainThread) }
  print("F")
}
```

It prints `B true`, `C true`, `D false`, `E true`, `F` and finally `A true`. `sync` runs the block on the calling thread, here the main thread, even when the queue is a different one; `async` hands the block to a worker thread; `main.async` waits until `run()` returns.

[Full answer →](https://irbisa.com/topic/gcd-threading/#q-71533)

### 6. How does a dispatch barrier turn a concurrent queue into a reader-writer lock, and when does the barrier flag do nothing?

A block submitted with `.barrier` waits until every block submitted before it has finished, runs alone, and only then lets later blocks start. Reads go through plain `sync` and run in parallel; writes go through barriers and run exclusively.

[Full answer →](https://irbisa.com/topic/gcd-threading/#q-71521)

### 7. Find the bug: this thread-safe cache crashes the first time `image(for:orInsert:)` is called.

```swift
final class ImageCache: @unchecked Sendable {
  private let queue = DispatchQueue(label: "com.app.image-cache")
  private var storage: [URL: UIImage] = [:]
  func image(for url: URL) -> UIImage? {
    queue.sync { storage[url] }
  }

  func image(for url: URL, orInsert make: () -> UIImage) -> UIImage {
    queue.sync {
      if let cached = image(for: url) { return cached }
      let image = make()
      storage[url] = image
      return image
    }
  }
}
```

`image(for:orInsert:)` runs inside `queue.sync` and then calls `image(for:)`, which calls `queue.sync` on the same serial queue. The queue waits for itself: a deadlock that libdispatch detects and turns into a crash.

[Full answer →](https://irbisa.com/topic/gcd-threading/#q-71522)

### 8. Find the bug: `loadAvatars` sometimes never calls its completion.

```swift
// api.fetchAvatar(id:completion:) is declared with a @MainActor completion, called on the main queue
@MainActor
func loadAvatars(ids: [String], completion: @escaping @MainActor ([UIImage]) -> Void) {
  let group = DispatchGroup()
  var images: [UIImage] = []
  for id in ids {
    group.enter()
    api.fetchAvatar(id: id) { result in
      guard case .success(let image) = result else { return }
      images.append(image)
      group.leave()
    }
  }
  group.notify(queue: .main) { completion(images) }
}
```

On the failure path the callback returns from the `guard` before `group.leave()`, so the group's counter never reaches zero and `notify` never fires. One failed request out of twenty and the screen waits forever, with no crash and no log.

[Full answer →](https://irbisa.com/topic/gcd-threading/#q-71523)

### 9. Find the bug: this retry timer is scheduled from a background queue and never fires.

```swift
final class UploadRetrier: @unchecked Sendable {
  private let queue = DispatchQueue(label: "com.app.upload")
  private var retryTimer: Timer?

  // Called by the upload code when a request fails
  func uploadDidFail() {
    queue.async {
      self.retryTimer?.invalidate()
      self.retryTimer = Timer.scheduledTimer(withTimeInterval: 5, repeats: false) { [weak self] _ in
        self?.retry()
      }
    }
  }

  private func retry() { /* start the upload again */ }
}
```

`Timer.scheduledTimer` adds the timer to the run loop of the current thread, and here that is a GCD worker thread whose run loop nobody runs. The timer is created, sits in that run loop and never fires: no error, no log.

[Full answer →](https://irbisa.com/topic/gcd-threading/#q-71534)

### 10. What does `DispatchQueue.concurrentPerform` do, and why is it better than `global().async` in a loop for CPU-heavy work?

`concurrentPerform(iterations:execute:)` runs a closure N times in parallel on about as many threads as there are cores, and returns only when every iteration has finished. A loop of `global().async` instead submits N independent blocks with no limit and nothing to wait on.

[Full answer →](https://irbisa.com/topic/gcd-threading/#q-71524)

### 11. Which locks do you use in Swift (`NSLock`, `NSRecursiveLock`, `OSAllocatedUnfairLock`, `Mutex`), and why not a raw `os_unfair_lock`?

On iOS 18+ use `Mutex`, on iOS 16–17 `OSAllocatedUnfairLock`: both own the state they protect and are the fastest options. `NSLock` is the classic fallback, `NSRecursiveLock` is for code that re-enters the same lock, and a raw `os_unfair_lock` stored in a Swift property has no address guarantee.

[Full answer →](https://irbisa.com/topic/gcd-threading/#q-71525)

### 12. Which of these are thread-safe in Swift: a `static let`, a `lazy var`, an `Array` you append to, an `Int` counter?

Only the `static let`: globals and static stored properties are initialised lazily exactly once, even when many threads reach them first at the same time. A `lazy var`, appending to a shared `Array` and `count += 1` are all data races when two threads do them at once.

[Full answer →](https://irbisa.com/topic/gcd-threading/#q-71526)

### 13. How do you chain work with `Operation` dependencies, and what happens to a dependent operation when its dependency is cancelled?

`b.addDependency(a)` keeps `b` from becoming ready until `a` has finished, even across queues. A cancelled operation still finishes, so its dependents start anyway; each operation has to check whether the operations it waits for were cancelled or failed.

[Full answer →](https://irbisa.com/topic/gcd-threading/#q-71527)

### 14. In a Swift 6 codebase built on async/await, when is GCD still the right tool?

When an API asks for a queue, when work blocks a thread and cannot be made async, and for timers and event sources. Shared state and ordinary background work belong to actors and tasks; GCD stays at the edges.

[Full answer →](https://irbisa.com/topic/gcd-threading/#q-71528)

### 15. How do you write an asynchronous `Operation` subclass, and why does the queue get stuck if `isFinished` is not KVO-compliant?

Override `start()` instead of `main()`, report `isExecuting` and `isFinished` yourself, and post KVO notifications for both when they change. The queue learns that an operation is done only from the `isFinished` KVO change; without it the operation stays in the queue forever.

[Full answer →](https://irbisa.com/topic/gcd-threading/#q-71529)

### 16. What QoS does a block submitted with `async` run at, and how does GCD resolve priority inversion (and when can it not)?

By default `async` captures the submitting thread's QoS, and the block runs at it on a queue without its own QoS; a queue with a QoS uses its own. Priority inversion (high-QoS work waiting on low-QoS work) is resolved automatically only when the system knows whom the waiter is waiting for.

[Full answer →](https://irbisa.com/topic/gcd-threading/#q-71530)

### 17. Instruments shows 60+ threads because every object has its own serial queue. Why does that happen, and how do target queues fix it?

Independent serial queues can all run at the same time, each on its own worker thread, and custom serial queues may get a new thread even when every core is busy. Pointing them at one serial target queue per subsystem makes the whole group run one block at a time, on a bounded set of threads.

[Full answer →](https://irbisa.com/topic/gcd-threading/#q-71531)

### 18. Is `DispatchQueue.main.async` the same as running on `@MainActor`, and what changes when you migrate GCD code to Swift Concurrency?

At runtime they meet in the same place: the main actor's executor is the main dispatch queue, so both run on the main thread. The difference is the compiler and timing. `@MainActor` is checked statically and runs immediately when you are already on it, while `main.async` always enqueues for later.

[Full answer →](https://irbisa.com/topic/gcd-threading/#q-71532)
