# Security, Privacy & Keychain interview questions

Data protection classes, ATS and TLS, certificate pinning, biometrics, jailbreak detection, secrets in the build, permissions, ATT and location. Middle · 20 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/ios-security-keychain/)** · [All iOS topics](../README.md#all-topics)

### 1. What is the Keychain on iOS, how is it protected, and what kinds of items can it hold?

The Keychain is an encrypted database of small secrets that the system keeps for you: apps reach it only through the `SecItem` API, and every item is encrypted with keys protected by the device passcode and the Secure Enclave.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71601)

### 2. Can you hide an API key in `Info.plist` or an `.xcconfig` file? Where should a secret the app relies on actually live?

No. Everything in the `.ipa` ships to every user: `Info.plist` is a readable file in the bundle and string constants sit in the binary, so the key is extracted in minutes. A key that grants real power stays on your server, and the app calls your backend instead.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71603)

### 3. How do you add Face ID / Touch ID to an app with `LocalAuthentication`?

Create an `LAContext`, call `canEvaluatePolicy` to check that biometrics are available and enrolled, then call `evaluatePolicy` with a reason string. Face ID also needs `NSFaceIDUsageDescription` in `Info.plist`.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71604)

### 4. How do runtime permissions work on iOS: what happens if `NSCameraUsageDescription` is missing, and what can the app do after the user taps Don't Allow?

The app declares a purpose string in `Info.plist`, and the system asks the user once, on the first request. Without `NSCameraUsageDescription` the app is killed at that first access; after Don't Allow the prompt never returns, and only the user can turn access back on in Settings.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71617)

### 5. How does HTTPS protect a request: what does the TLS handshake agree on, and what does the app check in the server's certificate?

TLS gives a request encryption, integrity and the server's identity. In the handshake the client and server agree on a version, a cipher and a shared key through ECDHE, and the server proves it holds the certificate's private key; the client checks that certificate before it sends the request.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71618)

### 6. What are the Data Protection classes for files, which one does a file get by default, and when would you change it?

Each file is encrypted with a key from a class that decides when it can be read: `complete` only while unlocked, `completeUnlessOpen` keeps already open files usable after lock, `completeUntilFirstUserAuthentication` (the default) after the first unlock since boot, `none` always.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71605)

### 7. Find the bug: users get signed out at random, usually when the app syncs in the background and the phone is locked.

```swift
func saveToken(_ token: Data) {
  SecItemAdd([kSecClass: kSecClassGenericPassword, kSecAttrService: "auth", kSecValueData: token,
              kSecAttrAccessible: kSecAttrAccessibleWhenUnlocked] as CFDictionary, nil)
}
func readToken() -> String? {
  var result: CFTypeRef?
  let query = [kSecClass: kSecClassGenericPassword, kSecAttrService: "auth", kSecReturnData: true] as CFDictionary
  guard SecItemCopyMatching(query, &result) == errSecSuccess, let data = result as? Data else { return nil }
  return String(decoding: data, as: UTF8.self)
}
// Runs from a silent push and a BGAppRefreshTask, usually with the phone locked:
func backgroundSync() async {
  guard let token = readToken() else { return session.logOut() }
  await api.sync(token: token)
}
```

The token is saved as `kSecAttrAccessibleWhenUnlocked`, so a background sync on a locked phone gets `errSecInteractionNotAllowed`. `readToken()` turns that error into `nil`, and the app signs out a user whose token is perfectly valid.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71606)

### 8. An `https` request fails with `-1200` and `-9802` in the log, yet the same URL loads in a browser. What does ATS demand that the server lacks, and how do you check?

ATS rejects the TLS handshake because the server is below Apple's minimum: TLS 1.2 or later, a forward-secret ECDHE cipher, and a certificate signed with SHA-256 on an RSA key of 2048+ bits or an ECC key of 256+ bits. Browsers are more lenient, so they connect.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71607)

### 9. Find the bug: pinning works in the test, but a proxy with a user-installed CA can still read the app's traffic.

```swift
final class PinningDelegate: NSObject, URLSessionDelegate {
  let pinnedKeyHash: Data   // SHA-256 of the server's public key
  init(pinnedKeyHash: Data) { self.pinnedKeyHash = pinnedKeyHash }

  func urlSession(_ session: URLSession, didReceive challenge: URLAuthenticationChallenge,
                  completionHandler: @escaping @Sendable (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
    guard let trust = challenge.protectionSpace.serverTrust,
          SecTrustEvaluateWithError(trust, nil),
          let leaf = (SecTrustCopyCertificateChain(trust) as? [SecCertificate])?.first,
          let key = SecCertificateCopyKey(leaf),
          let raw = SecKeyCopyExternalRepresentation(key, nil) as Data?,
          Data(SHA256.hash(data: raw)) == pinnedKeyHash
    else {
      completionHandler(.performDefaultHandling, nil)   // pin mismatch: use system trust
      return
    }
    completionHandler(.useCredential, URLCredential(trust: trust))
  }
}
```

