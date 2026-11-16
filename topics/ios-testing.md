# iOS Testing interview questions

XCTest, Swift Testing, XCUITest, test doubles, async tests, snapshots, coverage, flaky tests. Middle · 25 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/ios-testing/)** · [All iOS topics](../README.md#all-topics)

### 1. What is the difference between unit, integration and UI tests, and why write tests for an iOS app at all?

A unit test checks one type in isolation with its dependencies faked and runs in milliseconds; an integration test runs several real pieces together; a UI test drives the built app through its screens the way a user does.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66917)

### 2. What does a good unit test look like, and what is Arrange-Act-Assert?

A good unit test checks one behaviour, is fast and deterministic, and reads in three blocks: Arrange the inputs and fakes, Act by calling the one method under test, Assert on the observable result.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66918)

### 3. What does `@testable import` do, and what does it require of the build?

`@testable import MyApp` lets the test target use the module's `internal` declarations as if they were `public`, so you can test without widening access in production code. It only works if the module was compiled with testing enabled.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66919)

### 4. What do `setUp` and `tearDown` do in XCTest, and how many instances of the test-case class does XCTest create?

XCTest creates a separate instance of your `XCTestCase` subclass for every test method and calls `setUp` before and `tearDown` after each one, so instance properties always start fresh.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66901)

### 5. This test is green even though `FailingAPI` always fails. Why do the assertions never fail?

```swift
final class ProfileViewModelTests: XCTestCase {
  func testLoadReportsOffline() {
    let sut = ProfileViewModel(api: FailingAPI(error: .offline))

    sut.load { result in
      XCTAssertEqual(result, .failure(.offline))
      XCTAssertEqual(sut.state, .error)
    }
  }
}

// ProfileViewModel: the completion fires after an await
func load(completion: @escaping (Result<Profile, APIError>) -> Void) {
  Task { completion(await api.fetch()) }
}
```

`load` calls its completion later, asynchronously, so the test method returns before the closure runs; XCTest considers the test finished and passed at that moment, and the assertions inside the closure are never checked for this test.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66920)

### 6. What is the difference between `#expect` and `#require` in Swift Testing, and what do they report that `XCTAssertTrue` cannot?

`#expect` records a failure and lets the test continue; `try #require` throws and stops the test, which is right for a precondition such as unwrapping an optional. Both are macros, so a failure shows the value of every sub-expression.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66902)

### 7. How does an XCUITest UI test interact with your app, and how is it different from a unit test?

A UI test runs in its own runner process and drives the installed app from outside through the accessibility layer, like a user; it cannot call your code or read its state. It only finds elements, interacts with them and checks what is on screen.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66921)

### 8. How does Swift Testing differ from XCTest, and what has to stay in XCTest when you migrate?

Swift Testing replaces `XCTestCase` subclasses and the `XCTAssert` family with `@Test` functions, the `#expect` and `#require` macros and traits, and runs tests in parallel inside one process by default. UI tests and `measure` performance tests have no equivalent and stay in XCTest.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66903)

### 9. How many test cases does each of these parameterised tests run, and what goes wrong if the two lists passed to `zip` differ in length?

```swift
enum Plan: CaseIterable { case free, pro, team }

@Test(arguments: Plan.allCases, [1, 5, 50])
func priceIsNeverNegative(plan: Plan, seats: Int) {
  #expect(Pricing.price(plan, seats: seats) >= 0)
}

@Test(arguments: zip(Plan.allCases, [0, 9, 29]))
func basePrice(plan: Plan, expected: Int) {
  #expect(Pricing.price(plan, seats: 1) == expected)
}
```

The first runs 9 cases, because two argument collections produce every combination (3 × 3); the second runs 3, because `zip` pairs elements up. Each case is reported separately, and a shorter list in `zip` silently drops the extra cases.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66922)

### 10. What are a stub, a spy, a mock and a fake, and how do you build them in Swift without a mocking library?

All four stand in for a real dependency: a stub returns canned answers, a spy also records its calls, a mock verifies expectations set in advance, and a fake is a working lightweight implementation. Swift has nothing that creates them at run time, so you write or generate them.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66905)

### 11. Find the bug: this test fails even though `CheckoutViewModel.confirm()` does call `track("purchase")`.

```swift
protocol Analytics { mutating func track(_ event: String) }
struct AnalyticsSpy: Analytics {
  private(set) var events: [String] = []
  mutating func track(_ event: String) { events.append(event) }
}
final class CheckoutViewModel {
  private var analytics: any Analytics
  init(analytics: any Analytics) { self.analytics = analytics }
  func confirm() { analytics.track("purchase") }
}
@Test func confirmTracksPurchase() {
  let spy = AnalyticsSpy()
  let sut = CheckoutViewModel(analytics: spy)
  sut.confirm()
  #expect(spy.events == ["purchase"])
}
```

