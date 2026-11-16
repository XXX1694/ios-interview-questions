# Combine interview questions

Publishers, Subscribers, operators, Subjects, bridging with async/await. Senior · 24 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/combine/)** · [All iOS topics](../README.md#all-topics)

### 1. What is Combine, and how do `Publisher`, `Subscriber` and operators fit together?

Combine is Apple's reactive framework (iOS 13+): a `Publisher` emits values over time followed by one completion, operators transform that stream, and a `Subscriber` at the end requests and receives the values.

[Full answer →](https://irbisa.com/topic/combine/#q-3601)

### 2. What is the difference between `PassthroughSubject` and `CurrentValueSubject`?

`PassthroughSubject` forwards each value only to whoever is subscribed at that moment and keeps nothing; `CurrentValueSubject` stores the latest value, exposes it as `.value` and hands it to every new subscriber immediately.

[Full answer →](https://irbisa.com/topic/combine/#q-3602)

### 3. What is the difference between `debounce` and `throttle`?

`debounce` waits until the upstream has been quiet for the given interval and then emits the last value; `throttle` emits at most one value per interval while values keep arriving.

[Full answer →](https://irbisa.com/topic/combine/#q-63618)

### 4. What is `AnyCancellable`, and how do you manage the lifetime and memory of a subscription?

`AnyCancellable` is the token a subscription returns: it cancels the subscription when you call `cancel()` or when it is deallocated, so a pipeline lives exactly as long as someone holds that token.

[Full answer →](https://irbisa.com/topic/combine/#q-3606)

### 5. Why does this view model never print `deinit`, and how do `assign(to:on:)` and `assign(to:)` differ?

```swift
final class ProfileViewModel {
  @Published var title = ""
  private var bag = Set<AnyCancellable>()

  init(user: AnyPublisher<User, Never>) {
    user
      .map(\.name)
      .assign(to: \.title, on: self)
      .store(in: &bag)
  }

  deinit { print("deinit") }
}
```

`assign(to:on:)` keeps a strong reference to the object it writes to, and the returned cancellable is stored in that same object, so `self → bag → subscription → self` is a retain cycle that lasts as long as the upstream does not finish. `assign(to: &$title)` ties the lifetime to the property instead and creates no cycle.

[Full answer →](https://irbisa.com/topic/combine/#q-63615)

### 6. Find the bug: after a few trips to a post and back, every feed update is rendered and logged several times.

```swift
final class FeedViewController: UIViewController {
  private let viewModel = FeedViewModel()
  private var bag = Set<AnyCancellable>()

  override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    viewModel.$posts
      .receive(on: DispatchQueue.main)
      .sink { [weak self] posts in
        self?.render(posts)
        Analytics.log("feed_shown", count: posts.count)
      }
      .store(in: &bag)
  }
}
```

The subscription is created in `viewWillAppear`, which runs every time the screen comes back, and each new one is added to `bag` while the old ones stay alive. After three visits three sinks run, so every update renders and logs `feed_shown` three times.

[Full answer →](https://irbisa.com/topic/combine/#q-63620)

### 7. What does `eraseToAnyPublisher()` do, and why do APIs return `AnyPublisher`?

`eraseToAnyPublisher()` wraps a publisher in `AnyPublisher<Output, Failure>` and hides the nested operator types, so an API exposes only what it emits and how it fails, not how the chain is built.

[Full answer →](https://irbisa.com/topic/combine/#q-63616)

### 8. What is the difference between `merge`, `combineLatest` and `zip`?

`merge` interleaves values of the same type from several publishers; `combineLatest` emits a tuple of the latest value from each input whenever any input changes; `zip` pairs values by position and emits only when every input has a new one.

[Full answer →](https://irbisa.com/topic/combine/#q-3603)

### 9. What is the difference between `flatMap` and `switchToLatest`, and which one does a search field need?

`flatMap` keeps every inner publisher alive and merges their results, so a slow old request can overwrite a newer one; `map` followed by `switchToLatest` cancels the previous inner publisher when a new value arrives, which is what a search field needs.

[Full answer →](https://irbisa.com/topic/combine/#q-63601)

### 10. What does this code print, and how do you keep the stream alive after an error?

```swift
let subject = PassthroughSubject<Int, Never>()
let cancellable = subject
  .tryMap { value -> Int in
    if value == 2 { throw URLError(.badURL) }
    return value * 10
  }
  .replaceError(with: -1)
  .sink { print($0) }

subject.send(1)
subject.send(2)
subject.send(3)
```

It prints `10` and `-1`: the thrown error is a completion, so `replaceError` emits one last value and the stream ends, and `3` never arrives. To survive an error, catch it inside a `flatMap`.

[Full answer →](https://irbisa.com/topic/combine/#q-63606)

### 11. What are hot and cold publishers, and which Combine publishers are which?

A cold publisher starts its work separately for each subscriber at the moment of subscription; a hot one produces values on its own schedule whether or not anyone listens, so a late subscriber sees only what comes after it joins.

[Full answer →](https://irbisa.com/topic/combine/#q-63617)

### 12. Find the bug: after one "Saved" toast, every new screen the user opens shows the same toast again.

```swift
@MainActor final class ToastCenter {
  static let shared = ToastCenter()
  let messages = CurrentValueSubject<String?, Never>(nil)
  func show(_ text: String) { messages.send(text) }
}

class BaseViewController: UIViewController {
  private var bag = Set<AnyCancellable>()
  override func viewDidLoad() {
    super.viewDidLoad()
    ToastCenter.shared.messages
      .compactMap { $0 }
      .sink { [weak self] in self?.presentToast($0) }
      .store(in: &bag)
  }
}
```

`CurrentValueSubject` stores the last value and sends it to every new subscriber, so each screen that subscribes in `viewDidLoad` gets the old message at once and shows the toast again. A one-shot event needs `PassthroughSubject`, which stores nothing, and only one screen must present it.

[Full answer →](https://irbisa.com/topic/combine/#q-63621)

### 13. What is `Future` in Combine, and why is it usually wrapped in `Deferred`?

`Future` is a publisher that delivers exactly one value or one error through its `promise` callback; it runs its closure as soon as it is created and caches the result, so `Deferred` is needed to make it lazy and repeatable.

[Full answer →](https://irbisa.com/topic/combine/#q-63607)

### 14. What is the difference between `receive(on:)` and `subscribe(on:)`?

`receive(on:)` sets the scheduler on which values and the completion are delivered to everything below it; `subscribe(on:)` sets where the upstream receives the subscription, requests and cancellation, which is where the work starts, and does not control delivery (a synchronous upstream emits on that queue anyway).

[Full answer →](https://irbisa.com/topic/combine/#q-63608)

### 15. What does this code print, and why does the sink see the old value of `items`?

```swift
final class Cart: ObservableObject {
  @Published var items: [String] = []
  private var bag = Set<AnyCancellable>()

  init() {
    $items
      .sink { [weak self] newItems in
        print(newItems.count, self?.items.count ?? -1)
      }
      .store(in: &bag)
  }
}

let cart = Cart()
cart.items = ["apple", "pear"]
```

It prints `0 0` and then `2 0`: `$items` emits the current value on subscription, and `@Published` publishes from `willSet`, so the sink runs before the property is written and `self.items` still holds the old array.

[Full answer →](https://irbisa.com/topic/combine/#q-63609)

### 16. Which Combine publishers do Foundation and UIKit give you, and how do you get one for a `UIButton`?

Apple ships a short list (`NotificationCenter`, KVO, `Timer`, `URLSession` and your own `@Published` properties), and there is no built-in publisher for `UIControl`, so for a button you wrap `UIAction` in a subject yourself.

[Full answer →](https://irbisa.com/topic/combine/#q-63605)

### 17. Why does a `Timer.publish` countdown freeze while the user scrolls, and how do you fix it?

The timer was scheduled in `RunLoop.Mode.default`, and while a finger drags a scroll view the main run loop runs in tracking mode, so timers in the default mode do not fire; schedule it with `in: .common` instead.

[Full answer →](https://irbisa.com/topic/combine/#q-63610)

### 18. How do Combine and async/await interoperate, and when is Combine still the better tool?

A publisher becomes an `AsyncSequence` through `.values`, and async work becomes a publisher through `Deferred { Future { ... } }` or a subject fed from a `Task`; Combine is still the better tool for time-based composition of several streams.

[Full answer →](https://irbisa.com/topic/combine/#q-3605)

### 19. A Combine pipeline emits twice and then goes quiet. How do you find where the events stop?

Tag each stage with `print("name")` or `handleEvents` and read the lifecycle log: the first stage that stops logging values is where the chain died, and its last event says how.

[Full answer →](https://irbisa.com/topic/combine/#q-63614)

### 20. One expensive publisher has three subscribers. How many times does the work run, and how do you share it?

Three times, because each subscription runs the whole chain again; `share()`, `multicast` or `makeConnectable()` make the subscribers use one upstream execution.

[Full answer →](https://irbisa.com/topic/combine/#q-63602)

### 21. What is backpressure in Combine, and why doesn't it protect you when a `sink` meets a fast producer?

Backpressure means the subscriber tells the publisher how many values it can take through `Subscribers.Demand`; `sink` and `assign` request `.unlimited` up front, so the protocol is effectively switched off and every value is pushed straight through.

[Full answer →](https://irbisa.com/topic/combine/#q-63611)

### 22. How do you write a custom Combine operator, and when do you need your own `Publisher` and `Subscription`?

A custom operator is almost always an extension on `Publisher` that composes existing operators and returns `AnyPublisher` or `some Publisher`; a hand-written `Publisher` with its own `Subscription` is worth it only for a new source that must honour demand and cancellation itself.

[Full answer →](https://irbisa.com/topic/combine/#q-63619)

### 23. How do you unit-test a pipeline that uses `debounce` without sleeping in the test?

Inject the scheduler instead of hard-coding `DispatchQueue.main`, and pass a test scheduler with a virtual clock that the test moves forward by hand.

[Full answer →](https://irbisa.com/topic/combine/#q-63603)

### 24. What breaks when a Combine-heavy module moves to the Swift 6 language mode, and how do you fix it?

Combine predates `Sendable` and actor isolation, so Swift 6 rejects values and closures that cross isolation boundaries through it, and it adds runtime checks that crash a main-actor `sink` fed from a background queue.

[Full answer →](https://irbisa.com/topic/combine/#q-63604)
