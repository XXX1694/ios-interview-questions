# Swift OOP & Protocols interview questions

Classes, structs, enums, protocols, extensions, protocol-oriented programming. Junior · 26 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/swift-oop-protocols/)** · [All iOS topics](../README.md#all-topics)

### 1. What is the difference between a struct and a class in Swift?

Beyond copy vs share, a struct has no inheritance, `deinit` or identity, and a class has no memberwise initializer, `mutating` or automatic `Sendable`.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-2501)

### 2. What are the main principles of OOP, and how does Swift express each one?

Encapsulation, inheritance, polymorphism and abstraction; Swift supports all four, but leans on protocols and value types where classic OOP languages lean on class hierarchies.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62517)

### 3. What is the difference between overloading and overriding, and what can a subclass override?

Overloading is several functions with the same name and different signatures, chosen by the compiler at the call site; overriding is a subclass replacing an inherited implementation, chosen at runtime by the object's actual class.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62510)

### 4. What is a protocol in Swift, and how does it differ from an interface or an abstract class?

A protocol is a contract of methods, properties, initializers and associated types that classes, structs and enums can all adopt, even types you do not own; unlike an abstract class it holds no stored state.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-2502)

### 5. What is the difference between raw values and associated values in a Swift enum?

A raw value is one fixed constant of the same type for every case, known at compile time; an associated value is a payload attached to a case when you create it, and each case can carry different types.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-2503)

### 6. What can an extension add to a type in Swift, and what can it not?

An extension adds computed properties, methods, initializers, subscripts, nested types and protocol conformances to any type, even one you do not own; it cannot add stored properties or override existing members.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-2504)

### 7. What does `typealias` do, and does it create a new type?

`typealias` gives an existing type a second name; it does not create a new type. The alias and the original are interchangeable everywhere, so it helps readability but adds no type safety.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62522)

### 8. What access levels does Swift have, and what is the difference between `public` and `open`?

Swift has six levels: `open`, `public`, `package`, `internal` (the default), `fileprivate` and `private`; `public` and `open` are both visible to other modules, but only `open` lets those modules subclass a class or override a member.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-2506)

### 9. What is a `lazy` property, how does it differ from a computed and an observed property, and can it be computed more than once?

A `lazy var` runs its initializer on first read and caches the result, while `willSet`/`didSet` run code around every assignment; `lazy` computes once, unless two threads hit the first read at the same time.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-2507)

### 10. What does `mutating` do, and why do struct methods need it while class methods do not?

`mutating` marks a struct or enum method that changes `self`: `self` is passed `inout`, so the change is written back into the caller's variable. A class method changes an object through a reference, and the reference itself never changes.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62506)

### 11. What do `Equatable` and `Hashable` require, and what happens when two different values have the same hash?

`Equatable` requires `==`, and `Hashable` adds `hash(into:)` with one rule: equal values must produce equal hashes. Two different values sharing a hash is a normal collision; `Set` and `Dictionary` fall back to `==` to tell them apart, just a little slower.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62502)

### 12. This delegate declaration does not compile. What is the error, and how do you fix it?

```swift
protocol DownloaderDelegate {
  func downloader(_ downloader: Downloader, didFinish text: String)
}

final class Downloader {
  weak var delegate: DownloaderDelegate?

  func finish(with text: String) {
    delegate?.downloader(self, didFinish: text)
  }
}
```

`weak` works only with class instances, and nothing stops a struct from conforming to this protocol, so the compiler reports `'weak' must not be applied to non-class-bound 'any DownloaderDelegate'`. Make the protocol class-bound: `protocol DownloaderDelegate: AnyObject`.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62503)

### 13. What is `deinit`, when is it called, and on which thread does it run?

`deinit` is a class's cleanup method, called automatically just before ARC frees the instance, when the last strong reference goes away. It runs synchronously on whichever thread released that last reference, not necessarily the main thread.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62518)

### 14. What is the difference between `static` and `class` members, and what does `final` change?

Both declare members on the type rather than on an instance, but a `class` member can be overridden by a subclass and a `static` one cannot: in a class, `static` means `class final`.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62505)

### 15. What is the difference between `self` and `Self` in Swift?

`self` is a value: the current instance, or the type itself inside a `static` or `class` method. `Self` is a type: the conforming type in a protocol, the type itself in a struct or enum, and the dynamic runtime type in a class.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62523)

### 16. How does class initialization work in Swift: designated vs convenience initializers, the two phases, and failable inits?