`AnalyticsSpy` is a struct, so handing it to `CheckoutViewModel` copies it: `confirm()` records `"purchase"` in the view model's copy, while the test's `spy` keeps an empty `events` array. The test fails although the production code is correct.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66927)

### 12. Why does `wait(for:timeout:)` hang in an `async` XCTest, and when do you still need an expectation with async/await?

`wait(for:timeout:)` blocks the thread it runs on, and in an `async` test that thread belongs to the cooperative pool the awaited work needs, so the test can deadlock; in an `async` test use `await fulfillment(of:timeout:)`, which suspends instead.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66904)

### 13. A test calls `load()` on a `@MainActor` view model and asserts right away, but the state is still `.idle`. Why, and how do you fix the test?

`load()` almost certainly starts a `Task` and returns immediately, so the assertion runs before the task body has executed even once. Make the work awaitable and `await` it, instead of sleeping or yielding.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66906)

### 14. How do you prove an analytics event was never sent, without every run paying a five-second inverted-expectation wait?

Usually you do not wait at all: inject a spy for the analytics sink, drive the code to a point you can `await`, and assert the spy recorded nothing, which is exact and instant.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66907)

### 15. What does the teardown block in this test check, and why does it fail for this view model?

```swift
final class FeedViewModel {
  private let service: FeedService
  private(set) var items: [Post] = []

  init(service: FeedService) {
    self.service = service
    service.onUpdate = { posts in self.items = posts }
  }
}

func testFeedViewModelIsReleased() {
  let sut = FeedViewModel(service: FeedService())
  addTeardownBlock { [weak sut] in
    XCTAssertNil(sut, "FeedViewModel leaked")
  }
}
```

It checks for a memory leak: once the test method has returned, nothing but the weak capture should point at `sut`, so it must be `nil`. It is not, because the view model owns `service` and `service.onUpdate` captures `self` strongly, which makes a retain cycle.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66924)

### 16. How do you test a SwiftUI view, given that you cannot inspect what `body` rendered in a unit test?

Move the decisions out of the view into a model and unit-test that; then check pixels with snapshot tests and behaviour with XCUITest. Apple ships no API that hands a unit test the rendered SwiftUI view tree.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66915)

### 17. What are snapshot tests, and why do they so often fail on CI with a barely visible diff?

A snapshot test renders a view and compares it with a reference image or text dump committed to the repo, catching visual regressions no assertion would. It fails on CI because the rendering depends on the simulator, OS and machine that produced the reference.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66908)

### 18. Your XCUITest suite is green in English and red the day the German build lands. What were the queries matching on, and what should they match?

They were matching the label, which is the translated, user-visible text. Queries should match an `accessibilityIdentifier`, which is never localised and never spoken by VoiceOver.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66909)

### 19. A UI test needs a signed-in user with three items in the cart. How do you get the app into that state without tapping through sign-in?

Pass a scenario in at launch and let the app build itself from stubs: the UI test runs in a different process, so `launchArguments` and `launchEnvironment` are the main channel into the app.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66910)

### 20. What does Xcode's code coverage number actually measure, and why can a view model at 87% coverage still ship a bug?

It measures which code regions ran at least once during the tests, and nothing about whether anything was asserted, or which combinations of inputs ran. 87% means 13% is certainly untested, not that the rest is tested.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66913)

### 21. What is TDD, and where does it pay off in iOS development and where does it not?

Test-driven development means writing a failing test first, then the minimum code that makes it pass, then refactoring with the tests green: red, green, refactor. It pays off most on logic with clear inputs and outputs, and least on layout and exploratory UI.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66925)

### 22. How do you set up the test jobs of an iOS pipeline: what runs on every pull request and what on merge or nightly?

Run the fast, deterministic suites on every pull request (unit and integration tests under a Smoke test plan) and move the slow ones (full UI, snapshots, performance) to merge or nightly.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66926)

### 23. Turning on parallel testing took the suite from 20 minutes to 6 and made four tests fail at random. What did parallelism change?

Your tests stopped having the machine to themselves: XCTest distributes test classes across cloned processes and simulators, and Swift Testing also runs tests concurrently inside one process. Tests that relied on shared state or a fixed order now fail by timing.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66912)

### 24. One XCUITest fails about once in eight CI runs and never on your laptop. How do you tell a real race from a missing wait?

Make it fail on demand first, then change one thing: if a longer wait fixes it, it was synchronisation; if it still fails with a 30-second timeout, the app reached the wrong state and you have a real race.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66914)

### 25. A 900-line view controller calls `AnalyticsManager.shared`, `URLSession.shared`, `UserDefaults.standard` and `Date()`. How do you get it under test this week?

Cut a seam at each of those four calls and change nothing else: the goal is an object you can construct in a test without globals, not a rewritten screen.

[Full answer →](https://irbisa.com/topic/ios-testing/#q-66916)
