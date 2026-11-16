# Animations & Core Animation interview questions

CALayer, implicit animations, UIViewPropertyAnimator, transitions, PhaseAnimator, KeyframeAnimator. Senior · 20 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/ios-animations/)** · [All iOS topics](../README.md#all-topics)

### 1. This dot slides to x = 300 and stays there. What does `print` show a second later, and why does moving it to 100 change nothing on screen?

```swift
let dot = ballView.layer                 // ballView is centred at x = 40

let slide = CABasicAnimation(keyPath: "position.x")
slide.fromValue = 40
slide.toValue = 300
slide.duration = 0.5
slide.fillMode = .forwards
slide.isRemovedOnCompletion = false
dot.add(slide, forKey: "slide")

DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
  print(dot.position.x)
  dot.position.x = 100                   // nothing moves on screen
}
```

It prints 40. The animation changed only the presentation layer, and `fillMode = .forwards` with `isRemovedOnCompletion = false` freezes that picture over a model that never moved, and while the frozen animation stays attached, it overrides every new model value.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67218)

### 2. Why does changing a property on a standalone `CALayer` animate by itself, while the same change on a view's backing layer just snaps?

Both changes go through the same action search and get different answers: a standalone layer has no delegate and falls through to the default 0.25 s animation, while a backing layer asks its `UIView`, which returns `NSNull` (no animation) unless the change is inside an animation block.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67202)

### 3. What is `CATransaction`, and how do you stop a single layer change from animating?

`CATransaction` is the batch that carries layer changes to the render server: it sets their animation duration, timing function and completion, and commits them together. To keep one change from animating, wrap it in a transaction with `setDisableActions(true)`.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67203)

### 4. Where is the card drawn with transform `a` and with transform `b`, and what does `print` show each time?

```swift
// card is 100 x 100, centred at (200, 200), with an identity transform
let a = CGAffineTransform(translationX: 100, y: 0).rotated(by: .pi / 2)
let b = CGAffineTransform(rotationAngle: .pi / 2).translatedBy(x: 100, y: 0)

card.transform = a
print(card.center)      // what prints, and where is the card drawn?

card.transform = b
print(card.center)      // and now?
```

With `a` the card is drawn 100 pt to the right, centred at (300, 200); with `b` it is 100 pt lower, at (200, 300); both are rotated 90°. `print` shows (200, 200) both times, because `transform` changes only how the view is drawn, not its `center` or `bounds`.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67219)

### 5. Which `CALayer` subclasses do you use, and why build an animated progress ring with `CAShapeLayer` instead of `draw(_:)`?

Specialised layers (`CAShapeLayer`, `CAGradientLayer`, `CATextLayer`, `CAReplicatorLayer`, `CAEmitterLayer`) describe content the render server draws itself. `draw(_:)` instead gives the view a bitmap that the CPU repaints and uploads again on every change.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67213)

### 6. What is `CADisplayLink`, when do you need it instead of Core Animation, and how do you get 120 Hz from it on a ProMotion iPhone?

`CADisplayLink` is a timer synchronised with the display: it calls you once per frame so you can compute that frame's value yourself. Use it only when Core Animation cannot interpolate the thing for you (physics, a live waveform, a custom renderer).

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67212)

### 7. What does `UIViewPropertyAnimator` give you over `UIView.animate`, and how do you drive one from a pan gesture?

`UIView.animate` is fire-and-forget; `UIViewPropertyAnimator` is an object you keep, so you can pause it, scrub `fractionComplete`, reverse it, add animations or completions while it runs and finish it with new timing, which is what interactive, interruptible UI needs.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67205)

### 8. Find the bug: tapping Filters should slide the panel open, but it jumps to full height with no animation.

```swift
final class FiltersViewController: UIViewController {
  let panel = UIView()
  private var panelHeight: NSLayoutConstraint!   // panel.heightAnchor, starts at 0
  private var isExpanded = false

  @objc func toggleFilters() {
    isExpanded.toggle()
    UIView.animate(withDuration: 0.3) {
      self.panelHeight.constant = self.isExpanded ? 320 : 0
    }
  }
}
```

Changing `constant` moves nothing by itself: it only marks the layout as needing an update. The frames change in the next layout pass, after the animation block has returned, so the panel snaps to its new height.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67221)

### 9. How do you implement a custom view controller transition, and who adds and removes the views in the container?

