# SwiftUI State Management interview questions

@State, @Binding, @StateObject, @ObservedObject, @EnvironmentObject, Observation. Middle · 24 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/swiftui-state/)** · [All iOS topics](../README.md#all-topics)

### 1. What is the difference between `@State` and `@Binding` in SwiftUI?

`@State` is storage a view owns: SwiftUI keeps the value outside the view struct and re-renders the view when it changes. `@Binding` is a read-write reference to state owned by someone else; the child can change the value but does not store it.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-2901)

### 2. In this view, what do `count`, `$count` and `_count` each refer to?

```swift
struct CounterView: View {
  @State private var count: Int

  init(start: Int = 0) {
    _count = State(initialValue: start)
  }

  var body: some View {
    VStack {
      Text("\(count)")
      Stepper("Count", value: $count)
      Button("Reset") { count = 0 }
    }
  }
}
```

`count` is the current value (the wrapper's `wrappedValue`), `$count` is a `Binding<Int>` to it (the `projectedValue`), and `_count` is the `State<Int>` wrapper itself, which you only touch in `init`.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-62915)

### 3. What is the difference between `@Environment`, `@EnvironmentObject` and `@ObservedObject`, and when do you use the environment instead of `init`?

`@ObservedObject` receives an object explicitly through the initializer, while `@EnvironmentObject` and `@Environment` read something injected higher up the tree, so the views in between do not have to pass it along (prop drilling).

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-2903)

### 4. Which `@Environment` values do you use in everyday code? For example, how does a child view dismiss its own screen?

It reads `@Environment(\.dismiss)` and calls it. The environment knows whether the view was presented as a sheet or pushed onto a navigation stack, so the child needs no binding back to whoever presented it.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-62905)

### 5. The Done button on your form's keyboard does nothing and Next never jumps to the following field. Which property wrapper is missing?

`@FocusState` (together with `.onSubmit`) is missing. `@FocusState` mirrors keyboard focus: bind fields to it with `.focused(...)`, assign a field to move focus there and `nil` to hide the keyboard.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-62907)

### 6. The parent changes `start` from 0 to 5, yet `Counter` keeps showing 0. Why, and how do you fix it?

```swift
struct Counter: View {
  @State private var count: Int

  init(start: Int) {
    _count = State(initialValue: start)
  }

  var body: some View {
    Button("\(count)") { count += 1 }
  }
}

// In the parent:
// Counter(start: selectedPreset.start)
```

`State(initialValue:)` is used only the first time a view with this identity appears. After that SwiftUI keeps its own stored value and ignores every new initial value the parent passes, so `count` stays at 0.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-62917)

### 7. When do you use `@StateObject` vs `@ObservedObject`, and what breaks if you pick the wrong one?

`@StateObject` means the view owns the object: SwiftUI creates it once and keeps it alive for the view's lifetime. `@ObservedObject` only watches an object someone else owns, so an object created inline with it is recreated whenever the parent re-renders.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-2902)

### 8. What does `@Published` do, and how does an `ObservableObject` announce changes?

`@Published` wraps a property and emits on the object's `objectWillChange` publisher just before the value changes; views observing the object (`@StateObject`, `@ObservedObject`, `@EnvironmentObject`) then re-run `body`.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-62921)

### 9. Find the bug: tapping a product adds it to the cart, but the cart counter in the toolbar does not change.

```swift
final class Cart: ObservableObject { @Published var items: [Product] = [] }
final class ShopModel: ObservableObject { @Published var cart = Cart() }

struct ShopScreen: View {
  let products: [Product]
  @StateObject private var model = ShopModel()
  var body: some View {
    List(products) { p in
      Button(p.name) { model.cart.items.append(p) }
    }
    .toolbar { Text("Cart: \(model.cart.items.count)") }
  }
}
```

`Cart` is a class, so `@Published var cart` holds a reference that never changes. Appending to `cart.items` fires `Cart`'s own `objectWillChange`, which no view observes, so `ShopModel` stays silent and the toolbar keeps the old count.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-62922)

### 10. What does the `@Observable` macro change compared with `ObservableObject` and `@Published`?

`@Observable` makes every stored property of a class observable without `@Published`, and SwiftUI tracks which properties each view actually read, so a view re-renders only when those properties change instead of on any change to the object.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-2904)

### 11. What is `@Bindable`, and when do you need it with an `@Observable` model?

`@Bindable` turns an `@Observable` object into a source of bindings: `$profile.name` becomes a `Binding<String>` for a `TextField`. You need it only in a view that did not create the model but has to hand two-way bindings to controls.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-62918)

### 12. You tap the button and the text never changes. Why, and what is the fix?

