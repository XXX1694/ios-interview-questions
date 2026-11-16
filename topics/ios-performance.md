# iOS Performance & Memory interview questions

ARC, retain cycles, Instruments, main thread, image and table optimization. Senior · 25 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/ios-performance/)** · [All iOS topics](../README.md#all-topics)

### 1. How does ARC work in Swift, and how is it different from a garbage collector?

ARC frees a class instance the moment its strong reference count drops to zero, using retain and release calls the compiler inserts. A garbage collector finds unreachable objects at runtime instead: it costs pauses or background work, but it also frees reference cycles, which ARC never does.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-3501)

### 2. What is the difference between the stack and the heap, and which Swift values live where?

The stack is per-thread memory where allocating is just moving a pointer and everything is freed when the function returns. The heap is shared memory allocated through `malloc`: slower, and for class instances also reference-counted.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63515)

### 3. What is a retain cycle, where does it come from in iOS code, and how do you break it?

A retain cycle is two or more objects holding strong references to each other, so no count ever reaches zero and all of them leak. You break it by making one link `weak` or `unowned`, usually the back reference.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-3502)

### 4. Which Xcode debugging tools do you use day to day, and what is each one for: breakpoints, LLDB, the view debugger and the Memory Graph?

Breakpoints and LLDB stop the app and let you inspect its state, the Debug View Hierarchy explains layout, the Memory Graph finds leaks, and the scheme's runtime checks catch bugs that only show up while the app runs.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63522)

### 5. Which of the four closures in this view controller create a retain cycle, and which ones only delay deallocation?

```swift
final class ProfileViewController: UIViewController {
  var onSave: (() -> Void)?
  private let saveButton = UIButton(type: .system)

  override func viewDidLoad() {
    super.viewDidLoad()
    view.addSubview(saveButton)
    UIView.animate(withDuration: 0.3) { self.view.alpha = 1 }                    // A
    URLSession.shared.dataTask(with: URL(string: "https://example.com/feed")!) { _, _, _ in self.reload() }.resume()  // B
    onSave = { self.reload() }                                                  // C
    saveButton.addAction(UIAction { _ in self.reload() }, for: .touchUpInside)  // D
  }

  func reload() {}
}
```

C and D are cycles; A and B are not. All four capture `self` strongly, but A and B are released once they have run, so they can at most delay `deinit`, while C and D are stored by something `self` owns, which closes the loop.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63520)

### 6. Find the bug: the Memory Graph shows one more `OTPViewController` alive every time the user opens and closes the code screen.

```swift
final class OTPViewController: UIViewController {
  private var timer: Timer?
  private var secondsLeft = 60

  override func viewDidLoad() {
    super.viewDidLoad()
    timer = Timer.scheduledTimer(timeInterval: 1, target: self,
                                 selector: #selector(tick), userInfo: nil, repeats: true)
  }

  @objc private func tick() {
    secondsLeft -= 1
    title = "Resend the code in \(secondsLeft) s"
  }
  deinit { timer?.invalidate() }  // the timer must not outlive the screen
}
```

The run loop retains the scheduled timer and the timer retains its target, `self`, so closing the screen never drops the controller's count to zero. `deinit`, the only place that calls `invalidate()`, never runs: every visit leaks a controller that keeps ticking.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63521)

### 7. What is a memory leak in an iOS app, and why does the Leaks instrument miss most of them?

A leak is memory the app no longer needs but never frees. The Leaks instrument reports only objects nothing can reach, which in practice means retain cycles, while most real growth is still reachable from a cache, an array or a singleton and shows up only in Allocations and the Memory Graph.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63514)

### 8. How should an iOS app react to a memory warning, and why use `NSCache` instead of a `Dictionary`?

Free everything you can rebuild (image caches, decoded data, the heavy state of screens that are not visible), because the system may kill the process next. `NSCache` does much of this for you: it evicts on its own under memory pressure, while a `Dictionary` only grows.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63516)

### 9. Memory climbs to 1.5 GB while this export loop runs and drops the moment it returns. Why, and how do you fix it?

```swift
func exportThumbnails(_ urls: [URL], to folder: URL) throws {
  for url in urls {                                   // 3 000 photos
    guard let image = UIImage(contentsOfFile: url.path),
          let thumb = image.preparingThumbnail(of: CGSize(width: 512, height: 512)),
          let data = thumb.jpegData(compressionQuality: 0.8)
    else { continue }
    try data.write(to: folder.appendingPathComponent(url.lastPathComponent))
  }
}
```

The loop fills an autorelease pool that is drained only when the current run-loop turn or work item ends, so every iteration's temporary images and data stay alive until the whole loop returns. Wrapping the loop body in `autoreleasepool { }` drains them on every iteration.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63517)

### 10. Why do images cause most memory problems in iOS apps, and how do you downsample them?

A decoded image costs width × height × 4 bytes however small the file is, so a 12-megapixel photo takes about 48 MB in memory even as a 2 MB JPEG. Downsampling with ImageIO decodes straight to the display size, so the full bitmap never exists.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63501)