On a pin mismatch the delegate falls back to `.performDefaultHandling`, so any certificate that a trusted CA signed is accepted and pinning protects nothing. The pin check only decides which branch runs.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71602)

### 10. How do you hash, encrypt and authenticate data with CryptoKit, and where does the encryption key live?

`SHA256.hash` gives a fingerprint, `AES.GCM.seal` (or `ChaChaPoly`) encrypts and detects tampering in one step, and `HMAC` proves who produced a message. The `SymmetricKey` lives in the Keychain, never in the bundle or next to the encrypted file.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71608)

### 11. How does privacy work in `Logger` and `os_log`, and what must never reach the device log?

The unified log redacts interpolated values you do not mark `.public`: strings and objects appear as `<private>`, while numbers and booleans are public by default. Tokens, passwords and one-time codes must not be logged at all, redacted or not.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71609)

### 12. How do you keep sensitive screens out of the app-switcher snapshot and out of screen recordings on iOS?

iOS has no flag that blocks screenshots, so you cover the content yourself: show a privacy overlay as soon as the scene becomes inactive, before the system takes the app-switcher snapshot, and hide sensitive views while `sceneCaptureState` is `.active`.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71610)

### 13. What is App Tracking Transparency, when may you show its prompt, and what happens to the IDFA and ad attribution when the user declines?

ATT is the permission an app needs before it tracks the user across other companies' apps and websites, which includes reading the IDFA. The prompt appears only while the app is active and only once; after a decline the IDFA is all zeros, and attribution moves to SKAdNetwork or AdAttributionKit.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71619)

### 14. What is the difference between When In Use and Always location access, and what does turning off Precise Location change for your code?

When In Use delivers locations while the app is in use; Always also lets the system relaunch the app for geofences, visits and significant changes. With Precise Location off, the location is approximate: a circle of about 5 km radius, recomputed about four times an hour.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71620)

### 15. This jailbreak check ships in a banking app. What does it catch, how is it bypassed, and what should the app do instead of `exit(0)`?

```swift
@MainActor func isJailbroken() -> Bool {
  let paths = ["/Applications/Cydia.app", "/bin/bash", "/usr/sbin/sshd", "/etc/apt"]
  if paths.contains(where: { FileManager.default.fileExists(atPath: $0) }) { return true }
  if UIApplication.shared.canOpenURL(URL(string: "cydia://")!) { return true }
  do {
    try "x".write(toFile: "/private/jb.txt", atomically: true, encoding: .utf8)
    return true   // the sandbox should have refused this write
  } catch {
    return false
  }
}

// application(_:didFinishLaunchingWithOptions:)
// if isJailbroken() { exit(0) }
```

It catches only an old rootful jailbreak with Cydia on an unmodified copy of the app. Modern rootless jailbreaks keep their files under `/var/jb`, and hiding tweaks hook `fileExists` and `canOpenURL` so every check returns `false`.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71611)

### 16. What is the Secure Enclave, and how do you create and use a key in it with CryptoKit?

The Secure Enclave is a separate coprocessor that generates and uses keys without ever revealing them to iOS. With CryptoKit you create a `SecureEnclave.P256` key, store its encrypted handle, and ask the enclave to sign or to perform key agreement.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71612)

### 17. What do `.userPresence`, `.biometryAny` and `.biometryCurrentSet` change for a Keychain item, and what happens to it when the user enrolls a new face?

They set what the system demands before it decrypts the item: `.userPresence` accepts biometrics or the passcode, `.biometryAny` any enrolled face or finger, `.biometryCurrentSet` only the set enrolled when the item was saved; after a new enrollment that item can never be read again.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71613)

### 18. You pin the server's public key. How do you rotate that key without locking out users on old app versions?

Ship the next key's pin before you need it: every release pins the current key and a pre-generated backup, so the server can switch to the backup and old builds keep connecting. The rotation is planned around the oldest app version still in use, not the certificate's expiry date.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71614)

### 19. What are DeviceCheck and App Attest, and what does each one let your server verify?

Both let your server ask Apple about the caller. DeviceCheck keeps two bits per device for your team that survive a reinstall; App Attest proves that a request was signed by a genuine instance of your app on a real Apple device.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71615)

### 20. How would you design the app lock for a banking app with a 6-digit PIN, Face ID and auto-lock after a minute in the background?

Keep the real credential behind device-bound Secure Enclave keys and make the server the judge: Face ID unlocks a key through `SecAccessControl`, the PIN is checked by the server with an attempt limit, and the session is a short-lived token kept only in memory.

[Full answer →](https://irbisa.com/topic/ios-security-keychain/#q-71616)
