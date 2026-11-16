# Auto Layout & UIKit Layout interview questions

Constraints, intrinsic content size, UIStackView, frames, safe area. Junior · 25 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/uikit-layout/)** · [All iOS topics](../README.md#all-topics)

### 1. What is Auto Layout, and how is it different from frame-based layout?

Auto Layout is a constraint-based layout system: you describe relationships between views ("16 pt below the title, as wide as the card"), and a solver turns them into frames on every layout pass. With frame-based layout you compute and assign every rectangle yourself.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-3101)

### 2. What is the Auto Layout constraint equation, and what are layout anchors?

Every constraint is a linear equation `item1.attribute = multiplier × item2.attribute + constant`, where the relation can also be `≤` or `≥`. Layout anchors (`topAnchor`, `widthAnchor`, …) are the type-safe way to build that equation.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63115)

### 3. How many constraints does a view need before its layout is unambiguous?

Enough to fix four values (x position and width, y position and height), which usually means four constraints, two per axis. A view with an intrinsic content size, such as a `UILabel`, supplies its own size, so two positional constraints are enough.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63116)

### 4. You constrained a view created in code, but it never appears where expected and the console lists broken constraints. What is wrong here?

```swift
override func viewDidLoad() {
  super.viewDidLoad()
  let card = UIView()
  card.backgroundColor = .systemBlue
  view.addSubview(card)
  NSLayoutConstraint.activate([
    card.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 16),
    card.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
    card.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),
    card.heightAnchor.constraint(equalToConstant: 120),
  ])
}
```

`card` still has `translatesAutoresizingMaskIntoConstraints` set to `true`, so UIKit turns its frame (`.zero`) and autoresizing mask into constraints that contradict yours. Set `card.translatesAutoresizingMaskIntoConstraints = false` before activating the constraints.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63101)

### 5. This setup code crashes with "Unable to activate constraint ... because they have no common ancestor". What did it do in the wrong order?

```swift
let badge = UILabel()
badge.text = "NEW"
badge.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
  badge.topAnchor.constraint(equalTo: card.topAnchor, constant: 8),
  badge.trailingAnchor.constraint(equalTo: card.trailingAnchor, constant: -8),
])

card.addSubview(badge)
```

It activates the constraints before `badge` is in the view hierarchy. Activation installs a constraint on the nearest common ancestor of both views, and a view without a superview has none, so UIKit throws. Call `card.addSubview(badge)` first, then activate.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63108)

### 6. What is the safe area, and how do you keep content inside it in UIKit?

The safe area is the part of a view not covered by the status bar, navigation and tab bars, the home indicator or the Dynamic Island. Pin content to `view.safeAreaLayoutGuide` and it stays visible on every device and in every orientation.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-3104)

### 7. Why do you constrain to `leadingAnchor` and `trailingAnchor` rather than `leftAnchor` and `rightAnchor`?

Leading and trailing follow the reading direction, so the layout mirrors automatically in Arabic and Hebrew; left and right are physical sides and never flip.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63106)

### 8. What is `UIStackView`, and what are its advantages and drawbacks?

`UIStackView` arranges its subviews in a row or a column and creates the constraints between them for you: you set `axis`, `distribution`, `alignment` and `spacing`, and constrain only the stack itself.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-3103)

### 9. In a horizontal `UIStackView` every item came out the same width and the icon is stretched to the row's height. Which two properties are wrong?

`distribution` sizes views along the axis and `alignment` positions them across it: the equal widths come from `distribution = .fillEqually`, the stretched icon from the default `alignment = .fill`.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63107)

### 10. What is intrinsic content size, and how do content hugging and compression resistance decide which view gives way?

Intrinsic content size is the size a view wants from its own content: a label's text, a button's title plus insets. Content hugging resists growing beyond it, compression resistance resists shrinking below it, and the view with the lower priority gives way first.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-3102)

### 11. What do `@IBDesignable` and `@IBInspectable` do, and what do teams use instead of live rendering today?

`@IBInspectable` exposes a custom view's property in Interface Builder's Attributes inspector, and `@IBDesignable` makes Interface Builder build the view and render it on the canvas. Today UIKit views are usually previewed with the `#Preview` macro (Xcode 15+) instead.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63119)

### 12. What are constraint priorities, and why do people use 999 instead of 1000 for a constraint that must hold?

A priority from 1 to 1000 says how hard the solver tries to satisfy a constraint: 1000 (`.required`) must hold, anything lower is optional: the solver satisfies it as closely as it can and gives way without a log message when it conflicts with something stronger. 999 means "as strong as possible, but allowed to yield".

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63117)

### 13. This badge view draws the new number after `count` changes but keeps its old width. What is missing?

