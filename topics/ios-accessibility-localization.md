# Accessibility & Localization interview questions

VoiceOver, traits, Dynamic Type, Reduce Motion, String Catalogs, plurals, RTL, locale formatting. Middle · 25 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/ios-accessibility-localization/)** · [All iOS topics](../README.md#all-topics)

### 1. What does making an iOS app accessible involve: which users do you build for, and which system features do you support?

It means the app still works for someone who cannot see the screen, needs large text, cannot tell colours apart, feels sick from motion or cannot touch precisely. In practice: VoiceOver, Dynamic Type, contrast, Reduce Motion and alternative input.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67517)

### 2. What are an element's accessibility label, value, traits and hint, and in what order does VoiceOver read them?

VoiceOver reads label, value, traits, hint: the label says what the element is, the value its current state, the traits what kind of element it is and how it behaves, and the hint what happens when you activate it.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67501)

### 3. What does VoiceOver say as it swipes through this row, and what would you change?

```swift
struct UserRow: View {
  let user: User
  let onDelete: () -> Void

  var body: some View {
    HStack {
      Image("avatar_placeholder")
      Text(user.name)
      Spacer()
      Button(action: onDelete) {
        Image("ic_trash")
      }
    }
  }
}
```

Three stops, two of them noise: the placeholder's asset name as an image, the user's name, and the trash icon's asset name as a button. With nothing better to go on, SwiftUI uses an image's asset name as its label.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67518)

### 4. What is Dynamic Type, and how do you support it in SwiftUI and UIKit?

Dynamic Type is the system-wide text size setting; text set in a text style such as `.body` or `.headline` scales with it automatically, across seven standard sizes and five larger accessibility sizes.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67519)

### 5. How do you localize an iOS app, and what must never be hardcoded?

Add the languages to the project, keep every user-facing string in a String Catalog (`Localizable.xcstrings`) that Xcode fills from your code on each build, and let the user's locale format numbers, dates and prices.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67520)

### 6. A product card is five VoiceOver stops: image, title, price, badge, heart button. How do you make it one, and when should you not?

Wrap the card in `.accessibilityElement(children: .combine)` so it becomes one element whose label joins the children's, and expose the heart as a named custom action; in UIKit set `isAccessibilityElement = true` on the container and write its label.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67502)

### 7. VoiceOver reads a card's price before its title, although the title is drawn above it. How do you control the reading order?

In SwiftUI group the views with `.accessibilityElement(children: .contain)` and order them with `.accessibilitySortPriority` (higher is read first); in UIKit set the container's `accessibilityElements` array in the order you want.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67503)

### 8. A row has swipe-to-delete and a custom volume dial. How does a VoiceOver user reach them without those gestures?

Every gesture needs an accessibility equivalent: custom actions (`.accessibilityAction(named:)`, `accessibilityCustomActions`) appear on VoiceOver's Actions rotor, and a value control becomes adjustable so swiping up and down changes it.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67504)

### 9. After a save VoiceOver jumps to the top of the screen and re-reads it. Which notification was posted by mistake and which should be used?

It posted `.screenChanged`, which tells VoiceOver a new screen appeared and moves focus to its first element; a status message should be an `.announcement`, which is spoken without moving focus.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67505)

### 10. A form fails validation. How do you move VoiceOver focus to the error message in SwiftUI?

Attach `.accessibilityFocused(_:)` to the error view, bind it to an `@AccessibilityFocusState` property, and set that property once the error is on screen; VoiceOver moves its cursor there and reads it.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67521)

### 11. QA reports that this row ignores the user's text size, and that once it scales it clips at the largest sizes. What do you change?

```swift
HStack(spacing: 12) {
  Image(systemName: "envelope")
    .frame(width: 40, height: 40)
  Text(item.title)
    .font(.system(size: 17))
    .lineLimit(1)
}
.frame(height: 44)
```

Use a text style so the text scales, then remove every fixed dimension around it: `minHeight` instead of `height`, a `@ScaledMetric` icon size, no single-line limit, and a vertical layout at accessibility sizes.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67506)

### 12. Find the bug: the user raises the text size from Control Center, and this header's labels grow but stay side by side and clip until the screen is reopened.

```swift
final class ContactHeaderViewController: UIViewController {
  private let stack = UIStackView(arrangedSubviews: [UILabel(), UILabel()])
  override func viewDidLoad() {
    super.viewDidLoad()
    for case let label as UILabel in stack.arrangedSubviews {
      label.font = .preferredFont(forTextStyle: .body)
      label.adjustsFontForContentSizeCategory = true
    }
    view.addSubview(stack)
    stack.frame = view.bounds
  }
  override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    stack.axis = traitCollection.preferredContentSizeCategory.isAccessibilityCategory
      ? .vertical : .horizontal
  }
}
```

The axis is chosen only in `viewWillAppear`, which does not run when the text size changes while the screen is open: the labels follow Dynamic Type live, but the stack stays horizontal, so the larger text clips.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67524)

