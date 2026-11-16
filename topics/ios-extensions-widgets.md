# App Extensions & WidgetKit interview questions

WidgetKit timelines, Live Activities, App Intents, share and notification extensions, App Groups. Middle · 20 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/ios-extensions-widgets/)** · [All iOS topics](../README.md#all-topics)

### 1. What is an app extension, which kinds are common, and how does it differ from the containing app in process, lifecycle and memory?

An app extension is a separate target with its own bundle id and its own process, launched by the system when a host such as the Home Screen or the share sheet needs it. It shares your source code but none of the app's running state.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67401)

### 2. What are the building blocks of a WidgetKit widget, and what actually runs where?

A widget is a `Widget` with a configuration, a timeline provider that produces dated entries, and a SwiftUI view that renders each entry. WidgetKit renders those views ahead of time and displays the stored result, so nothing in the view runs while it is visible.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67417)

### 3. What do `placeholder`, `snapshot` and `timeline` do in a widget's timeline provider, and when does WidgetKit call each?

`placeholder` returns an instant, redacted skeleton; `snapshot` returns one entry for the widget gallery and other transient views; `timeline` returns the dated entries the widget will actually show, plus when to ask again.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67402)

### 4. How do you deep-link from a widget into a specific screen, and what is the difference between `widgetURL` and `Link`?

`.widgetURL(_:)` makes the whole widget one tap target with one URL; `Link` gives separate destinations to parts of the widget, but only in the medium, large and extra-large families; small widgets and Lock Screen accessories get just `widgetURL`.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67407)

### 5. What is a Live Activity, and how does it differ from a widget and from a push notification?

A Live Activity is a card for one ongoing event (a delivery, a ride, a match) that the app starts and then updates in place on the Lock Screen and in the Dynamic Island. Unlike a widget it is tied to one event with instant, explicit updates; unlike a push it is one card that changes instead of a pile of messages.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67420)

### 6. This widget view should count down every second. Why does this countdown never tick on the Home Screen, and how do you fix it?

```swift
struct CountdownView: View {
  let entry: CountdownEntry
  @State private var secondsLeft = 0

  var body: some View {
    Text("\(secondsLeft) s left")
      .onAppear {
        secondsLeft = Int(entry.deadline.timeIntervalSinceNow)
      }
      .onReceive(Timer.publish(every: 1, on: .main, in: .common).autoconnect()) { _ in
        secondsLeft = Int(entry.deadline.timeIntervalSinceNow)
      }
  }
}
```

It never ticks: WidgetKit renders the body once and displays the archived result, so neither `onAppear` nor the timer can change `@State` on screen, and the number never changes. A countdown in a widget must be a view the system animates itself, such as `Text(timerInterval:countsDown:)`.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67418)

### 7. How do the reload policies `.atEnd`, `.after(_:)` and `.never` work, and how does the app refresh a widget right after its data changes?

A reload policy tells WidgetKit the earliest time it may ask the provider for a new timeline: a request, not a schedule. To refresh at once, the app writes the new data where the widget reads it and calls `WidgetCenter.shared.reloadTimelines(ofKind:)`.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67403)

### 8. Find the bug: the steps widget always shows 0, although the app shows the right count and reloads the widget after every save.

```swift
// Compiled into both the app and the widget extension.
enum StepsStore {
  private static var defaults: UserDefaults { .standard }
  static func save(_ steps: Int) { defaults.set(steps, forKey: "todaySteps") }
  static func todaySteps() -> Int { defaults.integer(forKey: "todaySteps") }
}

// App: called when HealthKit delivers new samples.
func didUpdateSteps(_ steps: Int) {
  StepsStore.save(steps)
  WidgetCenter.shared.reloadTimelines(ofKind: "StepsWidget")
}

func makeEntry() -> StepsEntry {   // widget extension, in getTimeline
  StepsEntry(date: .now, steps: StepsStore.todaySteps())
}
```

`UserDefaults.standard` is private to each process's sandbox, so the app saves into its own container and the widget extension reads its own, empty one; `integer(forKey:)` returns 0 for the missing key. Both targets must use the App Group suite.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67421)

### 9. This provider should show each of the next four hourly forecasts as its hour arrives. What happens instead, and what is the fix?

```swift
func timeline(for configuration: CityIntent,
              in context: Context) async -> Timeline<WeatherEntry> {
  let forecast = await WeatherCache.hourly(for: configuration.city)

  let entries = forecast.prefix(4).map { hour in
    WeatherEntry(date: .now, temperature: hour.temperature)
  }
  return Timeline(entries: entries, policy: .atEnd)
}
```

