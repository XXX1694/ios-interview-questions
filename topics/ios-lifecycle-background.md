# App Lifecycle, Background & Push interview questions

Cold launch, scene phases, background modes, BGTaskScheduler, APNs, silent push, universal links. Middle · 24 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/ios-lifecycle-background/)** · [All iOS topics](../README.md#all-topics)

### 1. What states can an iOS app be in (not running, inactive, active, background, suspended), and what moves it from one to the next?

An app is not running, inactive (on screen but not receiving events), active (on screen and receiving events), in the background (off screen, still running code) or suspended (in memory, running nothing). Launches, interruptions, leaving the app and memory pressure move it between them.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67125)

### 2. What is the difference between `AppDelegate` and `SceneDelegate`, and why did iOS 13 move the UI lifecycle into scenes?

`AppDelegate` handles the process: launch, push tokens, background `URLSession` events, choosing a scene configuration. Each `SceneDelegate` owns one window and its foreground and background lifecycle. iOS 13 split them so that one app can show several windows, each in its own state.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67126)

### 3. How do you observe the app lifecycle in SwiftUI with `scenePhase`, and when do you still need `@UIApplicationDelegateAdaptor`?

Read `@Environment(\.scenePhase)` and react in `.onChange(of:)`: `.active`, `.inactive` and `.background` match the UIKit scene states, and `@UIApplicationDelegateAdaptor` brings back an `AppDelegate` for the callbacks SwiftUI has no modifier for.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67118)

### 4. Is `applicationWillTerminate` guaranteed to run, and where should you save state instead?

No. It runs only if the app is terminated while still executing, and most apps die while suspended, when the system kills them with no callback at all. Save in `sceneDidEnterBackground` (or on `scenePhase == .background`), the last moment you are guaranteed to run.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67102)

### 5. The system killed your app in the background and the user returns to the root screen, not the half-filled form. How do you restore UI state in a scene-based app?

Save a small description of where the user was, not the data itself: UIKit asks each scene for an `NSUserActivity` in `stateRestorationActivity(for:)` and hands it back as `session.stateRestorationActivity` on the next connect; SwiftUI does the same per window with `@SceneStorage`.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67127)

### 6. What is the difference between local and remote notifications, and how do you ask the user for permission?

A local notification is scheduled by the app on the device with a trigger; a remote one is sent by your server through APNs to a device token. Both look the same to the user and share one permission, requested with `UNUserNotificationCenter.requestAuthorization(options:)`.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67119)

### 7. Session counts doubled after this shipped. Why, and which callback should start a session?

```swift
final class SceneDelegate: UIResponder, UIWindowSceneDelegate {
  var window: UIWindow?

  func sceneDidBecomeActive(_ scene: UIScene) {
    Analytics.log("session_start")
  }

  func sceneDidEnterBackground(_ scene: UIScene) {
    Analytics.log("session_end")
  }
}
```

`sceneDidBecomeActive` fires again after every interruption that only makes the app inactive (a Face ID prompt, Control Center, a system alert), so it logs extra starts with no matching end. Start a session in `sceneWillEnterForeground`, the counterpart of `sceneDidEnterBackground`.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67101)

### 8. What happens between the user tapping the icon and `application(_:didFinishLaunchingWithOptions:)`?

The system creates the process, `dyld` loads and links every dynamic library, the Objective-C runtime and static initialisers run, and then `main` calls `UIApplicationMain`, which creates `UIApplication` and your app delegate and only then calls `willFinishLaunching` and `didFinishLaunching`.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67120)

### 9. How does iOS limit background work, and what legitimate ways are there to run code after the user leaves the app?

After the user leaves, an app gets a few seconds in the background and is then suspended. Anything longer needs a sanctioned mechanism: a background task, a `BGTaskScheduler` task, a background `URLSession`, a push, or a background mode used for its stated purpose.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67104)

### 10. This code keeps an upload alive after the user leaves the app. What is wrong with it?

```swift
func sceneDidEnterBackground(_ scene: UIScene) {
  let app = UIApplication.shared
  let id = app.beginBackgroundTask(withName: "upload")

  uploader.upload(draft) { result in
    if case .success = result {
      app.endBackgroundTask(id)
    }
  }
}
```

There is no expiration handler and the task is ended only on success, so a slow or failed upload leaves the identifier open; when the budget runs out, the watchdog kills the app with `0x8badf00d`, which users see as a crash on switching away.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67103)

### 11. Find the bug: a user checks another app for a few minutes mid-workout, and on return this timer is minutes behind.

```swift
struct WorkoutTimerView: View {
  @State private var elapsed = 0
  private let ticker = Timer.publish(every: 1, on: .main, in: .common).autoconnect()

  var body: some View {
    Text(Duration.seconds(elapsed).formatted(.time(pattern: .minuteSecond)))
      .monospacedDigit()
      .onReceive(ticker) { _ in
        elapsed += 1
      }
  }
}
```

The view counts ticks instead of measuring time. In the background the app is suspended within seconds and the timer stops firing, so every minute spent away is lost and the workout comes back short by exactly that time.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67123)

