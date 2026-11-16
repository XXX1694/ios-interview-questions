# UIKit Fundamentals interview questions

UIView, UIViewController lifecycle, view hierarchy, touch handling. Junior · 25 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/uikit-fundamentals/)** · [All iOS topics](../README.md#all-topics)

### 1. What is `UIWindow`, how does the view hierarchy start, and how is a window different from an ordinary `UIView`?

`UIWindow` is the root of a scene's view hierarchy: a `UIView` subclass with no superview that holds the `rootViewController` and receives events from `UIApplication` to pass to the views inside it.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-3001)

### 2. Walk through the `UIViewController` lifecycle: which callbacks run once, which run on every appearance, and where do you start loading data?

`loadView` and `viewDidLoad` run once per controller instance; `viewWillAppear`, `viewIsAppearing`, `viewDidAppear` and the two disappear callbacks run every time the screen is shown or hidden.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-3002)

### 3. What is `loadView`, when do you override it, and how is it different from `viewDidLoad`?

`loadView` creates the controller's root view and assigns it to `view`; `viewDidLoad` runs right after it and configures a view that already exists. You override `loadView` only to make your own view class the root view.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63015)

### 4. What is the difference between `frame` and `bounds`, and when is `bounds.origin` not zero?

`frame` is the view's position and size in its superview's coordinate system; `bounds` is the same rectangle in the view's own coordinates. `bounds.origin` is non-zero when the view shows a shifted part of its content: a scroll view's `bounds.origin` is its `contentOffset`.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-3003)

### 5. What are the trade-offs between storyboards, XIBs and building the UI in code?

All three produce the same UIKit views: storyboards and XIBs give you a visual editor, code gives readable diffs and far fewer merge conflicts. Most teams now build new screens in code or SwiftUI.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-3006)

### 6. What is the difference between `@IBOutlet` and `@IBAction`, and how does target-action connect a control to your code?

`@IBOutlet` is a property Interface Builder fills with a reference to a view, so code can read and change that view; `@IBAction` is a method Interface Builder connects to a control event, so the control can call your code. The second one is target-action.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63016)

### 7. How does `UINavigationController` work, and what is the difference between pushing and presenting a view controller?

`UINavigationController` manages a stack of view controllers: `push` adds one on top with a back button and swipe-back, `pop` removes it. `present` shows a controller modally above the current one, outside any stack, until it is dismissed.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63003)

### 8. How does `UITableView` work: what do the data source and the delegate do, and what is cell reuse?

A table view asks its data source how many rows there are and for a cell for each visible row, and tells its delegate about selection, sizing and scrolling. Only on-screen cells exist; cells that scroll off go into a reuse queue and come back from `dequeueReusableCell(withIdentifier:for:)`.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-3005)

### 9. How does `UIScrollView` work: what are `contentSize`, `contentOffset` and `contentInset`?

A scroll view is a view whose `bounds.origin` moves as the user drags: `contentSize` is how big the scrollable content is, `contentOffset` is which point of it sits at the top-left corner, and `contentInset` adds extra scrollable space around it.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63017)

### 10. What is the difference between `UIImage` and `UIImageView`, and between `.scaleAspectFit` and `.scaleAspectFill`?

`UIImage` is image data (pixels and scale) that is not on screen; `UIImageView` is a view that shows it using `contentMode`. `.scaleAspectFit` shows the whole image with empty bands, `.scaleAspectFill` fills the view and crops the overflow.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63018)

### 11. How do you round a view's corners and add a shadow, and why do the two conflict?

Round with `layer.cornerRadius` plus `clipsToBounds` and cast the shadow with `layer.shadowOpacity`/`shadowRadius`/`shadowOffset`; clipping cuts the shadow off, so use two views: an outer one with the shadow and an inner one that clips.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63008)

### 12. This label is updated from a `URLSession` callback and the app later crashes at random. What is wrong, and how do you fix it?

```swift
func loadProfile() {
  URLSession.shared.dataTask(with: profileURL) { data, _, _ in
    guard let data else { return }
    let profile = try? JSONDecoder().decode(Profile.self, from: data)
    self.nameLabel.text = profile?.name
    self.view.setNeedsLayout()
  }.resume()
}
```

The `dataTask` completion handler runs on a background queue, and UIKit may only be used on the main thread. Update the UI on the main thread: wrap it in `DispatchQueue.main.async`, or use async/await from a main-actor context.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63009)

### 13. How do you support Dark Mode in UIKit, and why does a layer's border colour not switch along with it?

Use dynamic colours (system semantic colours such as `.label` and `.systemBackground`, or asset catalog colours with a dark variant), and UIKit re-resolves them on every appearance change. A `CGColor` set on a layer is resolved once, so you have to set it again when the trait changes.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63012)

### 14. How do you animate a view in UIKit, and how do you animate an Auto Layout constraint change?

