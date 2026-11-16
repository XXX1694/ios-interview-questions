# Swift Basics interview questions

Variables, optionals, types, control flow, closures, value vs reference. Junior · 26 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/swift-basics/)** · [All iOS topics](../README.md#all-topics)

### 1. What is the difference between `let` and `var` in Swift?

A `let` constant can be assigned exactly once, while a `var` variable can be reassigned any number of times; use `let` by default and switch to `var` only when the value really has to change.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-2401)

### 2. What is an optional in Swift, how is it implemented, and how do you unwrap it safely?

An optional either holds a value or holds `nil`, so "this may be missing" is checked by the compiler instead of crashing at runtime; `T?` is shorthand for the generic enum `Optional<T>` with the cases `.some(T)` and `.none`.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-2402)

### 3. What is the difference between `if let` and `guard let`, and when do you reach for `guard`?

Both unwrap an optional, but `if let` keeps the value inside its own braces, while `guard let` makes it available for the rest of the scope and forces the `else` branch to leave that scope.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-2407)

### 4. What does this code print, and why does the first line show `Optional("Ada")`?

```swift
let name: String? = "Ada"
let bonus: Int? = 5

print("Hello, \(name)")
print("Hello, \(name ?? "guest")")
print(bonus ?? 0 + 1)
```

It prints `Hello, Optional("Ada")`, then `Hello, Ada`, then `5`: interpolating an optional prints the description of the `Optional` wrapper, and `??` binds more loosely than `+`.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62406)

### 5. What is the difference between `String?` and `String!`, and why does an `@IBOutlet` declared as `UILabel!` crash with "unexpectedly found nil"?

`String!` is an implicitly unwrapped optional: the same `Optional<String>` underneath, but the compiler inserts a force unwrap wherever you use it as a non-optional, so reading it while it is `nil` crashes exactly like `!` does.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62407)

### 6. What is the difference between value types and reference types in Swift?

A value type is copied when you assign it or pass it to a function, so every variable owns its own data; a reference type is shared, so several variables point at one instance and see each other's changes.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-2403)

### 7. What is the difference between `==` and `===` in Swift?

`==` asks whether two values are equal by content and comes from the `Equatable` protocol; `===` asks whether two class references point at the very same instance.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62414)

### 8. What is the difference between `is`, `as`, `as?` and `as!`?

`is` checks a value's runtime type, `as?` attempts a downcast and returns an optional, `as!` forces the downcast and crashes if it fails, and plain `as` is an upcast or bridging conversion the compiler can prove safe.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62405)

### 9. What is the difference between `Any` and `AnyObject`?

`Any` can hold a value of any type, including structs, enums, optionals and functions; `AnyObject` holds only class instances, so it is also the constraint for class-only protocols.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62418)

### 10. What is the difference between `Array`, `Set` and `Dictionary`, and how do you choose between them?

Choose by what you need: an `Array` keeps order and allows duplicates, a `Set` stores unique elements with fast lookup, and a `Dictionary` maps unique keys to values.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-2405)

### 11. What is the difference between `map`, `compactMap` and `flatMap` in Swift?

`map` transforms every element one-to-one, `compactMap` transforms and drops the `nil` results, and `flatMap` turns each element into a sequence and joins all of them into one array.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62402)

### 12. What is the difference between a `for-in` loop and `forEach`, and why does `return` inside `forEach` not stop the iteration?

`forEach` calls a closure once per element, so `return` leaves only the current call and the iteration goes on; `break` and `continue` do not compile inside it. A `for-in` loop is a statement: it supports `break`, `continue`, `return` from the enclosing function, a `where` filter and `await`.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62420)

### 13. What can Swift's `switch` do that a C-style `switch` cannot?

Swift's `switch` matches patterns rather than just integers, must be exhaustive, and never falls through into the next case unless you write `fallthrough`.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62403)

### 14. Why can't you add an `Int` to a `Double` in Swift, and how do `Float`, `Double` and `CGFloat` differ?

Swift never converts numeric types implicitly, so `Int` plus `Double` is a compile error until you write `Double(count)`; `Float` is 32-bit, `Double` is 64-bit and the default, and `CGFloat` is the type Core Graphics and UIKit use for sizes.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62408)

### 15. What is a closure in Swift, and is it a value type or a reference type?