A designated initializer gives every stored property of its class a value and then calls a designated initializer of the superclass; a convenience initializer must call another initializer of the same class. Together they guarantee nothing reads `self` before the whole chain is initialized.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62501)

### 17. What is protocol-oriented programming, and how is it different from class inheritance?

Protocol-oriented programming builds behaviour from small protocols with default implementations in extensions instead of a tree of base classes. A type picks exactly the capabilities it needs, and structs and enums can take part, not only classes.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-2505)

### 18. What does this code print, and why do the two calls behave differently?

```swift
protocol Greeter {
  func greet() -> String
}

extension Greeter {
  func greet() -> String { "default greet" }
  func shout() -> String { "default shout" }
}
struct Robot: Greeter {
  func greet() -> String { "beep" }
  func shout() -> String { "BEEP" }
}

let robot: any Greeter = Robot()
print(robot.greet())
print(robot.shout())
```

It prints `beep` and then `default shout`. `greet()` is declared in the protocol, so it is a requirement dispatched through the witness table and the type's version wins; `shout()` exists only in the extension, so the compiler picks it statically from the declared type `any Greeter`.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62504)

### 19. Can a Swift protocol have optional methods? Compare `@objc optional` with a default implementation.

Only an `@objc` protocol can mark requirements `optional`, and then only classes can conform; in pure Swift you get the same effect with a default implementation in a protocol extension.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62515)

### 20. Why does this struct not compile, and how would you model a linked list or a tree instead?

```swift
struct Node {
  let value: Int
  var next: Node?
}

var head = Node(value: 1, next: Node(value: 2, next: nil))
```

A struct is stored inline, so a `Node` containing an optional `Node` would need infinite size; the compiler reports "value type 'Node' cannot have a stored property that recursively contains it". `Optional` does not help, because it is also an inline value type.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62519)

### 21. When would you model variants with an enum with associated values, and when with a protocol and one type per variant?

Use an enum when you own the whole set of cases and want the compiler to find every place that must change; use a protocol when new variants must be added without touching existing code, for example by another module.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62509)

### 22. Find the bug: after the 2.3 update, orders cached by the previous version show the wrong status.

```swift
enum OrderStatus: Int, Codable {
  case draft      // added in 2.3
  case paid
  case shipped
  case delivered
}

struct Order: Codable {
  let id: Int
  var status: OrderStatus
}

// Orders are cached on disk between launches
func loadCachedOrder(from data: Data) throws -> Order {
  try JSONDecoder().decode(Order.self, from: data)
}
```

Implicit `Int` raw values are positions: inserting `draft` at the top shifts every case by one, so a cached `1` that meant `shipped` in 2.2 now decodes as `paid`, and old `delivered` orders show as `shipped`. Nothing crashes; the data is silently wrong.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62520)

### 23. Why must a non-final class mark the initializer that satisfies a protocol's `init` requirement as `required`?

Every subclass inherits the conformance, so every subclass must also have that initializer, and `required` is how the compiler enforces it. A `final` class, a struct or an enum has no subclasses and needs no keyword.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62511)

### 24. How does method dispatch work in Swift (static, vtable, witness table and message dispatch), and what does each cost?

Swift uses the cheapest dispatch a declaration allows: direct calls for value types and `final` members, a vtable for overridable class methods, a witness table for protocol requirements, and Objective-C messaging only for `@objc dynamic`.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62514)

### 25. Find the bug: pull-to-refresh calls `refreshAll`, but the feed screen never reloads.

```swift
@MainActor protocol Refreshable: AnyObject {
  func refresh()
}
extension Refreshable {
  func refresh() {}  // default: nothing to reload
}

class BaseViewController: UIViewController, Refreshable {}

final class FeedViewController: BaseViewController {
  func refresh() { reloadFeed() }
}

@MainActor func refreshAll(_ screens: [any Refreshable]) {
  screens.forEach { $0.refresh() }
}
```

`BaseViewController` conforms using the extension's empty default, so the protocol's witness table points at that default. `FeedViewController.refresh()` is just a new method, so `refreshAll` never calls it and pull-to-refresh does nothing on the feed.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62521)

### 26. A type conforms to two protocols that each provide a default `refresh()`. What does Swift do, and how do you resolve it?

If neither default is more specific, Swift does not guess: either the conformance fails or the call is ambiguous, and the fix is to implement `refresh()` on the type itself.

[Full answer →](https://irbisa.com/topic/swift-oop-protocols/#q-62512)
