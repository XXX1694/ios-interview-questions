# Xcode Build & Modularization interview questions

Targets, schemes, xcconfig, SPM, static vs dynamic linking, XCFramework, build times, thinning. Senior · 22 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/ios-build-modularization/)** · [All iOS topics](../README.md#all-topics)

### 1. What is the difference between a target, a scheme and a build configuration, and between `.xcodeproj` and `.xcworkspace`?

A target describes one product to build (an app, a framework, a test bundle), a build configuration is a named set of build settings such as `Debug` or `Release`, and a scheme says which targets each action (Run, Test, Profile, Archive) builds and with which configuration.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67301)

### 2. How do you choose an app's minimum deployment target, and how do you use newer APIs without raising it?

Pick the oldest iOS version that still carries a meaningful share of your users (usually the current major release minus one or two), and reach newer APIs behind `if #available` checks and `@available` annotations.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67322)

### 3. What are a signing certificate, a provisioning profile and entitlements, and how do they fit together?

The certificate proves who signed the app, entitlements list the capabilities the app claims (push, App Groups, Keychain sharing), and the provisioning profile is Apple's signed permission that ties an App ID, certificates, entitlements and (for development and Ad Hoc) specific devices together.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67317)

### 4. What is the difference between CocoaPods, Carthage and Swift Package Manager, and why have most teams moved to SPM?

CocoaPods generates a workspace and builds pods from a central spec repo, Carthage only builds frameworks that you link yourself, and SPM is Apple's own dependency manager built into Swift and Xcode: no extra tool and no generated workspace.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67321)

### 5. What does this function do in a Debug build and in a Release build, and why do the two differ?

```swift
func loadItem(id: Int?) -> String {
  assert(id != nil, "id is required")
  #if DEBUG
  print("loading \(String(describing: id))")
  #endif
  precondition((id ?? 0) >= 0, "negative id")
  return "item-\(id ?? -1)"
}

print(loadItem(id: nil))
```

In Debug the `assert` fails and the app traps with "Assertion failed: id is required"; in Release `assert` is compiled out and the `#if DEBUG` block does not exist, so the call silently prints `item--1`.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67309)

### 6. What are `.xcconfig` files for, and in what order does Xcode resolve a build setting?

An `.xcconfig` is a plain-text file of build settings assigned to a configuration, so settings live in reviewable, mergeable lines instead of inside `project.pbxproj`, which Xcode rewrites as a machine-generated blob and two branches will conflict on.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67302)

### 7. What happens when you press Build in Xcode, from the task graph to the signed app?

Xcode's build system turns the targets into a dependency graph of tasks, then for each target compiles the sources, processes resources, links one binary, assembles the bundle, runs script phases and code-signs the result, in parallel wherever the graph allows.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67323)

### 8. In `Package.swift`, what is the difference between a target, a product and a package dependency, and why can another package not import your target?

A target is one module, a product is what the package exports to other packages, a package dependency is another package you consume; your target is invisible outside unless a library product lists it, however much of it is `public`.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67303)

### 9. How does Swift Package Manager resolve dependency versions, and why should `Package.resolved` be committed?

SwiftPM collects the version requirements of every package in the graph and picks one version per package (the newest that satisfies all of them), then records the exact versions and revisions in `Package.resolved`. Committing that file is what makes every machine build the same graph.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67304)

### 10. What is an XCFramework, and why did it replace fat frameworks built with `lipo`?

An `.xcframework` is a bundle that holds one build of a library per platform and variant (device, simulator, Mac Catalyst) side by side, with an `Info.plist` Xcode uses to pick the right one. A fat framework merged everything into one binary with `lipo`, which broke once simulators became `arm64`.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67307)

### 11. SwiftLint and a code generator run on every build even when nothing changed: what is the build system missing, and how do you fix it?

A Run Script phase with no declared input and output files cannot be proven up to date, so the build system runs it every time, and if it writes a file the compiler reads, everything downstream recompiles too.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67315)

### 12. What is app thinning, and why do the universal `.ipa` size, the per-device download size and the installed size differ?