```swift
final class Draft {
  var title = ""
}

struct Editor: View {
  @State private var draft = Draft()

  var body: some View {
    VStack {
      Text(draft.title.isEmpty ? "empty" : draft.title)
      Button("Set") { draft.title = "Hello" }
    }
  }
}
```

`@State` re-renders the view when the value it stores is replaced, and `draft.title = "Hello"` changes the object behind the reference without replacing the reference, so SwiftUI sees no change.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-62908)

### 13. How do you choose the right property wrapper for a piece of state in SwiftUI?

The view that creates the state owns it: `@State` for a value or an `@Observable` model. Everyone else gets a plain `let` to read, `@Binding` (or `@Bindable` for a model) to write, or the environment for a whole subtree.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-62919)

### 14. How do you hand a child view a `Binding` when the value is not stored in `@State`?

A `Binding` is just a getter plus a setter, so you can build one with `Binding(get:set:)` whenever the value lives where a `$` cannot reach: behind a type conversion or inside an optional.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-62902)

### 15. Edits made on a detail screen never appear in the list behind it, even though both draw the same `Order` struct. What went wrong?

The detail screen got a copy. `Order` is a struct, so passing it as `let order` or seeding a local `@State` from it gives the child its own value, and every edit lands on that copy.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-62910)

### 16. When does SwiftUI re-run a view's `body`, and when does the view's `@State` get reset?

SwiftUI re-runs `body` when something the view read has changed: its state, a binding, an environment value, an observed property, or the inputs its parent passed. It resets `@State` when the view's identity changes, because state belongs to identity.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-2905)

### 17. A row's expanded/collapsed toggle keeps resetting after the user scrolls it off screen and back. What is actually happening?

The flag lives in the row's `@State`, and a `List` or `LazyVStack` is free to destroy rows that are far off screen. SwiftUI does not promise to keep their state, so the row comes back built fresh with the default value.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-62911)

### 18. Where do side effects and async work belong in a SwiftUI view, and what happens to a `.task` when the view disappears?

Never in `body`, which SwiftUI may run at any time and any number of times. Side effects go in action closures and event modifiers (`.task`, `.onChange`, `.onAppear`), and a `.task` is cancelled automatically when its view disappears.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-2906)

### 19. What do `@AppStorage` and `@SceneStorage` do, and where do they stop being the right tool?

`@AppStorage` is a property backed by `UserDefaults` that re-renders the view when the key changes; `@SceneStorage` is per-window UI state that the system saves and restores when the app is relaunched.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-62904)

### 20. How does `@Observable` know which views to invalidate, and what kinds of change does that tracking silently miss?

The macro turns each stored property into a computed one that reports reads and writes to an `ObservationRegistrar`, and SwiftUI runs `body` inside observation tracking, so a view depends on exactly the properties it touched during that run.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-62913)

### 21. Your model writes its properties from a background network callback and the list updates late, glitches, or crashes. What does SwiftUI actually require here?

Every write a view can observe has to happen on the main actor. SwiftUI views, `@State` and the rendering that reads your model are `@MainActor`, so a write from another thread is a data race, not merely a slow update.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-62914)

### 22. Find the bug: the chat server logs a new connection every time the parent screen redraws, though the user has only one chat open.

```swift
// ChatSocket wraps a WebSocket: connect() opens it, disconnect() closes it, messages() streams incoming text
@MainActor @Observable final class ChatModel {
  var messages: [String] = []
  private let socket = ChatSocket(url: supportURL)

  init() {
    Task {                               // bug: side effects in init
      socket.connect()
      for await message in socket.messages() { messages.append(message) }
    }
  }
}

struct SupportChatScreen: View {
  @State private var model = ChatModel()
  var body: some View {
    List(model.messages, id: \.self) { Text($0) }
  }
}
```

`@State` evaluates its initial value every time the view struct is created, and SwiftUI recreates `SupportChatScreen` on every parent redraw. Each time a new `ChatModel` opens a socket in `init`; `@State` keeps the first model and drops the rest, but they have already connected.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-62923)

### 23. A SwiftUI screen stutters and you suspect too many body evaluations. How do you confirm it and what do you change?

First prove which views re-render and why, then fix the cause. The tools are `Self._printChanges()` inside the suspect `body` and the SwiftUI instrument in Instruments, run on a release build on a real device.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-62906)

### 24. How would you organise state ownership and data flow in a large SwiftUI app?

Give every piece of state exactly one owner at the narrowest scope that needs it. Data flows down as values or observable models, and changes flow back up as calls to the owner's methods, never as writes to a copy.

[Full answer →](https://irbisa.com/topic/swiftui-state/#q-62920)