A closure is a block of code you can store in a variable and pass around, and it remembers the variables of the scope where it was created; closures, like functions, are reference types.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-2404)

### 16. What is the difference between `weak` and `unowned`, and when do you need `[weak self]` in a closure?

Neither keeps the object alive: `weak` is an optional that becomes `nil` when the object is deallocated, while `unowned` is non-optional and crashes if you touch it after the object is gone. Use `[weak self]` in an escaping closure that `self` itself owns.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62415)

### 17. How does error handling work in Swift: `throws`, `try`, `try?`, `try!`, `do`/`catch` and `Result`?

A function that can fail is marked `throws`, every call to it must be marked with `try`, and the error is either handled in `do`/`catch` or passed up to the caller, so failure is visible in the function's signature.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-2406)

### 18. What does this code print, and why does changing an element of the copy also change the original array?

```swift
final class Item {
  var title: String
  init(title: String) { self.title = title }
}

var original = [Item(title: "A"), Item(title: "B")]
var copy = original
copy[0].title = "Changed"
copy.append(Item(title: "C"))

print(original[0].title)
print(original.count)
```

It prints `Changed` and then `2`: the array itself was copied, so appending to `copy` does not touch `original`, but both arrays hold references to the same `Item` objects.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62411)

### 19. Find the bug: tapping "Add" on a product that is already in the cart never increases its quantity.

```swift
struct CartItem {
  let name: String
  var quantity: Int
}

final class Cart {
  private(set) var items: [CartItem] = []

  func add(_ name: String) {
    if var existing = items.first(where: { $0.name == name }) {
      existing.quantity += 1
    } else {
      items.append(CartItem(name: name, quantity: 1))
    }
  }
}
```

`if var existing = items.first(...)` binds a copy of the struct, so `quantity += 1` changes that local copy and the array keeps the old value: every product stays at quantity 1.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62419)

### 20. What does this code print, and what does the capture list `[value]` change?

```swift
func demo() {
  var value = 1
  let captured = { print("captured:", value) }
  let copied = { [value] in print("capture list:", value) }

  value = 2
  captured()
  copied()
}

demo()
```

It prints `captured: 2` and then `capture list: 1`: a normal closure captures the variable itself and reads it at call time, while a capture list copies the value at the moment the closure is created.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62416)

### 21. What does this code print, and when exactly does a `defer` block run?

```swift
func demo() {
  var counter = 0
  func next() -> Int {
    defer { counter += 1 }
    return counter
  }
  print(next(), counter)
}

func order() {
  defer { print("A") }
  defer { print("B") }
  print("C")
}

demo()
order()
```

It prints `0 1`, then `C`, `B`, `A`: a `defer` block runs when its scope exits, after the return value has already been computed, and several `defer` blocks run in reverse order.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62410)

### 22. What does `inout` do, and how is it different from passing a class instance?

An `inout` parameter is copy-in, copy-out: the function works on a mutable copy of your variable and writes it back when it returns, so the caller sees the change; you pass the argument with `&`.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62404)

### 23. Why can't you write `name[3]` on a Swift `String`, and what is a `Substring`?

A Swift `Character` is a grapheme cluster that can take several bytes, so there is no constant-time way to jump to the fourth one; `String` deliberately has no integer subscript, and you move through it with `String.Index` instead.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62401)

### 24. `let rest = items.dropFirst()` and then `rest[0]` crashes with "Index out of bounds", although `rest` is not empty. Why?

`dropFirst` returns an `ArraySlice` that keeps the original array's indices, so its first element sits at index 1, not 0, and `rest[0]` really is out of range.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62412)

### 25. Three items at 1.10 add up to 3.3000000000000003 on a receipt. What went wrong, and what type should store money?

`Double` is binary floating point, and values like 1.1 or 0.1 have no exact binary representation, so each operation adds a tiny error that shows up once the result is printed; store money as integer minor units or as `Decimal`.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62409)

### 26. What is the difference between `assert`, `precondition` and `fatalError`, and which of them still work in a Release build?

`assert` is checked only in Debug builds, `precondition` is checked in both Debug and Release, and `fatalError` always stops the app; pick by whether a broken assumption is allowed to continue in production.

[Full answer →](https://irbisa.com/topic/swift-basics/#q-62417)
