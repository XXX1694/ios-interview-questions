# iOS Persistence interview questions

UserDefaults, Keychain, Core Data, SwiftData, FileManager. Middle · 25 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/ios-persistence/)** · [All iOS topics](../README.md#all-topics)

### 1. What persistence options does iOS give you, and how do you choose between them?

Pick the lightest store that fits the data: small preferences in `UserDefaults`, secrets in the Keychain, documents and blobs as files, and structured related data in SwiftData or Core Data.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-3401)

### 2. What is `UserDefaults` for, and what must never go in it?

`UserDefaults` is a small key-value store for user preferences, saved as a property list in the app's container and read into memory at launch.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63416)

### 3. This toggle should be on by default, yet the sound stays off on a fresh install. Why, and how do you fix it?

```swift
struct SettingsView: View {
  @AppStorage("soundEnabled") private var soundEnabled = true

  var body: some View {
    Toggle("Sound", isOn: $soundEnabled)
  }
}

final class SoundPlayer {
  func play(_ name: String) {
    guard UserDefaults.standard.bool(forKey: "soundEnabled") else { return }
    // ... plays the sound
  }
}
```

Nothing was ever written under that key: the `true` in `@AppStorage` is only a fallback inside the property wrapper, and `bool(forKey:)` answers a missing key with `false`. Register the default with `register(defaults:)` at launch.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63408)

### 4. Where do you store an auth token on iOS, and why not in `UserDefaults`?

In the Keychain: the system encrypts it with keys tied to the device and its passcode, while `UserDefaults` is a plain plist that can be read from an unencrypted backup or a jailbroken phone.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-3404)

### 5. Does Keychain data survive deleting the app, and what do accessibility classes and access groups control?

Yes: Keychain items live outside the app's container, so they survive deletion and a reinstall finds them. The accessibility class decides when an item can be decrypted, and the access group decides which of your apps and extensions can read it.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63421)

### 6. What are `Documents`, `Application Support`, `Caches` and `tmp` for, and which of them are backed up or can be purged?

Each app gets a sandbox container: `Documents` holds what the user created, `Library/Application Support` holds the app's own data, both backed up; `Library/Caches` is not backed up and can be purged; `tmp` is short-lived scratch space.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63404)

### 7. `NSCoding`/`NSKeyedArchiver` vs `Codable`: what is the difference, and which do you use today?

`Codable` is the Swift protocol pair the compiler synthesises for structs, enums and classes; `NSCoding` is the older Objective-C protocol for `NSObject` classes, archived with `NSKeyedArchiver`. New code uses `Codable`.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63417)

### 8. The app was killed mid-write and the settings file came back as garbage on the next launch. How do you make a file write crash-safe?

Never write over a live file: put the new bytes somewhere else and swap them in with a rename, which the filesystem performs atomically.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63412)

### 9. After a restore from a backup on a new phone, every saved photo is missing, although the files are in `Documents` and the paths are in the database. What broke?

The container directory carries a UUID that can change (restore from a backup, device migration, reinstall), so an absolute path saved earlier points at a directory that no longer exists. Persist the file name, never the path.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63410)

### 10. What are the parts of the Core Data stack, and what does each one do?

Four pieces: `NSManagedObjectModel` is the schema, `NSPersistentStoreCoordinator` connects that model to the store files, `NSManagedObjectContext` is the scratchpad where you fetch and change objects, and `NSPersistentContainer` builds and owns all of them.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63418)

### 11. SwiftData or Core Data: how are they related, and which do you choose for a new app?

SwiftData is a Swift-native layer over the Core Data engine, with the schema written as `@Model` classes instead of an `.xcdatamodeld` file. Choose it for a new app on iOS 17+, and stay on Core Data for an existing store or for features SwiftData lacks.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-3403)

### 12. How does a list stay in sync with the store: `NSFetchedResultsController` in UIKit, `@FetchRequest` and `@Query` in SwiftUI?

