# Swift Concurrency interview questions

async/await, Task, actors, structured concurrency, AsyncSequence. Middle · 25 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/swift-concurrency/)** · [All iOS topics](../README.md#all-topics)

### 1. What is the difference between concurrency and parallelism, and between synchronous and asynchronous execution?

Concurrency means several tasks are in progress at the same time and may interleave; parallelism means they literally execute at the same instant on different CPU cores. Synchronous code makes the caller wait for the result, asynchronous code lets the caller continue and delivers the result later.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62715)

### 2. What is GCD, and what is the difference between serial and concurrent queues and between `sync` and `async` dispatch?

GCD (Grand Central Dispatch) is Apple's library for submitting closures to queues that the system runs on a thread pool it manages. A serial queue runs one item at a time in FIFO order, a concurrent queue may run several at once; `async` returns immediately, `sync` blocks until the closure finishes.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62716)

### 3. How does `async`/`await` work in Swift, and how does it differ from completion handlers?

An `async` function can suspend at each `await` and resume later, and while suspended it holds no thread. Compared with completion handlers, the code reads top to bottom, errors go through `try`/`catch`, and the compiler guarantees the result arrives exactly once.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-2701)

### 4. How do you call an `async` function from synchronous code, and can you call synchronous code from an `async` function?

From synchronous code you start a `Task { }` and `await` inside it; the synchronous function returns immediately and never sees the result directly. The other direction needs nothing: an `async` function can call any synchronous function, as long as that call does not block for long.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62723)

### 5. What does this code print when `test()` runs on the main thread, and what changes if it runs on a background queue?

```swift
func test() {
  print("A")
  DispatchQueue.main.async { print("B") }
  print("C")
  DispatchQueue.main.sync { print("D") }
  print("E")
}
test()   // called on the main thread
```

On the main thread it prints `A`, `C` and then gets stuck: `main.sync` waits for the main queue to run `D`, but the main queue is busy running `test()` itself, which is waiting; this is a deadlock that libdispatch usually turns into a crash.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62717)

### 6. What are the Quality of Service classes, and how do they affect scheduling?

QoS tells the system how much the work matters to the user right now, and the system uses it to set thread priority, share CPU time and choose between performance and efficiency cores. Lower QoS work gets less CPU and can be delayed for a long time under load.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62718)

### 7. What is `DispatchGroup`, and when do you use it?

`DispatchGroup` counts a set of asynchronous jobs and tells you when all of them have finished; the classic case is firing several requests in parallel and updating the UI once every one has returned.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62719)

### 8. What is the difference between GCD and `OperationQueue`, and when would you pick `OperationQueue`?

`OperationQueue` is an Objective-C layer over GCD that wraps work in `Operation` objects and adds dependencies, a concurrency limit, cancellation and observable state; GCD just runs closures. Pick it when the work is a graph of steps to order, limit or cancel as a whole.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62720)

### 9. What is the difference between a race condition, a data race and a deadlock, and how do you prevent each?

A data race is two threads touching the same memory at once, at least one writing, with no synchronisation: undefined behaviour. A race condition is a logic bug where the result depends on timing even with synchronised access. A deadlock is threads waiting on each other forever.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62721)

### 10. What is a semaphore, how does it differ from a mutex, and when is `DispatchSemaphore` a bad idea?

A semaphore is a counter that lets up to N threads through and blocks the rest; a mutex lets exactly one thread into a critical section and has an owner: only the thread that locked it may unlock it.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62722)

### 11. What is structured concurrency, and when do you use `async let` versus a task group?

Structured concurrency binds child tasks to the scope that created them: they cannot outlive it, cancellation flows down, errors flow up, and the parent waits for every child. `async let` suits a fixed number of children; a task group suits a number known only at runtime.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-2702)

### 12. What is the difference between `Task { }` and `Task.detached`, and how is a task’s priority decided?

`Task { }` inherits the actor isolation, priority and task-local values of the code that creates it; `Task.detached` inherits none of them and runs nonisolated with its own priority. Neither is a child task, so cancelling the code that created them does not cancel them.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62713)

### 13. How does task cancellation work in Swift Concurrency?

Cancellation is cooperative: `cancel()` only sets a flag on the task and all its children, and nothing stops until code checks that flag or awaits an API that checks it for you.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-2706)

### 14. Find the bug: after the screen calls `stop()`, the ticker keeps hitting the API and CPU usage jumps.