### 12. How do you schedule work with `BGTaskScheduler`, and why might a registered task never run?

Register a handler for the identifier before `didFinishLaunching` returns, list the identifier in `Info.plist`, then `submit` a `BGAppRefreshTaskRequest` or `BGProcessingTaskRequest`. The system decides when to run it, and a few silent setup mistakes mean it never does.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67105)

### 13. How do you test a `BGTaskScheduler` task without waiting for the system to run it?

On a device, background the app after it has submitted the request, pause in Xcode and call the private `_simulateLaunchForTaskWithIdentifier:` from LLDB; the registered handler runs immediately. `_simulateExpirationForTaskWithIdentifier:` fires the expiration handler the same way.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67106)

### 14. How do push notifications work end to end: APNs, the device token and the payload?

The app asks APNs for a device token and sends it to your server; the server posts a JSON payload for that token to APNs over HTTP/2, authenticated with a `.p8` key or a certificate, and APNs delivers it so iOS can show it or wake the app.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67107)

### 15. What does a silent push need to wake your app, and why might it still not arrive?

The payload needs `"content-available": 1` and nothing visible, the request needs `apns-push-type: background` and `apns-priority: 5`, and the app needs the `remote-notification` background mode. Even then the system treats it as a hint and may delay or drop it.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67108)

### 16. Find the bug in this silent push handler: the push arrives, yet the inbox is still stale when the user opens the app.

```swift
func application(_ application: UIApplication,
                 didReceiveRemoteNotification userInfo: [AnyHashable: Any],
                 fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
  guard userInfo["reason"] as? String == "inbox" else {
    completionHandler(.noData)
    return
  }
  Task {
    try await inbox.sync()
  }
  completionHandler(.newData)
}
```

`completionHandler(.newData)` runs right after the `Task` is created, before the sync has done anything. It tells iOS the work is finished, so the app is suspended soon after and the request freezes mid-flight; the inbox catches up only on the next foreground.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67124)

### 17. How do you show or hide a notification that arrives while the app is open, and how do taps and action buttons reach `userNotificationCenter(_:didReceive:)`?

Implement `userNotificationCenter(_:willPresent:)` to decide whether a foreground notification is shown, and register a `UNNotificationCategory` with actions at launch; the payload's `category` key picks the buttons, and `userNotificationCenter(_:didReceive:)` handles taps and actions.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67110)

### 18. What does provisional notification authorization give you, and why can an `.authorized` user still see nothing?

Adding `.provisional` to `requestAuthorization` grants permission without any prompt, but notifications go quietly to Notification Center, with no banner or sound, until the user taps Keep. And `.authorized` only means the app may post, not that banners are switched on.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67111)

### 19. What is the difference between universal links and custom URL schemes, and what does a universal link need to open your app?

A custom scheme like `myapp://` is unverified (any app can claim it and it fails when the app is not installed), while a universal link is an ordinary `https` URL that opens the app only after iOS verifies the domain, and opens the website otherwise.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67112)

### 20. Universal links open the right screen while the app is running, but after a cold start they land on the home screen. What is missing here?

```swift
final class SceneDelegate: UIResponder, UIWindowSceneDelegate {
  var window: UIWindow?

  func scene(_ scene: UIScene, willConnectTo session: UISceneSession,
             options connectionOptions: UIScene.ConnectionOptions) {
    guard let windowScene = scene as? UIWindowScene else { return }
    window = UIWindow(windowScene: windowScene)
    window?.rootViewController = RootViewController()
    window?.makeKeyAndVisible()
  }

  func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
    guard let url = userActivity.webpageURL else { return }
    router.open(Route(url) ?? .home)
  }
}
```

On a cold launch the link does not come through `scene(_:continue:)`: it arrives in `connectionOptions.userActivities` inside `scene(_:willConnectTo:options:)`, and this code ignores it.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67122)

### 21. How can iOS launch your app straight into the background, and what must `didFinishLaunching` avoid on that path?

A silent push, a `BGTaskScheduler` task, a finished background `URLSession`, or a location or Bluetooth event can start the process with no UI. The same `didFinishLaunching` runs, but no scene connects and no UI is shown.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67115)

### 22. After the user swipes your app away in the app switcher, which background wake-ups still reach it?

A force-quit tells iOS the user wants the app stopped: silent pushes, `BGTaskScheduler` tasks and background `URLSession` transfers stop until the user opens it again, while alert pushes, VoIP pushes and location events still get through.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67116)

### 23. What is app prewarming, and how does it distort your launch-time metrics?

Since iOS 15 the system may start your process ahead of a predicted launch, run `dyld` and static initialisers, and then pause before `UIApplicationMain`. A launch timer anchored at process start then counts minutes the user never waited.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67114)

### 24. What is `BGContinuedProcessingTask` in iOS 26, and when would you use it instead of `BGProcessingTask`?

It lets work the user explicitly started (an export, an upload, a large edit) keep running after they leave the app, with a system progress UI they can watch and cancel. Unlike `BGProcessingTask`, it starts now, from the foreground, rather than whenever the system chooses.

[Full answer →](https://irbisa.com/topic/ios-lifecycle-background/#q-67121)