Put the property changes inside `UIView.animate(withDuration:animations:)`. For Auto Layout, change the constraint's `constant` first, then call `layoutIfNeeded()` on the superview inside the animation block; it is the layout pass that gets animated.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63005)

### 15. What is `viewIsAppearing(_:)`, and why is it a better place for geometry-dependent setup than `viewWillAppear` or `viewDidLayoutSubviews`?

`viewIsAppearing` runs once per appearance, after the view is in the hierarchy with its final size and traits but before it is on screen. `viewWillAppear` is too early for geometry, and `viewDidLayoutSubviews` runs on every layout pass.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63011)

### 16. Find the bug: after a few visits to the cart screen, a single cart change is logged to analytics several times.

```swift
final class CartViewController: UIViewController {
  override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    NotificationCenter.default.addObserver(
      self, selector: #selector(cartDidChange),
      name: .cartDidChange, object: nil)
    reloadBadge()
  }

  @objc private func cartDidChange() {
    reloadBadge()
    analytics.log("cart_changed")
  }
}
```

The observer is added in `viewWillAppear`, which runs on every appearance, and is never removed. `NotificationCenter` does not deduplicate registrations, so after three visits one `.cartDidChange` calls `cartDidChange()` three times: three analytics events, three reloads.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63024)

### 17. How do you pass data forward to the next view controller, and back to the previous one?

Forward, give the next controller what it needs before showing it, ideally through its `init`. Back, the child reports through a callback the parent set (a closure or a weak delegate), never through a direct reference to the parent's type.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63019)

### 18. What is view controller containment, and which calls do you make to add and remove a child controller?

Containment lets one view controller embed another's view inside its own while the child stays a real controller with its own lifecycle. Add it with `addChild`, add its view, then call `didMove(toParent:)`; remove it in the mirror order.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63020)

### 19. What is the responder chain, and how does `hitTest(_:with:)` decide which view receives a touch?

UIKit first hit-tests from the window down to find the deepest view under the finger and delivers the touch there; whatever that view does not handle travels up the responder chain: superview, view controller, window, application.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-3004)

### 20. How do gesture recognizers work, and how do you make a single tap wait for a double tap or a pan work alongside a scroll view?

A gesture recognizer is a state machine attached to a view: it watches the touches and calls its action when it recognizes its gesture. By default only one recognizer wins; `require(toFail:)` sets an order, and the delegate's `shouldRecognizeSimultaneouslyWith` lets two run together.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63010)

### 21. What is the difference between `UIView` and `CALayer`, and when do you work with a layer directly?

Every `UIView` is backed by a `CALayer`: the view handles touches, layout, the responder chain and accessibility, while the layer holds the content that Core Animation draws and animates. Layers are lighter, but they cannot receive touches.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63002)

### 22. What is the difference between `setNeedsLayout` and `setNeedsDisplay`, and when does `draw(_:)` run?

`setNeedsLayout` marks a view for a new layout pass, so `layoutSubviews` runs; `setNeedsDisplay` marks its content as stale, so `draw(_:)` runs to redraw it. Neither acts immediately; both are batched until the end of the current run loop iteration.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63021)

### 23. How does the main run loop work, and why does a `Timer` stop firing while the user scrolls?

The main run loop is the main thread's event loop: it handles touches, timers and queued blocks, then commits layout and drawing. A scheduled `Timer` fires only in `.default` mode, while scrolling runs the loop in `.tracking` mode, so the timer waits.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63022)

### 24. This controller is dismissed 3 seconds after it opens. What prints, when, and is this a memory leak?

```swift
final class DetailViewController: UIViewController {
  private let statusLabel = UILabel()

  override func viewDidLoad() {
    super.viewDidLoad()
    DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
      self.statusLabel.text = "Loaded"
      print("closure ran")
    }
  }

  deinit { print("deinit") }
}
```

Nothing prints at 3 seconds. At 5 seconds the closure runs and prints "closure ran", then releases `self`, and "deinit" prints. The closure keeps the dismissed controller alive 2 seconds longer, but there is no retain cycle, so it is not a leak.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63023)

### 25. Find the bug: while the user scrolls fast, rows briefly show another user's avatar, and sometimes the wrong one stays.

```swift
override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
  let cell = tableView.dequeueReusableCell(withIdentifier: "Avatar", for: indexPath) as! AvatarCell
  let user = users[indexPath.row]
  cell.nameLabel.text = user.name
  Task {
    let image = await imageLoader.image(for: user.avatarURL)
    cell.avatarView.image = image
  }
  return cell
}
```

The `Task` captures the cell, not the row. Cells are reused, so when a slow download finishes the cell may already show another user and gets the wrong avatar; the previous image also stays visible until the new one arrives.

[Full answer →](https://irbisa.com/topic/uikit-fundamentals/#q-63025)