Every entry is dated `.now`, so all four become due at the same instant and the widget never steps through the hours. And because the last entry's date has already passed, `.atEnd` asks for a new timeline right away, again and again.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67419)

### 10. How often can a widget refresh, and what counts against its refresh budget?

Each placed widget gets a rolling budget, typically 40–70 reloads a day (roughly one every 15 to 60 minutes), sized by how often the user actually looks at it. Once it is spent, the system silently stops honouring your reload policy until the budget refills.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67404)

### 11. How do you build a user-configurable widget with App Intents, and where does the list of choices come from?

Use `AppIntentConfiguration` with a `WidgetConfigurationIntent` whose `@Parameter` is an `AppEntity`; the entity's `EntityQuery` supplies the choices, and it runs inside the widget extension while the user edits the widget.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67405)

### 12. How do interactive widgets work since iOS 17, and why does an `onTapGesture` in a widget do nothing?

Only `Button(intent:)` and `Toggle(isOn:intent:)` are interactive: a tap runs an `AppIntent` in the background, the intent updates shared data, and WidgetKit then reloads the widget. Gestures and button closures never run: the view on screen is an archived snapshot.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67406)

### 13. What are widget rendering modes, and why do a widget's colours disappear on the Lock Screen or a tinted Home Screen?

A widget renders in `.fullColor`, `.accented` or `.vibrant` mode, and the last two drop your colours, so anything that carries meaning in hue alone disappears. Read the mode from the environment, pair colour with a glyph or a word, and declare the background with `containerBackground`.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67411)

### 14. In ActivityKit, what belongs in `ActivityAttributes` versus its `ContentState`, and how do you start, update and end a Live Activity?

`ActivityAttributes` holds what stays fixed for the activity's whole life, such as the order id and restaurant; its nested `ContentState` holds what changes, such as the step and ETA. You start with `Activity.request`, replace the state with `update`, and finish with `end`.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67408)

### 15. What presentations does a Live Activity have in the Dynamic Island, and when does the system use each one?

Compact (a leading and a trailing view) when yours is the only running activity, minimal when two or more apps have one, expanded when the user long-presses, plus the Lock Screen view, used on devices without an Island and for update banners.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67410)

### 16. What is an App Intent, and what do you add so an action works from Siri and Spotlight without the user setting up a shortcut?

An `AppIntent` is a Swift type describing one action (title, `@Parameter`s and `perform()`) that the system can run without your UI. On its own it only appears in the Shortcuts app; listing it in an `AppShortcutsProvider` with spoken phrases makes it available in Siri and Spotlight from install.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67413)

### 17. Find the bug: text-only pushes handled by this notification service extension arrive about 30 seconds late; pushes with an image arrive at once.

```swift
final class NotificationService: UNNotificationServiceExtension {
  var handler: ((UNNotificationContent) -> Void)?
  var best: UNMutableNotificationContent?
  override func didReceive(_ request: UNNotificationRequest,
                           withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
    handler = contentHandler
    best = request.content.mutableCopy() as? UNMutableNotificationContent
    guard let best, let link = best.userInfo["image"] as? String,
          let url = URL(string: link) else { return }
    Task {
      if let attachment = try? await ImageAttachment.download(url) { best.attachments = [attachment] }
      contentHandler(best)
    }
  }
  override func serviceExtensionTimeWillExpire() { if let handler, let best { handler(best) } }
}
```

When the payload has no `image` URL, the `guard` returns without calling `contentHandler`, so the system waits out the extension's time limit (about 30 seconds), calls `serviceExtensionTimeWillExpire()`, and only then shows the push.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67422)

### 18. How does a share extension receive a shared file, and how do you upload a large one reliably?

Read the `NSItemProvider` attachments from `extensionContext.inputItems`, copy the file into the App Group container, and hand the upload to a background `URLSession`. The extension process ends as soon as you call `completeRequest`, taking any in-process upload with it.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67412)

### 19. How do you start and update a Live Activity from your server with push notifications, and which tokens does the backend need?

The backend needs a push-to-start token, which belongs to the activity type and exists before any activity runs, and an update token for each running activity. Both arrive as async sequences and can change, so the app must upload every new value, not just the first.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67409)

### 20. A widget renders blank on a device while the app works fine, and your `print` output never appears. How do you debug the widget extension?

Debug it as the separate process it is: run the widget extension's scheme or attach Xcode to the extension by name, log with `Logger` instead of `print`, and read the extension's own crash reports; a blank widget is most often an extension that was killed.

[Full answer →](https://irbisa.com/topic/ios-extensions-widgets/#q-67415)
