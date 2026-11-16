# iOS Architecture interview questions

MVC, MVVM, VIPER, Coordinators, TCA, Clean Swift. Middle · 24 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/ios-architecture/)** · [All iOS topics](../README.md#all-topics)

### 1. What is MVC in UIKit, and why does it so often turn into a Massive View Controller?

In Apple's MVC the controller mediates between model and view, but `UIViewController` also owns the view's lifecycle, so view code, state, networking and navigation all settle in one class that cannot be tested without a view.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-3202)

### 2. What is dependency injection, and how does it make a view model testable?

Dependency injection means an object receives its collaborators from outside instead of creating or looking them up itself. Pass them through `init` as protocols, and a test can hand in a stub where the app hands in the real client.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-3204)

### 3. What is MVVM, what problem does it solve, and how does it look in UIKit and in SwiftUI?

MVVM moves presentation logic and screen state out of the view into a view model that exposes ready-to-show values; the view only binds to them and forwards user events, so the logic can be unit-tested without any UI.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63203)

### 4. Find the bug: type `"iphone"` quickly and the list sometimes shows the results for `"iph"`.

```swift
@MainActor @Observable
final class SearchViewModel {
  private(set) var results: [Product] = []
  private let api: any ProductAPI & Sendable
  init(api: any ProductAPI & Sendable) { self.api = api }

  func queryChanged(_ query: String) {
    Task {
      let found = (try? await api.search(query)) ?? []
      results = found
    }
  }
}
```

Every keystroke starts its own `Task` and nothing cancels the old ones, so whichever response arrives last wins: a slow reply for `"iph"` lands after the fast one for `"iphone"` and overwrites `results` with stale data.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63223)

### 5. Delegate, closure, NotificationCenter, KVO, Combine or AsyncSequence: how do you choose how two objects talk?

Choose by the shape of the link: several messages to one known counterpart is a delegate, a single result is an `async` call, and many unknown listeners means a notification or an observable value.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63206)

### 6. Which classic design patterns do UIKit and Foundation use, and where do you see them?

UIKit and Foundation are a catalogue of classic patterns: delegate and target-action for callbacks, observer through NotificationCenter and KVO, singletons for app-wide services, composite for the view tree and chain of responsibility for the responder chain.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63218)

### 7. What are the SOLID principles, with an iOS example for each?

SOLID is five rules for keeping types small and replaceable: one reason to change, extend without editing, subtypes keep the contract, small protocols, depend on abstractions. On iOS they are mostly the argument against the massive view controller.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63216)

### 8. What is the Coordinator pattern, and when is it worth using?

A coordinator is an object that owns navigation for a flow: screens report events such as "login succeeded", and the coordinator decides what to show next, so view controllers never push or present each other.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-3203)

### 9. Every finished checkout leaves its `CheckoutCoordinator` alive in memory. What is wrong with this code?

```swift
@MainActor
final class AppCoordinator {
  private var children: [AnyObject] = []
  private let nav: UINavigationController
  init(nav: UINavigationController) { self.nav = nav }

  func showCheckout() {
    let checkout = CheckoutCoordinator(nav: nav)
    checkout.onFinish = {
      self.nav.popToRootViewController(animated: true)
    }
    children.append(checkout)
    checkout.start()
  }
}
```

The child is appended to `children` but never removed, so the parent keeps every finished checkout alive, together with its view controllers and draft. The closure also captures `self` strongly, adding a cycle through `children`.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63219)

### 10. A push notification must open a specific comment on a specific post. Where does that routing state live?

In one observable router object above the navigation container, holding a typed path array that the URL handler, the notification handler and the UI all mutate.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63201)

### 11. What is the Repository pattern, and why keep DTOs, domain models and view state as separate types?

A repository is a protocol per domain concept that hides where data comes from (network, database, cache) and returns domain types. DTOs stay behind it, so a renamed JSON field becomes a one-file change instead of a change in three views.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63210)

### 12. When is a DI container such as Swinject or Factory worth it over plain constructor injection?

Rarely: constructor injection from a composition root is compile-checked and enough for most apps. A runtime registry such as Swinject pays off only with dozens of services and scoped lifetimes, and its price is a missing registration that fails at launch, not at build time.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63211)