### 13. What are Reduce Motion, Reduce Transparency, Increase Contrast and Bold Text, and how do you honour them in your own UI?

They are system accessibility settings that UIKit and SwiftUI honour in their own controls; for your own animations, blurs, colours and fonts you read them, as SwiftUI environment values or `UIAccessibility` flags, and adapt.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67507)

### 14. Design marks paid orders with a green dot and overdue ones with a red dot. What is wrong with that, and how do you fix it?

Colour is the only carrier of the meaning, so the status is lost for colour-blind users and completely for VoiceOver; add a second signal (a shape or a word) and put the status into the accessibility label or value.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67509)

### 15. How do you test an app's accessibility: what do the Accessibility Inspector and `performAccessibilityAudit()` catch, and what do they miss?

Automated audits catch the mechanical problems on one screen (missing labels, small hit areas, low contrast, clipped text), while only a person using VoiceOver can tell whether the labels make sense and the task can be finished.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67508)

### 16. What is a String Catalog, and what do the New, Needs Review and Stale states mean after a build?

A String Catalog (`.xcstrings`) is a JSON file Xcode keeps in sync with your code: every build extracts localizable literals into it and tracks each translation's state, and it is compiled into ordinary `.strings` and `.stringsdict` files in the app bundle.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67510)

### 17. The string is `"\(count) items"` and the Russian build shows «1 товаров». How should plurals be handled?

With a plural variation in the String Catalog: the system picks a CLDR plural category for the number and each language supplies its own forms, so your code never decides between singular and plural.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67511)

### 18. In the German build the navigation title is translated but the section titles stay English: why, and which type fixes it?

```swift
struct LibrarySection: Identifiable {
  let id = UUID()
  let title: String
}

let sections = [LibrarySection(title: "Recent"),
                LibrarySection(title: "Favourites")]

List(sections) { section in
  Text(section.title)
}
.navigationTitle("Library")
```

`Text` localizes only literals: `Text(section.title)` with a `String` uses the verbatim initializer, and "Recent" stored as a `String` was never extracted. Store the title as `LocalizedStringResource`, which is extracted and resolved when it is rendered.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67512)

### 19. How do you format dates, numbers and prices for the user, and who decides the format when the language is English but the region is Germany?

Use the `.formatted()` APIs and let `Locale.current` decide. The region, not the language, sets the number, date, unit and calendar conventions, so English with a German region shows English words with German-style numbers (`1.234,50`) and a 24-hour clock.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67513)

### 20. Find the bug: in the French and German builds of the country picker, several countries sit at the very bottom of the list.

```swift
struct CountryPicker: View {
  let regionCodes: [String]            // "AE", "AT", "DE", "ZW", …
  @Binding var selection: String

  private var countries: [(code: String, name: String)] {
    regionCodes
      .map { ($0, Locale.current.localizedString(forRegionCode: $0) ?? $0) }
      .sorted { $0.name < $1.name }
  }

  var body: some View {
    Picker("Country", selection: $selection) {
      ForEach(countries, id: \.code) { Text(verbatim: $0.name).tag($0.code) }
    }
  }
}
```

`<` on `String` compares Unicode code points, not the user's alphabet, so names starting with an accented letter (Égypte, Österreich) sort after Z, and lowercase after uppercase. A list people read needs a locale-aware comparison such as `localizedStandardCompare(_:)`.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67525)

### 21. Product wants a language switcher inside the app: what does iOS already give you, and why is writing `AppleLanguages` to `UserDefaults` a hack?

iOS has had a per-app language setting since iOS 13: if the app ships more than one localization, its page in Settings shows a Language option, and an in-app "switcher" should simply open that page with `UIApplication.openSettingsURLString`.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67522)

### 22. How do you support right-to-left languages like Arabic: what mirrors automatically, what do you flip yourself, and what must never flip?

With leading and trailing layout, UIKit and SwiftUI mirror the interface automatically; you decide per image whether it is directional, keep things like media controls and logos unmirrored, and let the Unicode bidirectional algorithm order the text.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67514)

### 23. A bar chart is drawn in a single `Canvas` or a `draw(_:)` override. How do you make each bar reachable with VoiceOver?

A drawn chart is one opaque element, so you build its accessibility tree by hand: `.accessibilityChildren` in SwiftUI, or `UIAccessibilityElement`s with frames in UIKit, plus a summary, and ideally a chart descriptor for Audio Graphs.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67523)

### 24. Translations arrive two weeks before release. How do you catch truncation and hardcoded strings in CI long before that?

Run UI tests under Xcode's pseudolanguages and the largest text size: they stretch, accent and bracket every localized string, so clipping, glued sentences and never-extracted literals show up with no translator involved.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67515)

### 25. Beyond the app UI, where do you localize the App Store listing, push notifications and App Shortcuts?

Each lives outside your view code: the store listing in App Store Connect, push text as `loc-key` resolved on the device or in a Notification Service Extension, and App Shortcut phrases in their own `AppShortcuts.xcstrings`.

[Full answer →](https://irbisa.com/topic/ios-accessibility-localization/#q-67516)
