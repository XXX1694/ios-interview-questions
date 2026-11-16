# SwiftUI Layout, Identity & Rendering interview questions

Layout protocol, GeometryReader, PreferenceKey, view identity, Canvas, lazy containers, insets. Senior · 21 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/swiftui-layout-rendering/)** · [All iOS topics](../README.md#all-topics)

### 1. In this horizontal `ScrollView` the `Spacer` collapses and the cards sit side by side: what is the `ScrollView` proposing, and how do you fix it?

```swift
ScrollView(.horizontal) {
  HStack {
    CardView(card: first)
    Spacer()
    CardView(card: second)
  }
}
```

A horizontal `ScrollView` proposes a `nil` width to its content, which means "tell me your ideal width". A `Spacer` asked for its ideal answers with its minimum length, so there is no extra space for it to fill.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69201)

### 2. When would you use `GeometryReader`, and what does it do to the layout around it?

`GeometryReader` hands its closure the size it was proposed, but it takes all of that space for itself and pins its content to the top-leading corner, so as a normal child it pushes its siblings away. Use it inside a `background` or `overlay`, or use a newer API.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69203)

### 3. What is a `PreferenceKey`, and how does a child pass a value up to its ancestors?

A `PreferenceKey` is a typed channel for sending values from descendants up the view tree: children set it with `.preference(key:value:)`, SwiftUI merges the values from all children with the key's `reduce`, and an ancestor reads the result in `onPreferenceChange`.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69219)

### 4. How does `ViewThatFits` pick which child to show, and why is it often better than checking `horizontalSizeClass`?

`ViewThatFits` asks each child, in order, for its ideal size along the axes you name and shows the first one that fits in the space it was offered. If none fits, it shows the last child.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69204)

### 5. How do you wrap a UIKit view with `UIViewRepresentable`, host SwiftUI in UIKit with `UIHostingController`, and how does each side learn the other's size?

`UIViewRepresentable` wraps a UIKit view for SwiftUI, and `UIHostingController` wraps a SwiftUI view as a view controller. Size crosses the border through `intrinsicContentSize` by default, through `sizeThatFits(_:uiView:context:)` into SwiftUI and through `sizingOptions` into Auto Layout.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69222)

### 6. How do alignment guides work, and how do you line up views from different rows with a custom alignment?

A stack asks each child for one value, its alignment guide, and shifts the children so those values line up. `alignmentGuide(_:computeValue:)` overrides the value for one child; a custom `AlignmentID` lets views in different rows share one line.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69205)

### 7. What is view identity in SwiftUI (structural versus explicit), and what happens when a view's identity changes?

Identity is how SwiftUI tells "the same view, updated" from "a different view": structural identity comes from a view's type and position in `body`, explicit identity from `.id(_:)` and `ForEach` ids. When it changes, the old view is destroyed and a new one starts with fresh state.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69207)

### 8. The user picks another item in the sidebar, but this editor still shows the previous item's title: why, and how do you fix it?

```swift
struct DetailEditor: View {
  let item: Item
  @State private var draft: String

  init(item: Item) {
    self.item = item
    _draft = State(initialValue: item.title)
  }

  var body: some View {
    TextField("Title", text: $draft)
  }
}
```

SwiftUI stores `@State` against the view's identity and uses the initial value only once. Selecting another item rebuilds the `DetailEditor` struct in the same slot, so identity is unchanged, `State(initialValue:)` is ignored and `draft` keeps the old title.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69208)

### 9. SwiftUI warns that an ID occurs multiple times in a `ForEach`, and rows start swapping state: what broke, and where do you fix it?

Two elements give `ForEach` the same id, so SwiftUI cannot match old rows to new ones: state, focus and animations attach to the wrong row, and one duplicate may not render at all. The fix belongs in the data; ids must be unique and stable.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69209)

### 10. Every time `isHighlighted` flips, the counter in this row resets to zero: what is wrong with this conditional-modifier helper?

```swift
extension View {
  @ViewBuilder
  func `if`<T: View>(_ condition: Bool, transform: (Self) -> T) -> some View {
    if condition { transform(self) } else { self }
  }
}

struct Row: View {
  let isHighlighted: Bool

  var body: some View {
    Counter()   // holds @State private var count = 0
      .if(isHighlighted) { $0.foregroundStyle(.red) }
  }
}
```

The `if`/`else` inside the helper produces `_ConditionalContent<T, Self>`, so the highlighted and the plain `Counter` live in two different branches: two structural identities. Flipping the flag destroys one `Counter` with its `@State` and creates the other from scratch.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69218)

### 11. Find the bug: every label in this form is clipped to the width of "Phone", so "Email address" is truncated instead of the widest label setting the width.