### 13. Find the bug: `testFreshInstallShowsOnboarding` passes alone on a clean simulator, but fails once the whole class has run, and keeps failing.

```swift
@MainActor final class OnboardingViewModel {
  private let defaults: UserDefaults
  init(defaults: UserDefaults = .standard) { self.defaults = defaults }
  var showsOnboarding: Bool { !defaults.bool(forKey: "onboardingDone") }
  func finish() { defaults.set(true, forKey: "onboardingDone") }
}
@MainActor final class OnboardingTests: XCTestCase {
  func testFreshInstallShowsOnboarding() {
    XCTAssertTrue(OnboardingViewModel().showsOnboarding)
  }
  func testFinishHidesOnboarding() {
    let model = OnboardingViewModel()
    model.finish()
    XCTAssertFalse(model.showsOnboarding)
  }
}
```

Both tests use the default `UserDefaults.standard`, which is persisted on disk. `testFinishHidesOnboarding` writes `onboardingDone = true`, and every later run of the other test (in this run or the next) reads that leftover value instead of a fresh install.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63224)

### 14. How do VIPER and Clean Swift (VIP) split a screen, and what does that ceremony buy you?

Both split a screen into single-purpose objects behind protocols (VIPER into View, Interactor, Presenter, Entity and Router, Clean Swift into a one-way ViewController → Interactor → Presenter loop), buying testable seams and a uniform shape for about five files per screen.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63212)

### 15. What are feature flags, and how do you use them to ship a risky change safely?

A feature flag is a runtime switch that decides whether code already in the binary is active, so you can merge unfinished work, roll a feature out to a share of users and turn it off remotely without waiting for App Review.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63220)

### 16. Compare MVC, MVP, MVVM, VIPER and TCA: how do you choose an architecture for a feature or a team?

They differ in where non-view logic lives and how strictly data flows: MVC keeps it in the controller, MVP and MVVM move it into a testable object, VIPER splits it further, TCA routes every change through a reducer. Pick by team, UI framework and what must be tested.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-3201)

### 17. What is unidirectional data flow, and what problems does TCA solve with it?

Unidirectional data flow means state changes travel one way: the view sends an action, a reducer produces the new state, the view renders it. TCA, Point-Free's Redux-style library, builds every feature this way from State, Action, Reducer, Effect and Store.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-3205)

### 18. What is Clean Architecture on iOS, and when does a use-case layer pay for itself?

Clean Architecture arranges code in rings (domain at the centre, then data and presentation) with every dependency pointing inward, so business rules know nothing about URLSession, SwiftData or SwiftUI and can be tested alone.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63202)

### 19. How do you structure a large iOS app, by feature or by layer, and where do the module boundaries go?

Split by feature first and by layer inside each feature: a local Swift package per feature on top of shared Core and DesignSystem modules, with a thin app target that only wires them together.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-3206)

### 20. A three-step checkout shares one draft order, and users report a new checkout opening with the previous order's address. Where should that draft live?

In an object owned by the flow itself, created when the flow starts and released when it ends. A stale address means the draft outlives the flow, which is exactly what a singleton or an app-root environment object guarantees.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63214)

### 21. You are adding SwiftUI to a ten-year-old UIKit app. Where do you draw the boundary?

Adopt SwiftUI one whole screen at a time behind a `UIHostingController`, so the seam falls on a navigation edge instead of inside a view.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63204)

### 22. The app is 300k lines of view controllers and feature work cannot stop. How do you reach a testable architecture?

Refactor along the seams you are already touching: new work is written in the target architecture and old screens change only when a ticket lands in them.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63205)

### 23. How would you design an image loading and caching pipeline for a scrolling feed?

Layer it: a memory cache of decoded, downsampled images, a disk cache of the original bytes, and a loader that coalesces duplicate requests, decodes off the main thread and cancels work for cells that scrolled away.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63221)

### 24. How would you design offline-first sync for a notes app, including conflict resolution?

Make the local database the source of truth: the UI reads and writes only locally, every change is also queued in an outbox, and a sync engine pushes the outbox and pulls server changes when it can, resolving conflicts by a rule chosen up front.

[Full answer →](https://irbisa.com/topic/ios-architecture/#q-63222)
