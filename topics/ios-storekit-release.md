# StoreKit & App Store Release interview questions

StoreKit 2, subscriptions and offers, entitlements, server notifications, refunds, testing purchases, App Review, phased release, forced update. Middle · 19 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/ios-storekit-release/)** · [All iOS topics](../README.md#all-topics)

### 1. Which purchases must go through Apple's in-app purchase, and which can use Apple Pay or your own checkout?

Anything digital that is used inside the app (premium features, a content subscription, coins, ad removal) must be sold through in-app purchase. Physical goods and real-world services (a taxi ride, a hoodie, a hotel night) must not use it: they go through Apple Pay or your own card form.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74101)

### 2. What are the four in-app purchase types, and what does each one oblige the app to support?

Consumables (coins, credits) are bought again and again, non-consumables (a lifetime unlock) once and forever, auto-renewable subscriptions renew until cancelled, and non-renewing subscriptions (a season pass) simply end. The type decides who keeps the record and what must be restorable.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74102)

### 3. Walk through a purchase with StoreKit 2: which calls happen between loading the product and unlocking the content?

Load the products with `Product.products(for:)`, call `purchase()` on the one the user taps, check that the result is `.success(.verified(transaction))`, deliver the content, then call `transaction.finish()`. Purchases that complete later arrive through `Transaction.updates`.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74103)

### 4. What is the difference between `CFBundleShortVersionString` and `CFBundleVersion`, and which one must change for every upload?

`CFBundleShortVersionString` is the version users see in the App Store (`2.4.0`); `CFBundleVersion` is the build number (`1873`). The build number must change for every upload; the version changes once per release and must be higher than the last approved one.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74104)

### 5. How do you ask for an App Store rating with `requestReview`, and why can the app not tell whether the dialog appeared?

Call the `requestReview` action from the SwiftUI environment (in UIKit, `AppStore.requestReview(in:)`) at a happy moment, and the system decides whether to show the dialog. It appears at most three times per app in 365 days and reports nothing back, so the app can neither know nor count on it.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74105)

### 6. Why must the app start listening to `Transaction.updates` at launch, and which purchases are lost without it?

Many transactions never pass through your `purchase()` call: renewals, Ask to Buy approvals, purchases on another device, refunds, offer codes redeemed in the App Store, and anything a crash left unfinished. They arrive only through `Transaction.updates`, so a listener that starts late misses them.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74106)

### 7. Find the bug: when the backend is down during a purchase, the user is charged and the coins never arrive, not even after the backend recovers.

```swift
func handle(_ result: VerificationResult<Transaction>) async {
  guard case .verified(let transaction) = result else { return }
  await transaction.finish()
  do {
    // The backend verifies the JWS and adds the coins to the account
    try await api.creditCoins(signedTransaction: result.jwsRepresentation)
    await wallet.reload()
  } catch {
    logger.error("credit failed: \(error)")
    showToast("Something went wrong. Please try again.")
  }
}
// Called for purchase() results and for every Transaction.updates element
```

`finish()` is called before the coins are credited. A finished transaction is never delivered again, so when `creditCoins` fails nothing hands the purchase back: the user has paid, has no coins, and no retry will ever happen.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74107)

### 8. How do you decide what the user owns with `Transaction.currentEntitlements`, and why is an `isPremium` flag in `UserDefaults` the wrong source of truth?

Recompute ownership from `Transaction.currentEntitlements` at launch and after every update: it holds the latest signed transaction for each active subscription and non-consumable. A `UserDefaults` flag is a copy that never learns about expiry, refunds, Family Sharing or another device.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74108)

### 9. What does Restore Purchases do in StoreKit 2, why does App Review still expect the button, and when should you call `AppStore.sync()`?

In StoreKit 2 restoring is mostly automatic: transactions sync to every device on the same Apple Account and show up in `currentEntitlements`. The button stays because App Review requires one, and the `AppStore.sync()` behind it is for the rare case where the device's copy is stale.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74109)

### 10. How do subscription groups and levels work, and what happens on an upgrade, a downgrade and a crossgrade?

A subscription group holds plans that replace each other; a user has at most one active plan per group. Plans are ranked by level: an upgrade starts at once with a prorated refund, a downgrade waits for the next renewal, and a crossgrade depends on the periods.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74110)

### 11. How do you show a free trial only to users eligible for it, and how do introductory offers, promotional offers and offer codes differ?

Show the trial only when `isEligibleForIntroOffer` says yes: one introductory offer per subscription group per Apple Account. Introductory offers target new subscribers, promotional offers current or lapsed ones (signed by your server), and offer codes are codes the user redeems.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74111)

### 12. A subscriber's card fails at renewal. What are billing retry and the billing grace period, and what should the app show in each state?

When a renewal fails, Apple retries the charge for up to 60 days (billing retry). With a billing grace period turned on, the user keeps access for its first days; after that, access stops until a retry succeeds. Both states need a "fix your payment" message.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74112)

### 13. A user gets a refund from Apple. How do the app and your server find out, and what does `beginRefundRequest` let you offer in the app?

Apple decides refunds, not you. When it grants one, the app gets the transaction again in `Transaction.updates` with `revocationDate` set, it drops out of `currentEntitlements`, and your server receives a `REFUND` notification. `beginRefundRequest` opens Apple's refund form without leaving the app.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74113)

### 14. How do you test in-app purchases with a StoreKit configuration file, the sandbox and TestFlight, and what does each one not catch?

Use three layers: a `.storekit` configuration file for fast local runs and unit tests, the sandbox for the real App Store Connect products and server notifications, and TestFlight for the release build in testers' hands. Each one fakes something, so none of them alone proves that purchasing works.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74114)

### 15. App Review rejects your build. How do you respond in App Store Connect, when do you appeal, and when do you ask for an expedited review?

Read the cited guideline and the reviewer's notes, then fix and resubmit, or reply in App Store Connect to explain or ask what they need. Appeal to the App Review Board only when the rule was misapplied, and ask for an expedited review only for a critical fix or a dated event.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74115)

### 16. The App Store has no forced update. How do you move users to a minimum supported version, and what must still work in the old build?

Ship the check before you need it: at launch the app fetches a minimum and a recommended version from your server, compares them with its own `CFBundleShortVersionString`, and shows a blocking screen or a banner that opens the App Store page. Only builds that already contain the check can be moved.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74116)

### 17. How does your backend learn about renewals, refunds and cancellations, and why trust Apple's signed JWS transaction rather than the app's word?

Apple pushes App Store Server Notifications V2 to an HTTPS endpoint you register, and the App Store Server API answers questions on demand. Every payload is a JWS signed by Apple that the server verifies itself, while an app saying "I bought Pro" is a request anyone can forge.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74117)

### 18. A user subscribes on iPhone, then signs in on Android and the web. Where does the entitlement live, and how does `appAccountToken` tie a purchase to your account?

On your server, keyed by your user ID: each store knows only its own purchases. The iOS app passes your account's UUID as `appAccountToken` when buying, Apple signs it into every transaction and notification, and the backend grants that account the entitlement everywhere.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74118)

### 19. A release crashes on launch for a third of users and the App Store has no rollback. What can you do in the first hour, and what should have existed before?

Stop the spread, then roll forward: pause the phased release, switch the crashing path off from the server if you can, and ship a fix (or the previous version's code under a higher version number) with an expedited review request. Users who already updated wait for that build.

[Full answer →](https://irbisa.com/topic/ios-storekit-release/#q-74119)
