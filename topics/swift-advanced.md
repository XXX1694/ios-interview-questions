# Swift Generics & Advanced interview questions

Generics, opaque types, property wrappers, result builders, key paths. Middle · 25 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/swift-advanced/)** · [All iOS topics](../README.md#all-topics)

### 1. What does `Codable` do, and how do you decode JSON whose keys do not match your property names?

`Codable` is `Encodable & Decodable`: conform a type and the compiler generates the code that maps its stored properties to and from JSON (or another format) by name. When names differ, you rename keys with a `CodingKeys` enum or set a key decoding strategy.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-2606)

### 2. What are generics in Swift, and what can a `where` clause express that angle brackets cannot?

Generics let you write one function or type that works with any type meeting some constraints, while the compiler still checks every use: no `Any`, no casts. A `where` clause adds constraints on associated types and relations between type parameters.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-2601)

### 3. What is an associated type, what limits does it put on a protocol, and what do primary associated types change?

An associated type is a placeholder type inside a protocol that each conforming type fills in, like `Element` in `Collection`. It makes the protocol generic, so the protocol name alone no longer describes one complete type.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62615)

### 4. Which line in this code does not compile, and what does conditional conformance have to do with it?

```swift
struct Box<Value> {
  let value: Value
}
extension Box: Equatable where Value: Equatable {}

struct Token { let raw: String }

print(Box(value: 1) == Box(value: 1))
print([Box(value: "a")] == [Box(value: "a")])
print(Box(value: Token(raw: "a")) == Box(value: Token(raw: "a")))
```

The third `print`. `Box` is `Equatable` only `where Value: Equatable`, and `Token` is not, so `Box<Token>` has no `==`: "operator function '==' requires that 'Token' conform to 'Equatable'". The first two compile because `Box<Int>` and `Box<String>` meet the condition.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62616)

### 5. What is the difference between `some` and `any` in Swift, and how do both relate to generics?

`some Shape` is an opaque type: one specific concrete type that the compiler knows and hides from the reader. `any Shape` is an existential: a box that can hold a value of any conforming type, and a different one each time.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-2602)

### 6. This function does not compile. Why, and how would you fix it?

```swift
protocol Shape { func area() -> Double }
struct Circle: Shape { let r: Double; func area() -> Double { .pi * r * r } }
struct Square: Shape { let side: Double; func area() -> Double { side * side } }

func makeShape(rounded: Bool) -> some Shape {
  if rounded {
    return Circle(r: 1)
  }
  return Square(side: 2)
}
```

`some Shape` promises one concrete type for the whole function, but the two `return`s produce `Circle` and `Square`. The compiler says the return statements "do not have matching underlying types".

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62620)

### 7. What does `@escaping` mean, and why are closure parameters non-escaping by default?

`@escaping` says the closure may be called after the function returns, because it is stored or handed to asynchronous work. Non-escaping is the default because a closure that cannot outlive the call is cheaper and safer: it cannot create a retain cycle.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62602)

### 8. What does `@autoclosure` do, and why do `&&`, `??` and `assert` take their arguments that way?

`@autoclosure` wraps the argument expression in a closure for you, so the caller writes a plain value and the function decides whether and when to evaluate it. That is lazy evaluation without braces: `??` computes its right side only when the left side is `nil`.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62626)

### 9. What are key paths in Swift, and where would you use them?

A key path is a compile-time-checked reference to a property, written `\User.name`, that you can store and pass around and later use to read, or write, that property on any instance.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-2605)

### 10. How does a property wrapper work, and how do you write a custom one?

A property wrapper is a type marked `@propertyWrapper` that owns a property's storage: for `@Clamped var volume` the compiler stores a hidden `_volume` and routes every get and set through its `wrappedValue`.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-2603)

### 11. Find the bug: `finished` ends up lower than the number of completed downloads, although every access to it goes through a lock.

```swift
@propertyWrapper
final class Locked<Value>: @unchecked Sendable {
  private let lock = NSLock()
  private var value: Value
  init(wrappedValue: Value) { value = wrappedValue }

  var wrappedValue: Value {
    get { lock.withLock { value } }
    set { lock.withLock { value = newValue } }
  }
}

final class DownloadStats: @unchecked Sendable {
  @Locked var finished = 0
  func markFinished() { finished += 1 }  // called from many download tasks
}
```

`finished += 1` is not one locked operation: it calls the getter (lock, read, unlock), then the setter (lock, write, unlock). Two threads can read the same value and both write it plus one, so increments are lost.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62623)

### 12. What is a result builder, and why does an `if` or a `for` inside a custom builder sometimes fail to compile?

A result builder is a type marked `@resultBuilder` whose static `build…` methods the compiler calls to turn the statements of a closure into one value. SwiftUI's `@ViewBuilder` is one. Each kind of control flow works only if the builder implements the matching method.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-2604)

### 13. What is the difference between `Sequence` and `Collection`, and how do you make your own type work in a `for-in` loop?