```swift
// api: a Sendable global whose fetchPrice() is async throws -> Decimal
@MainActor
final class PriceTicker {
  private var task: Task<Void, Never>?
  private(set) var price: Decimal = 0

  func start() {
    task = Task {
      while true {
        if let p = try? await api.fetchPrice() { price = p }
        try? await Task.sleep(for: .seconds(5))
      }
    }
  }
  func stop() { task?.cancel() }
}
```

`try?` swallows the `CancellationError` that tells the loop to stop. After `stop()`, a cancelled `Task.sleep` throws at once, the error becomes `nil`, and `while true` spins: a hot loop that pins the CPU and keeps firing requests.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62725)

### 15. `loadUser()` throws after 0.1 s while `loadPosts()` takes 10 s: what happens to the posts request, and when does `loadScreen()` return?

```swift
func loadScreen() async throws -> Screen {
  async let posts = api.loadPosts()   // takes 10 s
  async let user = api.loadUser()     // throws after 0.1 s
  let u = try await user
  let p = try await posts
  return Screen(user: u, posts: p)
}
```

The error surfaces at `try await user`, and before the function exits Swift implicitly cancels `posts` and waits for it, so `loadScreen()` throws only once the posts task has wound down: almost immediately if `loadPosts` honours cancellation, after the full 10 s if it ignores it.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62707)

### 16. You need to download 500 images but no more than 5 at a time: how do you cap concurrency in a task group?

Seed the group with 5 children, then every time one finishes (`next()` returns), add the next one, so the group never has more than 5 in flight.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62706)

### 17. How do you add a timeout to an `await` call when the API has no timeout parameter?

Race the operation against `Task.sleep` in a throwing task group: take whichever child finishes first and cancel the other. If the sleep wins, it throws your timeout error.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62708)

### 18. What is an actor, how does it prevent data races, and why do you `await` its methods from outside?

An actor is a reference type that protects its mutable state by running only one piece of its code at a time on its own serial executor. Code outside the actor cannot touch that state directly; it calls the actor's methods with `await`, because the call may have to wait until the actor is free.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-2703)

### 19. What does `@MainActor` guarantee, why not mark everything with it, and how do you move work off it?

`@MainActor` is a global actor whose executor is the main thread: annotated code is guaranteed to run there, and the compiler makes callers from other isolation domains `await` it. It is for UI state, not for work; everything it covers runs one piece at a time on the thread that draws your UI.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62701)

### 20. What is `Sendable`, and which types conform to it automatically?

`Sendable` is a marker protocol saying a value can cross concurrency domains (between tasks, into or out of an actor) without causing a data race. In the Swift 6 language mode the compiler checks it at every such crossing and rejects code that would share unprotected mutable state.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-2705)

### 21. How do you wrap a completion-handler API in `async`/`await`, and what is the difference between checked and unsafe continuations?

Call `withCheckedThrowingContinuation`, start the legacy call inside it, and resume the continuation exactly once from the callback; the awaiting task stays suspended until then. A checked continuation verifies that "exactly once" at runtime, an unsafe one does not.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62702)

### 22. What are `AsyncSequence` and `AsyncStream`, and when do you use them?

`AsyncSequence` is a sequence whose elements arrive over time and is consumed with `for await`; `AsyncStream` is the ready-made one you build yourself to turn callbacks, delegates or notifications into such a sequence.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-2704)

### 23. This actor still downloads the same image twice when two views ask for it at once: why, and how do you fix it?

```swift
// downloader: a Sendable global dependency whose fetch(_:) is async
actor ImageCache {
  private var cache: [URL: UIImage] = [:]

  func image(for url: URL) async throws -> UIImage {
    if let cached = cache[url] { return cached }
    let image = try await downloader.fetch(url)   // suspension point
    cache[url] = image
    return image
  }
}
```

Actors are reentrant: at the `await` the actor is released, the second call runs, finds the cache still empty and starts its own download. Isolation protects state between suspension points, not across them.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62724)

### 24. What is the cooperative thread pool, and why is blocking it with `DispatchSemaphore.wait()` or `Thread.sleep` a bug?

Swift Concurrency runs tasks on a cooperative pool with about one thread per CPU core and assumes every task either runs or suspends. A thread blocked in `wait()` or `Thread.sleep` is lost to the pool, and nothing replaces it.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62703)

### 25. How do you protect shared mutable state (actor, lock or serial queue) and how do you choose between them?

Use an actor when callers are async or the critical section must `await`; a lock (`Mutex`, `OSAllocatedUnfairLock`) when access is short, synchronous and hot; a serial `DispatchQueue` mainly inside GCD code. All three serialise access; they differ in whether callers suspend or block, and in cost.

[Full answer →](https://irbisa.com/topic/swift-concurrency/#q-62709)