```swift
struct LabelWidthKey: PreferenceKey {
  static let defaultValue: CGFloat = 0
  static func reduce(value: inout CGFloat, nextValue: () -> CGFloat) {
    value = nextValue()
  }
}
struct SettingsForm: View {
  @State private var labelWidth: CGFloat = 0
  let titles = ["Name", "Email address", "Phone"]
  var body: some View {
    VStack(alignment: .leading) {
      ForEach(titles, id: \.self) { title in
        Text(title)
          .background(GeometryReader { Color.clear.preference(key: LabelWidthKey.self, value: $0.size.width) })
          .frame(width: labelWidth == 0 ? nil : labelWidth, alignment: .leading)
      }
    }
    .onPreferenceChange(LabelWidthKey.self) { labelWidth = $0 }
  }
}
```

`reduce` is called once per child that sets the preference, and `value = nextValue()` overwrites: only the last row's width survives, so `labelWidth` becomes the width of "Phone" and every label is squeezed into it.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69220)

### 12. Why does `onAppear` in a `LazyVStack` fire before the row is on screen, and how do you build pagination that survives it?

A lazy stack builds rows a little before they are visible (the exact distance is not documented), and `onAppear` fires on creation, so it means "built", not "visible". Make the load trigger idempotent or use a real visibility API.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69210)

### 13. Why does `proxy.scrollTo(id)` called right after appending a message often miss, and what can you use instead of `ScrollViewReader` on iOS 17–18?

`scrollTo` looks for the id in the current view tree, and the row you just appended exists only after SwiftUI applies the state change, so the call resolves against the old content. The state-driven alternative is to make the scroll position state with `.scrollPosition` and `scrollTargetLayout()`.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69211)

### 14. What breaks when you put `.ignoresSafeArea()` on a screen's root, and how do you let only the background reach the edges?

`ignoresSafeArea` offers a larger area to the subtree it is attached to, so on the root it moves all content under the status bar and the home indicator. With no arguments it also ignores the keyboard region, which turns off keyboard avoidance.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69214)

### 15. How would you build a tag layout that wraps onto new lines with the `Layout` protocol, and what must `sizeThatFits` and `placeSubviews` do?

`sizeThatFits` answers a proposal with the size the container needs, measuring children through `LayoutSubview`; `placeSubviews` gets the final bounds and must place every subview exactly once. Both must use the same arithmetic, or children draw outside the size you reported.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69206)

### 16. Find the bug: this tag layout draws its tags in the top-left corner of the screen instead of under the header in the `VStack`.

```swift
struct TagFlow: Layout {
  var spacing: CGFloat = 8
  func sizeThatFits(proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) -> CGSize {
    measure(subviews, maxWidth: proposal.width ?? .infinity) // correct: sums the rows, no origin involved
  }
  func placeSubviews(in bounds: CGRect, proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) {
    var x: CGFloat = 0, y: CGFloat = 0, rowHeight: CGFloat = 0
    for subview in subviews {
      let size = subview.sizeThatFits(.unspecified)
      if x + size.width > bounds.width, x > 0 { x = 0; y += rowHeight + spacing; rowHeight = 0 }
      subview.place(at: CGPoint(x: x, y: y), proposal: ProposedViewSize(size))
      x += size.width + spacing
      rowHeight = max(rowHeight, size.height)
    }
  }
}
```

`placeSubviews` places the tags starting from `(0, 0)`, but `bounds` is the region the parent gave the layout, in the parent's coordinate space, and its origin is rarely zero. `place(at:)` uses that same space, so the tags ignore the header and padding above them.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69221)

### 17. What do `compositingGroup()` and `drawingGroup()` do, and when does each one help or hurt?

`compositingGroup()` makes a subtree render as one layer before the effects above it (opacity, blend mode, shadow) are applied, which fixes correctness. `drawingGroup()` goes further and rasterises the subtree offscreen with Metal into one image, a performance tool for heavy vector drawing.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69213)

### 18. A waveform built from 2,000 `Rectangle` views drops frames: when do you switch to `Canvas`, and what do you give up?

Switch when the elements are just pixels: nothing the user taps, focuses or animates one by one. Each `Rectangle` is a view with identity, layout and hit testing; `Canvas` is a single view that draws everything into a `GraphicsContext` in one pass.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69212)

### 19. Your `Card` container must put a divider between its children, but a `ForEach` passed in counts as one view: how do you reach the real children?

Resolve the content into its subviews: `Group(subviews:)` and `ForEach(subviewOf:)` (iOS 18) turn a `@ViewBuilder` value into a collection of individual children, with any `ForEach` inside already expanded.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69215)

### 20. Bodies are cheap, but the SwiftUI instrument shows time spent in layout and the screen still hitches: what makes layout expensive?

Expensive layout means the same subtree is measured many times per frame: nested flexible stacks, `fixedSize` and `ViewThatFits` ideal-size queries, text re-wrapped for every proposed width, and measurements fed back into state.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69216)

### 21. What is the AttributeGraph, and how does SwiftUI decide which `body` to run again after a state change?

The AttributeGraph is SwiftUI's private dependency graph: state, environment values, view inputs, `body` results and layout are nodes, and edges record who read what. A change marks its dependents dirty, and the next update recomputes only those.

[Full answer →](https://irbisa.com/topic/swiftui-layout-rendering/#q-69217)