```swift
final class BadgeView: UIView {
  var count = 0 {
    didSet { setNeedsDisplay() }
  }

  override var intrinsicContentSize: CGSize {
    let text = "\(count)" as NSString
    let font = UIFont.preferredFont(forTextStyle: .caption1)
    let size = text.size(withAttributes: [.font: font])
    return CGSize(width: ceil(size.width) + 16, height: ceil(size.height) + 8)
  }

  override func draw(_ rect: CGRect) {
    // draws the pill and the number
  }
}
```

`didSet` never calls `invalidateIntrinsicContentSize()`. Auto Layout caches the intrinsic size and re-reads it only after that call; `setNeedsDisplay()` just repaints the same rectangle.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63109)

### 14. What is the difference between pinning to a view's edge, its `layoutMarginsGuide` and its `readableContentGuide`?

The edge is the view's literal bounds, `layoutMarginsGuide` is the bounds minus the view's margins, and `readableContentGuide` additionally caps the width so lines of text stay a comfortable length on wide screens.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63105)

### 15. What is the difference between `setNeedsLayout()`, `layoutIfNeeded()` and `layoutSubviews()`?

`setNeedsLayout()` marks a view as needing layout in the next update cycle, `layoutIfNeeded()` performs any pending layout right now, and `layoutSubviews()` is the method UIKit calls during that layout; you call the first two and override the third.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-3106)

### 16. A layout must change when some state flips. Do you change the `constant`, swap active constraints, or change a priority?

Change the `constant` when only a number changes (it animates cleanly), swap active constraint sets when the relationships themselves change, and change priorities when two competing constraints should take turns winning.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63104)

### 17. Find the bug: the panel should slide up smoothly, but it jumps to its new position with no animation.

```swift
final class PlayerViewController: UIViewController {
  private let panel = UIView()
  // activated in viewDidLoad with leading, trailing and height 300
  private lazy var panelBottom = panel.bottomAnchor.constraint(
    equalTo: view.bottomAnchor, constant: 300)  // starts off-screen

  func showPanel() {
    panelBottom.constant = 0
    UIView.animate(withDuration: 0.3) {
      self.panel.layoutIfNeeded()
    }
  }
}
```

`layoutIfNeeded()` is called on `panel`, but the changed constraint positions `panel` inside `view`, so the block moves nothing. The panel jumps on the next layout pass, outside the animation, instead of sliding up.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63121)

### 18. The console prints "Unable to simultaneously satisfy constraints". How do you find the culprit?

Read the log: it lists the set of constraints that cannot all hold and names the one it broke. Give your constraints identifiers so the log shows names, then use the `UIViewAlertForUnsatisfiableConstraints` breakpoint and the View Debugger to see which code and which views are involved.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-3105)

### 19. What has to be true for a table view cell to size itself to its content?

There must be an unbroken chain of vertical constraints from the top of `contentView` to its bottom, so Auto Layout can compute the height; the table then uses `UITableView.automaticDimension`, which is already the default.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63102)

### 20. How do you lay out content inside a `UIScrollView` with Auto Layout so that it actually scrolls?

A scroll view computes its content size from its content's constraints, so pin the content to all four edges of `contentLayoutGuide` and take its width from `frameLayoutGuide`.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63103)

### 21. A text field at the bottom of a form ends up under the keyboard. What is the current UIKit way to lift it, and what did the old notification code get wrong?

Constrain the field's bottom to `view.keyboardLayoutGuide.topAnchor`. Since iOS 15 that guide tracks the keyboard in your view's coordinates and animates with it, so one constraint replaces the whole notification pipeline.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63111)

### 22. What are size classes, and how do you switch between a two-column iPad layout and a one-column iPhone layout?

Size classes are coarse traits (`.compact` or `.regular`) for the width and height your view actually gets. Branch on `traitCollection.horizontalSizeClass`, not on the device: an iPad app in Split View or a narrow window gets a compact width.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63113)

### 23. How do you handle device rotation in a UIKit layout?

Mostly you do not: constraints relative to the parent and the safe area re-solve for the new size on their own. Code is needed only when the structure changes, and it belongs in `viewWillTransition(to:with:)` or a size-class handler, not in device-orientation notifications.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63118)

### 24. One screen pins the CPU at 100% and freezes while `layoutSubviews()` runs over and over. What causes an Auto Layout feedback loop?

Something inside the layout pass invalidates layout again: constraints are changed from geometry that the same pass just produced, so every pass schedules the next one.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63112)

### 25. A scrolling list drops frames and Instruments shows the time in Auto Layout. What do you check and what do you change?

Find out how much layout work each cell does per scroll step, then cut it: fewer views and constraints per cell, no constraint changes on reuse, and frames for the few hot cells that still cost too much.

[Full answer →](https://irbisa.com/topic/uikit-layout/#q-63120)