A `Sequence` only promises that you can step through its elements with an iterator, possibly only once. A `Collection` adds indices, subscripting and repeatable, non-destructive iteration. `for-in` needs only `Sequence`.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62605)

### 14. What is the difference between `#if`, `#available` and `@available` in Swift?

`#if` is a compile-time condition: code in a false branch is not compiled into the binary at all. `if #available` is a run-time check of the OS version the app is running on, and `@available` marks a declaration so the compiler makes every caller perform that check.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62619)

### 15. How do you use Objective-C code from Swift and Swift code from Objective-C in one target, and what cannot cross the boundary?

Objective-C is exposed to Swift through a bridging header (an umbrella header in a framework). Swift is exposed to Objective-C through the generated `YourModule-Swift.h`, but only declarations marked `@objc` on `NSObject` subclasses and types Objective-C can represent.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62622)

### 16. What does this code print, and why does `map` run more often than you might expect?

```swift
let numbers = [1, 2, 3, 4, 5, 6]
let squares = numbers.lazy
  .map { n -> Int in
    print("map \(n)")
    return n * n
  }
  .filter { $0 % 2 == 0 }

print("start")
print(squares.first!)
print(squares.first!)
```

It prints `start`, then `map 1`, `map 2`, `map 2`, `4`, and the same four lines again for the second `first`. A lazy chain stores closures, not results, so every access re-runs them, and `first` runs `map` on the match twice: once while finding the start index, once when subscripting it.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62621)

### 17. What is type erasure, and when do you still need a hand-written type eraser now that `any P<T>` exists?

Type erasure hides a value's concrete type behind a wrapper with a fixed type, so different types that conform to the same protocol can be stored and passed as one type. `AnyPublisher`, `AnySequence`, `AnyHashable` and SwiftUI's `AnyView` are all type erasers.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62601)

### 18. How does Swift run generic code, and why can an `any` parameter be slower than a generic one?

The optimiser can specialise a generic function, emitting a copy for each concrete type it sees, so calls are direct and can be inlined. When it cannot, one shared body runs and receives type metadata plus witness tables; an `any` value always takes that indirect path, inside a box.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62611)

### 19. What is a phantom type, and what kind of bug does it prevent?

A phantom type is a generic parameter that the type never stores; it exists only so the compiler can tell otherwise identical values apart. `ID<User>` and `ID<Order>` both wrap a `String`, but passing one where the other is expected no longer compiles.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62618)

### 20. What are typed throws (`throws(MyError)`), and when should you use them instead of plain `throws`?

Since Swift 6 a function can declare the exact error type it throws, `throws(ParseError)`, so `catch` receives a `ParseError` instead of `any Error` and can switch over it exhaustively. Plain `throws` is still the right default for most code.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62617)

### 21. Passing a big `Array` around in Swift looks free. How does copy-on-write work, and how would you implement it for your own type?

The underlying buffer is shared between copies until someone writes; before the write, a uniqueness check decides whether the buffer has to be cloned first.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62604)

### 22. Find the bug: clearing a copy of this `Playlist` also empties the original.

```swift
struct Playlist {
  private final class Storage {
    var ids: [Int]
    init(ids: [Int]) { self.ids = ids }
  }
  private var storage = Storage(ids: [])
  var ids: [Int] { storage.ids }

  mutating func removeAll() { storage.ids.removeAll() }

  mutating func append(_ id: Int) {
    if !isKnownUniquelyReferenced(&storage) { storage = Storage(ids: storage.ids) }
    storage.ids.append(id)
  }

  mutating func insert(_ id: Int, at index: Int) {
    if !isKnownUniquelyReferenced(&storage) { storage = Storage(ids: storage.ids) }
    storage.ids.insert(id, at: index)
  }
}
```

`removeAll()` writes into the shared `Storage` without the uniqueness check. After `var draft = saved` both values point at one object, so clearing `draft` clears `saved` too: the struct quietly has reference semantics.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62624)

### 23. How does Swift implement `weak` and `unowned` under the hood (side tables, three reference counts), and why can an object's memory outlive its `deinit`?

Every Swift object keeps a strong and an unowned count inline in its header, and the first `weak` reference moves the counts into a side table that weak references point to. `deinit` runs when the strong count reaches zero, but the object's memory is freed only when the unowned count does too.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62625)

### 24. You need a wrapper for a raw file descriptor that cannot be copied, closed twice or used after closing. What does `~Copyable` give you?

Suppressing `Copyable` removes implicit copies, so the value has exactly one owner at any moment and the compiler, not code review, proves that it is closed once and never used after that.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62612)

### 25. When do you write a macro instead of a property wrapper, and why is `@Observable` a macro?

A property wrapper changes how one stored property is read and written at run time. A macro generates code at compile time and can add members, accessors and conformances to a whole declaration. `@Observable` must rewrite every property of a class and add a conformance, which no wrapper can do.

[Full answer →](https://irbisa.com/topic/swift-advanced/#q-62603)
