# SwiftUI Fundamentals interview questions

View protocol, modifiers, layout containers, navigation, lists. Junior · 25 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/swiftui-fundamentals/)** · [All iOS topics](../README.md#all-topics)

### 1. What is SwiftUI, and what are its advantages and drawbacks compared with UIKit?

SwiftUI is Apple's declarative UI framework: you describe what the screen looks like for the current state, and the framework works out which changes to apply. UIKit is imperative; you create views and then mutate them yourself.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-2801)

### 2. What does declarative UI mean, and why are SwiftUI views structs rather than classes?

Declarative means you describe the result (what the screen looks like for a given state) and the framework performs the updates. A SwiftUI view is a cheap struct that describes a piece of UI; it is not the object drawn on screen.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62815)

### 3. What is the `View` protocol, what is `body`, and what does `some View` mean?

`View` is the protocol every SwiftUI view conforms to, and its one requirement is a computed `body` that returns other views. `some View` is an opaque return type: `body` returns one concrete type that the compiler knows, but you never have to spell it out.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62816)

### 4. What do `@main`, `App` and `Scene` do in a SwiftUI app?

`@main` marks the app's entry point, the `App` protocol describes the app as a whole, and its `body` returns one or more scenes: containers such as `WindowGroup` that the system turns into windows.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62817)

### 5. How do `VStack`, `HStack` and `ZStack` lay out their children, and when do you need `LazyVStack` instead?

`VStack` places children top to bottom, `HStack` from leading to trailing, and `ZStack` layers them back to front. `LazyVStack` and `LazyHStack` lay out the same way but create children only as they come near the screen, which matters for long content in a `ScrollView`.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-2802)

### 6. What does each of these three badges look like, and why does swapping two modifiers change the result?

```swift
// Badge A
Text("Sale")
  .padding()
  .background(.red)

// Badge B
Text("Sale")
  .background(.red)
  .padding()

// Badge C
Text("Sale")
  .background(.red)
  .frame(width: 200)
```

Badge A is a red box with the text inside default padding, B is a tight red strip with invisible padding around it, and C is red only behind the text inside a wider, empty frame. Each modifier wraps the view it is called on, so it only affects what came before it in the chain.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62818)

### 7. How do you write a reusable style in SwiftUI: a custom `ViewModifier` or a `ButtonStyle`?

Use a `ViewModifier` to bundle modifiers you apply to any view, such as a card look, and a `ButtonStyle` to style buttons, because it also receives the pressed state and the button's role.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-2803)

### 8. You stretched a `Text` with `.frame(maxWidth: .infinity)` and it stayed centred. How do the different alignments in SwiftUI differ?

`.frame` centres its child by default, so you need `.frame(maxWidth: .infinity, alignment: .leading)`. A stack's `alignment` lines children up with each other, a frame's `alignment` places the child inside the frame, and `multilineTextAlignment` aligns the lines inside one `Text`.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62808)

### 9. A bundled `Image` ignores its `.frame` and `.scaledToFill()` spills over neighbours; in what order do the modifiers go, and what does `AsyncImage` not do for you?

`Image` keeps its natural size until you call `.resizable()`, and `.scaledToFill()` draws outside its frame unless you clip it. `AsyncImage` has no in-memory cache of decoded images, so a row that scrolls back into a list asks for the image again and decodes it again.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62809)

### 10. How do you build a `List` with `ForEach`, and why does every row need a stable `id`?

`List` is a scrolling, platform-styled container, and `ForEach` creates one row per element of a collection. SwiftUI matches rows between updates by their `id`, so the id must come from the data and never change, or rows lose their state and animate wrongly.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-2805)

### 11. How do you build a form in SwiftUI with `TextField`, `SecureField`, `Toggle` and `Picker`?

Wrap the controls in a `Form`, which gives them the grouped settings look, and bind each control to state with `$`, as in `TextField("Email", text: $email)`. The control reads and writes that state directly, so there is no delegate or target-action to wire up.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62819)

### 12. How do you present a sheet or an alert in SwiftUI, and when do you use the `item:` form instead of `isPresented:`?

Presentation is a modifier bound to state: set a `Bool` or an optional value and SwiftUI shows the sheet or alert; reset it and the screen goes away. Use `.sheet(item:)` whenever the sheet shows a specific value, such as the tapped row.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62803)

### 13. When do you use `LazyVGrid` and when `Grid`, and how do you make the column count follow the screen width?

`LazyVGrid` flows a collection into columns and builds cells lazily, which suits a gallery; `Grid` is an eager table whose rows share column widths, which suits a form or comparison table. `.adaptive(minimum:)` fits as many columns as the width allows.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62811)

### 14. How do you write SwiftUI previews for a view with several states or a service dependency?

Use the `#Preview` macro (one named preview per state that can break the layout) and feed it sample data and fake services, so the preview never touches the network or a real database.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-2806)

### 15. When do you use a `List`, and when a `ScrollView` with a `LazyVStack`?