Each one wraps a fetch request (entity, predicate, sort order) and watches its context for saved changes, so the list updates itself instead of you re-fetching after every write.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63419)

### 13. Find the bug: the customer search crashes for some users while they type a name.

```swift
func searchCustomers(matching text: String,
                     in context: NSManagedObjectContext) throws -> [Customer] {
  let request = Customer.fetchRequest()
  request.predicate = NSPredicate(format: "name CONTAINS[cd] '\(text)'")
  request.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
  request.fetchLimit = 50
  do {
    return try context.fetch(request)
  } catch {
    print("Search failed: \(error)")
    return []
  }
}
```

The search text is interpolated into the predicate's format string, so a name with an apostrophe such as `O'Brien` makes it unparseable and `NSPredicate(format:)` raises an exception that kills the app. Pass the text as a `%@` argument instead.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63422)

### 14. Deleting a playlist leaves orphaned tracks behind. How do delete rules and inverses work?

A relationship needs an inverse and an explicit delete rule, and the rule is what decides whether deleting the parent also deletes the children, merely unlinks them, or is refused.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63407)

### 15. This code crashes now and then with a Core Data threading error. What is wrong, and how do you pass an object to another context?

```swift
func markAsRead(_ article: Article) {
  // article was fetched from container.viewContext on the main thread
  let background = container.newBackgroundContext()
  background.perform {
    article.isRead = true
    article.readAt = .now
    try? background.save()
  }
}
```

`article` belongs to `viewContext` but is changed on the background context's queue, and it is not registered in `background` at all, so that `save()` writes nothing. Pass the `NSManagedObjectID` and load the object inside the other context.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63420)

### 16. Importing twenty thousand records freezes the UI. How do you write to the store off the main thread?

Do the writing in a context the UI does not read from, and move results across the boundary as identifiers rather than as objects.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63401)

### 17. Every background sync doubles the number of rows in the table. How do you make the import idempotent?

Make the server's identifier the model's identity with a uniqueness constraint, so importing a record that already exists updates the existing row instead of adding a second one.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63411)

### 18. How do you migrate a Core Data or SwiftData store when the schema changes between releases?

Let the framework run a lightweight migration for additive changes, write a custom stage only where existing data must be transformed, and test against real old stores, because a failed migration is a crash on launch.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-3405)

### 19. How do you unit test code that persists through SwiftData or Core Data?

Give each test its own store that exists only in memory, so every test starts from an empty database and nothing survives to the next one.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63405)

### 20. Your widget needs the user's data. How do you share a store between the app and its extensions?

An extension is a separate process with its own container, so everything shared has to live in an App Group directory both targets are entitled to open.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63402)

### 21. Logging out has to leave nothing of the user behind on the device. What does deleting their data actually involve?

Deleting rows is not deleting data: you destroy the store file together with its sidecars, then clear everything that lives outside it (Keychain, preferences, caches and the App Group container).

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63413)

### 22. When would you choose SQLite through GRDB, or Realm, over Core Data or SwiftData?

Choose GRDB or raw SQLite when you need real SQL (FTS5, CTEs, window functions), large write-heavy data or plain value types without faulting; Realm is mostly a legacy choice today.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-3406)

### 23. A list backed by @Query gets slower as the table grows. What do you check first?

Check that the filtering really happens in SQLite: a predicate the store can translate narrows rows in the database, while a closure over the fetched array loads every row into memory first.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63406)

### 24. Two screens edit the same record, both save without an error, and one edit quietly disappears. Where did it go?

Each context saves against the snapshot it fetched, and when the row in the store no longer matches that snapshot it is the merge policy, not your code, that decides which values survive.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63415)

### 25. The team wants iCloud sync on models you already shipped. What changes, and what will bite you?

Switch the store to a CloudKit-backed configuration, but first migrate the shipped schema to CloudKit's rules (every attribute optional or defaulted, every relationship optional with an inverse, no unique constraints), because the store will not load otherwise.

[Full answer →](https://irbisa.com/topic/ios-persistence/#q-63403)