### 11. The user pops this screen, requests keep firing and `deinit` never prints. What is this `Task` doing?

```swift
@MainActor
final class FeedModel {
  var posts: [Post] = []

  init(feed: FeedService) {
    Task {
      for await update in feed.updates {   // a stream that never finishes
        self.posts = update
      }
    }
  }

  deinit { print("FeedModel freed") }      // never prints
}
```

The `Task` holds `self` strongly for as long as its body runs, and the `for await` loop never ends because the stream never finishes. Nobody keeps or cancels the task, so it keeps the model alive forever.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63508)

### 12. How do you find a performance problem with Instruments instead of guessing?

Reproduce the slow case on a real device with a Release build, record it with the template that matches the symptom, and change one thing at a time, comparing the same trace before and after. The Simulator runs on your Mac's CPU, so its timings mean nothing.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-3503)

### 13. What is the difference between a hang and a hitch, and how do you keep the main thread responsive?

A hitch is a frame shown late: visible stutter in a scroll or animation. A hang is the main thread blocked for 250 ms or more, so taps get no response. Both usually mean too much work on the main thread; the fix is to move it off and keep each frame within budget.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-3504)

### 14. How do you make a `UITableView` or `UICollectionView` scroll smoothly?

Keep each cell's work inside the frame budget (about 16 ms at 60 Hz and 8 ms at 120 Hz): reuse cells, load and decode images off the main thread at display size, and avoid layout and rendering work that repeats every frame.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-3505)

### 15. What is offscreen rendering, which layer properties trigger it, and how do you find it?

Offscreen rendering is the GPU drawing a layer subtree into a separate buffer and compositing it back, because it cannot draw it in one pass. Masks, shadows without a `shadowPath`, group opacity and `shouldRasterize` trigger it, and the cost repeats every frame for every such layer.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63511)

### 16. Users report crashes you cannot reproduce. How do you get from a crash report to the line of code?

Symbolicate the report with the dSYM of that exact build, then read the exception type and the crashed thread's stack. The exception type tells you what kind of bug it is before you even open the code.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63518)

### 17. What does `EXC_BAD_ACCESS` mean, and which Xcode tools help you find its cause?

The app touched memory it must not: a freed object, a dangling pointer or an unmapped address. The crashing line is often just where the damage was noticed, so you use tools that stop the app at the moment of the bad access.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63519)

### 18. A crash report shows exception code `0x8badf00d` and no frame of your code. What killed the app, and what usually causes it?

The watchdog killed it: the app did not finish a lifecycle transition, such as launch or returning to the foreground, within its time limit, so the system sent `SIGKILL` and stamped the report `0x8badf00d` ("ate bad food").

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63509)

### 19. How do you track performance on real users' devices and stop a regression before it ships?

Pick a few user-visible numbers and measure them in two places: performance tests in CI on a fixed device catch a regression before release, and MetricKit plus the Xcode Organizer show what real devices experience after it.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63506)

### 20. How does Core Animation turn your UI changes into a frame, and why does a spinner keep spinning while the main thread is frozen?

Your app only describes each frame: at the end of a run-loop turn it commits the changed layer tree to the render server, a separate process that renders it on the GPU and runs the animations. An animation already handed over keeps running even while your main thread is blocked.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63510)

### 21. Xcode's memory gauge shows 480 MB, Allocations shows 90 MB, and the app still gets killed for memory. Which number is real?

The gauge. The system's memory limit applies to `phys_footprint` (all dirty plus compressed memory of the process), while Allocations mostly shows what went through `malloc` and the object runtimes, and misses large memory regions that are not malloc heap.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63512)

### 22. How do you measure and reduce app launch time?

Measure cold launch to the first frame on real devices, then cut the two phases separately: the work `dyld` and the runtime do before `main`, and your own code from `main` until the first screen is drawn. Apple's target is a first frame in about 400 ms.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-3506)

### 23. QA says the app drains the battery. How do you find the cause, and what do you actually change?

Energy goes into waking the hardware (the cellular radio, GPS, the GPU and the CPU) more than into running code, so the fixes are about doing work less often and in batches. Measure on a device first, because the cause is usually one subsystem left running.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63503)

### 24. The app's download size grows with every release. How do you find what takes the space and cut it?

Measure what users actually download (App Store Connect's per-device sizes or Xcode's App Thinning Size Report), then attack the largest part, usually assets and then third-party code. The `.ipa` on your Mac contains every device variant and is not that number.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63504)

### 25. What is thread explosion, and how does Swift Concurrency avoid it as long as you never block a task?

Thread explosion happens when work dispatched to a concurrent queue blocks, so GCD starts another thread for the next item, and then another. Dozens of mostly blocked threads then waste memory and CPU on context switches instead of doing work.

[Full answer →](https://irbisa.com/topic/ios-performance/#q-63505)