Use `List` for long, uniform data: it reuses rows and brings separators, swipe actions, selection and edit mode for free. Use `ScrollView` with `LazyVStack` when the design needs a custom layout that list styling would fight.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62805)

### 16. How does SwiftUI decide a view's size: who has the final say, the parent or the child?

Layout is a negotiation: the parent proposes a size, the child picks its own size, and the parent then places the child. The child has the final say, which explains most layout surprises.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62801)

### 17. How does `NavigationStack` work, and how do you navigate programmatically or from a deep link?

`NavigationStack` keeps its screens as a path of values: `NavigationLink(value:)` appends a value, `.navigationDestination(for:)` turns a value of that type into a screen, and Back removes the last one. Bind the path to state and you navigate by editing an array.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-2804)

### 18. Tapping a row in this list does nothing, and Xcode logs a navigation warning: what is wrong?

```swift
struct ShopView: View {
  let products: [Product]

  var body: some View {
    NavigationStack {
      List(products) { product in
        NavigationLink(product.name, value: product)
          .navigationDestination(for: Product.self) { product in
            ProductDetail(product: product)
          }
      }
      .navigationTitle("Shop")
    }
  }
}
```

The `.navigationDestination` sits inside the `List` rows. `List` builds rows lazily, so the stack cannot rely on seeing the destination when the link fires, and the link may do nothing; move the modifier onto the `List` itself.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62820)

### 19. What does this code print when the screen appears and on each tap, and what does that tell you about `init`, `body` and `onAppear`?

```swift
struct Child: View {
  init() { print("init") }
  var body: some View {
    let _ = print("body")
    Text("Child").onAppear { print("appear") }
  }
}

struct Parent: View {
  @State private var count = 0
  var body: some View {
    VStack { Text("Taps: \(count)"); Child(); Button("Tap") { count += 1 } }
  }
}
```

In a plain run, on first appearance it prints `init`, `body`, `appear`; after every tap, only `init`. The parent's `body` re-runs and recreates the `Child` struct, but the new value equals the old one, so SwiftUI skips `Child.body`, and the child never left the screen, so `onAppear` does not fire again.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62810)

### 20. Find the bug: each time the user comes back to this screen the stopwatch counts faster, and it keeps ticking while another screen is on top.

```swift
struct StopwatchView: View {
  @State private var seconds = 0

  var body: some View {
    Text("\(seconds) s")
      .font(.largeTitle.monospacedDigit())
      .onAppear {
        Task {
          do {
            while true {
              try await Task.sleep(for: .seconds(1))
              seconds += 1
            }
          } catch {}
        }
      }
  }
}
```

The `Task` started in `.onAppear` is not tied to the view, so nothing ever cancels it: each return to the screen starts one more endless loop, and all of them keep adding to `seconds`, even while another screen covers this one.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62821)

### 21. What is `@ViewBuilder` doing behind the scenes, and why is returning `AnyView` a bad habit?

`@ViewBuilder` is a result builder that folds a list of statements into one nested generic view type, which is how `body` can hold several views and an `if` without you writing that type out. `AnyView` erases that type, so SwiftUI can no longer compare the old and new trees structurally.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62802)

### 22. What is the difference between `withAnimation` and the `.animation` modifier, and why was the single-argument `.animation(_:)` deprecated?

`withAnimation` animates every change caused by the state mutation inside its closure, wherever the affected views sit; the `.animation(_:value:)` modifier animates one view's reaction to one named value.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62804)

### 23. A list row has a star button and a delete button, and tapping either one runs both plus the row's navigation: what is happening?

Inside a `List` row, buttons with the default style share the whole row as their tap target, so one tap triggers every button in the row. Give each button `.buttonStyle(.borderless)` or `.plain` so it owns only its own label's area.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62813)

### 24. After saving on the fourth pushed screen you must land back on the root, and the same flow also runs inside a sheet: how do you build that?

Pop-to-root is one data change (clearing the array that drives the `NavigationStack`), so the deep screen needs access to that path, because `@Environment(\.dismiss)` only ever undoes one level. In a sheet the flow gets its own `NavigationStack` and its own router.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62814)

### 25. Find the bug: picking another article updates the title, but this web view keeps showing the first page.

```swift
struct WebView: UIViewRepresentable {
  let url: URL

  func makeUIView(context: Context) -> WKWebView {
    let webView = WKWebView()
    webView.load(URLRequest(url: url))
    return webView
  }

  func updateUIView(_ webView: WKWebView, context: Context) {}
}

// In the parent: WebView(url: selectedArticle.url)
```

`updateUIView` is empty, so the new `url` never reaches the `WKWebView`. `makeUIView` runs once per view identity; when the parent passes another URL, SwiftUI keeps the same UIKit view and only calls `updateUIView`, which does nothing.

[Full answer →](https://irbisa.com/topic/swiftui-fundamentals/#q-62822)
