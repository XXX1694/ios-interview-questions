# Objective-C Runtime & Interop interview questions

Message dispatch, swizzling, bridging, KVO internals, when Objective-C is still needed. Senior · 18 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/objc-runtime-interop/)** · [All iOS topics](../README.md#all-topics)

### 1. What is the Objective-C runtime, and what does it mean that Objective-C sends messages instead of calling methods?

The Objective-C runtime (`libobjc`) is the library that keeps classes, methods and selectors as data while the app runs; `[obj doThing]` compiles to `objc_msgSend(obj, @selector(doThing))`, which finds the implementation only at the moment of the call.

[Full answer →](https://irbisa.com/topic/objc-runtime-interop/#q-71417)

### 2. What happens when you send a message to `nil` in Objective-C, and how does Swift handle the same situation?

Nothing happens and nothing crashes: `objc_msgSend` checks for `nil` first and returns zero, so you get `nil`, `0`, `NO` or `0.0`. Swift makes the same case explicit with optionals: `obj?.run()` is skipped and gives an optional result; force-unwrapping a `nil` crashes.

[Full answer →](https://irbisa.com/topic/objc-runtime-interop/#q-71418)

### 3. What is the difference between `@objc`, `@objcMembers` and `dynamic`, and why is `@objc` alone not enough for KVO or swizzling?

`@objc` publishes a member to the Objective-C runtime (a selector and an entry in the method list), `@objcMembers` does that for a whole class, and `dynamic` makes Swift itself call the member through `objc_msgSend`. Without `dynamic`, Swift callers bypass the runtime.

[Full answer →](https://irbisa.com/topic/objc-runtime-interop/#q-71419)

### 4. What are KVC and KVO, how do you observe a property from Swift with `observe(_:options:changeHandler:)`, and what replaces KVO in new Swift code?

KVC (key-value coding) reads and writes properties by a string key at run time; KVO (key-value observing) notifies observers when such a property changes. From Swift you call `observe(\.status)` on an `@objc dynamic` property and keep the token; your own new models use `@Observable` instead.

[Full answer →](https://irbisa.com/topic/objc-runtime-interop/#q-71433)

### 5. How does an Objective-C category differ from a class extension, and what happens when two categories add a method with the same name?

A category is a named add-on (`@interface UIView (Badge)`) that attaches methods to any class when the binary loads, even one you have no source for; a class extension (`@interface Foo ()`) is an anonymous part of the class itself, compiled with its `@implementation`, and can add ivars.

[Full answer →](https://irbisa.com/topic/objc-runtime-interop/#q-71420)

### 6. What is the difference between `+load` and `+initialize`, and why can a Swift class not override either?

`+load` runs eagerly, once for every class and category that implements it, when the binary is loaded and before `main()`; `+initialize` runs lazily, just before the first message to the class, and is inherited. Swift rejects both overrides with a compile error.

[Full answer →](https://irbisa.com/topic/objc-runtime-interop/#q-71421)

### 7. What do the Objective-C property attributes `atomic`, `nonatomic`, `strong`, `weak`, `copy` and `assign` mean, and does `atomic` make a property thread-safe?

They tell the compiler how to synthesise the accessors: `strong` retains, `weak` does not and becomes `nil`, `copy` stores a copy, `assign` stores the raw value, and `atomic` (the default) makes each single get or set atomic. That is not thread safety: `visits += 1` stays unprotected.

[Full answer →](https://irbisa.com/topic/objc-runtime-interop/#q-71434)

### 8. How do you attach a stored property to a class you do not own, and what do associated objects cost?

Extensions and categories cannot add storage, so you use associated objects: `objc_setAssociatedObject` stores a value against an object and a unique key in a side table, and the runtime releases it when the object is deallocated.

[Full answer →](https://irbisa.com/topic/objc-runtime-interop/#q-71422)

### 9. In an Objective-C block, why do you write the `weakSelf`/`strongSelf` dance, and what does `__block` change under ARC?

A block retains every object it captures, so a block stored by `self` that mentions `self` is a retain cycle; capturing `__weak weakSelf` breaks it, and turning it into `strongSelf` inside keeps the object alive for the rest of that run of the block.

[Full answer →](https://irbisa.com/topic/objc-runtime-interop/#q-71423)

### 10. How do you make an Objective-C header read well from Swift: nullability, `NS_SWIFT_NAME`, `NS_REFINED_FOR_SWIFT` and friends?

Annotate the header so the importer can produce Swift-native types: nullability turns `T!` into `T` or `T?`, lightweight generics turn `NSArray *` into `[String]`, and `NS_SWIFT_NAME` and `NS_REFINED_FOR_SWIFT` fix names and shapes that do not translate well.

[Full answer →](https://irbisa.com/topic/objc-runtime-interop/#q-71424)

### 11. What happens when an object receives a message it has no method for, and how can you step into that process?

Before crashing with "unrecognized selector sent to instance", the runtime gives the object three chances: add a method (`resolveInstanceMethod:`), name another object to handle it (`forwardingTargetForSelector:`), or take the whole call as an `NSInvocation` (`forwardInvocation:`).

[Full answer →](https://irbisa.com/topic/objc-runtime-interop/#q-71425)

### 12. What is method swizzling, how do you do it safely, and why is it risky in production?

Swizzling swaps the implementations behind two selectors at run time, so every call to, say, `viewWillAppear:` runs your code first. It works only for methods dispatched through `objc_msgSend` and changes behaviour for the whole process.

[Full answer →](https://irbisa.com/topic/objc-runtime-interop/#q-71426)

### 13. What is `isa`, and how does `objc_msgSend` use it, the method cache and the superclass chain to find an implementation?

Every object starts with an `isa` field that points to its class; `objc_msgSend` reads it, looks the selector up in that class's method cache, and on a miss walks the method lists up the superclass chain, caches the `IMP` it finds and jumps to it.

[Full answer →](https://irbisa.com/topic/objc-runtime-interop/#q-71427)

### 14. What does this code print, and what does KVO do to the object that explains the output?

```swift
class Upload: NSObject {
  @objc dynamic var progress = 0.0
  @objc var status = "idle"
}

let upload = Upload()
let a = upload.observe(\.progress) { u, _ in print("progress", u.progress) }
let b = upload.observe(\.status) { u, _ in print("status", u.status) }

upload.progress = 0.5
upload.status = "running"
upload.setValue("done", forKey: "status")
```

It prints `progress 0.5` and then `status done`. KVO replaces the object's class with a generated subclass whose setters send notifications, and only `@objc dynamic` makes Swift call a setter through `objc_msgSend`, where that subclass can catch it.

[Full answer →](https://irbisa.com/topic/objc-runtime-interop/#q-71428)

### 15. Find the bug: screen analytics work until an iPad user opens a second window, then stop for every screen. What is wrong with this swizzle?

```swift
extension UIViewController {
  static func enableScreenTracking() {
    let original = class_getInstanceMethod(UIViewController.self, #selector(viewDidAppear(_:)))!
    let tracked = class_getInstanceMethod(UIViewController.self, #selector(tracked_viewDidAppear(_:)))!
    method_exchangeImplementations(original, tracked)
  }
  @objc dynamic func tracked_viewDidAppear(_ animated: Bool) {
    tracked_viewDidAppear(animated)  // runs the original viewDidAppear
    Analytics.screen(String(describing: type(of: self)))
  }
}
final class SceneDelegate: UIResponder, UIWindowSceneDelegate {
  func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
    UIViewController.enableScreenTracking()
  }
}
```

The swizzle runs on every scene connection instead of once per process: the second window exchanges the two implementations back, so `viewDidAppear` is the original again and tracking silently stops, until a third window turns it on again.

[Full answer →](https://irbisa.com/topic/objc-runtime-interop/#q-71429)

### 16. What is toll-free bridging, and how do ARC bridge casts and `Unmanaged` decide who releases a Core Foundation object?

Toll-free bridged pairs such as `CFStringRef` and `NSString` share one memory layout, so a cast reuses the same object. ARC does not manage CF objects, so in Objective-C the bridge cast, and in Swift `Unmanaged`, states who owns the +1 reference.

[Full answer →](https://irbisa.com/topic/objc-runtime-interop/#q-71430)

### 17. What does bridging between `String` and `NSString` cost, and why does an `NSRange` built from `text.count` miss matches?

Swift's `String` stores UTF-8 and counts `Character`s (grapheme clusters), while `NSString` works in UTF-16 code units, so a bridge or an index conversion can mean transcoding and an O(n) walk, and `text.count` is not the length `NSRange` expects.

[Full answer →](https://irbisa.com/topic/objc-runtime-interop/#q-71431)

### 18. In 2026, when do you still need Objective-C in an iOS codebase, and how would you migrate a large Objective-C module to Swift?

New code is Swift, but Objective-C still earns its place for catching `NSException`, runtime work such as `forwardInvocation:` and `+load`, Objective-C++ glue to C++ that Swift's interop cannot import, and the large legacy modules and SDKs that already work.

[Full answer →](https://irbisa.com/topic/objc-runtime-interop/#q-71432)