App thinning means the App Store builds a variant per device from your upload: slicing keeps one architecture, one asset scale and matching GPU resources, so the user downloads a compressed variant far smaller than the universal `.ipa` and installs a larger uncompressed one.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67314)

### 13. What is a privacy manifest (`PrivacyInfo.xcprivacy`), and what happens to a build without one?

`PrivacyInfo.xcprivacy` is a plist bundled with the app or an SDK that declares what data it collects, whether it tracks, which domains it tracks through and why it calls "required reason" APIs. App Store Connect flags a build that uses those APIs without a declared reason, and you cannot ship it until the reason is declared.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67320)

### 14. How does a build get from Xcode to the App Store, and what happens in TestFlight and App Review along the way?

You archive a Release build, sign it for distribution and upload it to App Store Connect, where it is processed and can go to TestFlight testers; then you attach it to an app version, submit it for App Review and release it manually, automatically or as a phased rollout.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67319)

### 15. How do you set up CI/CD for an iOS app, and how do you handle code signing on a CI machine?

CI runs the same `xcodebuild` (usually through fastlane or Xcode Cloud) on a macOS runner, and the hard part is signing: the runner needs a distribution certificate's private key and a matching profile without anyone's Mac or personal Apple ID being involved.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67318)

### 16. How do static and dynamic linking differ on iOS, and why did cold launch get slower after twelve modules became dynamic frameworks?

A static library is copied into the app binary at link time and costs nothing extra at launch, while every embedded dynamic framework is a separate Mach-O image that dyld must find, map, fix up and initialise before `main`: twelve frameworks, twelve times that work.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67305)

### 17. `Feed` and `Profile` are dynamic frameworks that both depend on `Analytics` in this manifest: what goes wrong at runtime, and how do you fix it?

```swift
let package = Package(
  name: "App",
  products: [
    .library(name: "Analytics", type: .static, targets: ["Analytics"]),
    .library(name: "Feed", type: .dynamic, targets: ["Feed"]),
    .library(name: "Profile", type: .dynamic, targets: ["Profile"]),
  ],
  targets: [
    .target(name: "Analytics"),  // holds AnalyticsTracker.shared
    .target(name: "Feed", dependencies: ["Analytics"]),
    .target(name: "Profile", dependencies: ["Analytics"]),
  ]
)
```

`Analytics` is declared `type: .static`, so its code, including `AnalyticsTracker.shared`, is copied into both `Feed` and `Profile`. At runtime there are two independent singletons, and for Objective-C classes the console warns "Class X is implemented in both … One of the two will be used. Which one is undefined."

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67306)

### 18. A one-line change in a feature module costs twelve minutes to build: how do you find what is actually rebuilt and fix it?

Prove what is rebuilding before blaming the compiler: a one-line edit that costs minutes is far more often a dependency-graph problem (something forcing many files or modules to rebuild) than one slow file.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67308)

### 19. Every type in a feature package ended up `public` just to make the app compile: what does that cost, and what should the module expose?

`public` is a promise, not a convenience: each public symbol is API you have to keep working, and a wide surface lets any file in the app reach into the module's internals, which is the coupling the module was supposed to prevent.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67310)

### 20. Checkout must open Profile's address editor and Profile must open Checkout's payment sheet, and SwiftPM rejects the cycle: how do you break it?

Neither feature imports the other: move the contract they need into an interface module that both depend on, and let the app's composition root bind the implementations.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67311)

### 21. What are ABI stability and module stability, and what does turning on `BUILD_LIBRARY_FOR_DISTRIBUTION` commit an SDK to?

ABI stability lets a compiled binary run against the OS's Swift runtime; module stability lets a binary framework be imported by a newer compiler. `BUILD_LIBRARY_FOR_DISTRIBUTION` buys the second through library evolution, and turns every `public` declaration into a promise.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67312)

### 22. Why do large teams generate their Xcode project with Tuist or XcodeGen, and how would you migrate a 200-target project without freezing feature work?

Generators replace the conflict-prone `project.pbxproj` with a short, reviewable manifest. Migrate in one mechanical, provable step (generate the same project, prove it builds an identical app, delete the `.xcodeproj` from git) and restructure targets only afterwards.

[Full answer →](https://irbisa.com/topic/ios-build-modularization/#q-67316)