Return an object conforming to `UIViewControllerAnimatedTransitioning` from the navigation or transitioning delegate. UIKit calls its `animateTransition(using:)`: you add the incoming view to `containerView`, animate, and call `completeTransition(_:)`; UIKit removes the outgoing view.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67207)

### 10. When does a SwiftUI `.transition` actually run, and why does a view sometimes just appear without it?

A transition runs only when a view is inserted into or removed from the hierarchy (its identity changes) and that change is animated. Without an animation the view simply appears; without an identity change there is nothing to transition.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67208)

### 11. How do you run code when a SwiftUI animation finishes, and how do you chain one animation after another?

Since iOS 17, `withAnimation(_:completionCriteria:_:completion:)` calls a closure when the animation is done, and you chain by starting the next `withAnimation` inside it. Before that the only option was a delay equal to the duration, which breaks with springs and interruptions.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67220)

### 12. What is the difference between `phaseAnimator` and `keyframeAnimator`, and when is a `repeatForever` animation enough?

`phaseAnimator` steps through a list of states, moving every property to the next state together; `keyframeAnimator` gives each property its own timeline of keyframes, running in parallel. `repeatForever` is one animation between two values, looped: fine for a pulse, not for a sequence.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67210)

### 13. This `Ring` jumps to the new progress instead of animating: why, and what do `Animatable` and `animatableData` have to do with it?

```swift
struct Ring: Shape {
  var progress: Double                    // 0...1

  func path(in rect: CGRect) -> Path {
    var path = Path()
    path.addArc(center: CGPoint(x: rect.midX, y: rect.midY), radius: rect.width / 2,
                startAngle: .degrees(-90), endAngle: .degrees(-90 + 360 * progress),
                clockwise: false)
    return path
  }
}

// Ring(progress: done).stroke(.teal, lineWidth: 8)
//   .animation(.easeOut(duration: 0.6), value: done)
```

SwiftUI interpolates only what a type exposes through `Animatable.animatableData`. `Shape` is `Animatable`, but its default `animatableData` is `EmptyAnimatableData`, so `progress` is never interpolated and the ring is redrawn once, at the final value.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67211)

### 14. What is the difference between implicit and explicit Core Animation animations, and what are `CABasicAnimation`, `CAKeyframeAnimation` and `CAAnimationGroup`?

An implicit animation is one Core Animation creates for you when you change an animatable layer property; an explicit one is a `CAAnimation` you build and attach with `layer.add(_:forKey:)`. Explicit animations change only the presentation layer; setting the model value stays your job.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67217)

### 15. What is the difference between a layer's model and presentation layers, and why can't the user tap a view while it animates?

The model layer holds the values you set (during an animation, already the end values), while `presentation()` returns a copy with what is on screen this frame. UIKit hit-tests the model and by default disables interaction on animating views, so taps on the moving pixels go nowhere.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67201)

### 16. How does `matchedGeometryEffect` work, and why does a card that should grow into a detail view just fade and snap instead?

`matchedGeometryEffect` does not morph content: it takes the frame of the view marked as source and applies it to the other view with the same id and namespace. It animates only when both views exist in the same animated transaction, in one namespace, with exactly one source at a time.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67209)

### 17. How do you hand a gesture's release velocity to a spring animation, and what do a spring's `duration` and `bounce` mean?

Pass the finger's velocity as the spring's initial velocity, so the motion continues instead of restarting from rest. `duration` is the perceived settling time and `bounce` the amount of overshoot, and unlike mass, stiffness and damping the two can be tuned independently.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67214)

### 18. How do you make a custom transition interactive, and what must happen when the user cancels it halfway?

Return an interaction controller (usually `UIPercentDrivenInteractiveTransition`) from the delegate and drive it from the gesture with `update(_:)`, then `finish()` or `cancel()`. On cancel the animator must undo its changes and report `completeTransition(false)`, or the screen is left frozen.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67206)

### 19. Which layer properties are cheap to animate, and what forces the render server into offscreen rendering?

Animating `transform`, `opacity`, `position` or `bounds` of an already drawn layer is cheap: the app commits once and the render server interpolates every frame. It gets expensive when each frame needs an offscreen pass on the GPU or new pixels from the CPU.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67204)

### 20. An animation hitches on a real iPhone: how do you find out whether your app's commit or the render server is late?

Profile a release build on the device with the Animation Hitches template in Instruments: it attributes each hitch to the commit phase in your process or to the render phase in the render server. Commit-side means main-thread work; render-side means the layer tree is expensive to draw.

[Full answer →](https://irbisa.com/topic/ios-animations/#q-67215)
