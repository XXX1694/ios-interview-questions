# iOS Networking interview questions

URLSession, Codable, async APIs, interceptors, error handling. Middle · 25 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/ios-networking/)** · [All iOS topics](../README.md#all-topics)

### 1. What is REST, and how do `GET`, `POST`, `PUT`, `PATCH` and `DELETE` and the HTTP status codes differ?

REST is a convention for HTTP APIs: a URL names a resource, the HTTP method says what to do with it, and the status code tells the client how it went; its first digit is the class.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63315)

### 2. What happens when an app makes an HTTP request, and how do you send one with `URLSession`?

The system resolves the host through DNS, opens a TCP or QUIC connection, negotiates TLS, sends the method, headers and body, then reads back a status, headers and body; in Swift all of that is one `try await session.data(for:)` call.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-3301)

### 3. What is the difference between `URL` and `URLRequest`?

`URL` is only an address (scheme, host, path and query), while `URLRequest` wraps a URL with everything needed to send it: HTTP method, headers, body, timeout and cache policy.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63316)

### 4. What is the difference between a data task, a download task and an upload task in `URLSession`?

A data task returns the response body in memory, a download task writes it to a temporary file and hands you the file URL, and an upload task sends a body from memory or straight from a file.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63317)

### 5. What is App Transport Security, and how do you allow a domain that only speaks plain HTTP?

App Transport Security is the system policy that makes `URLSession` and `WKWebView` refuse insecure connections by default: plain HTTP, TLS older than 1.2, and certificates or ciphers below Apple's minimum.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63318)

### 6. This search returns nothing for `"C++ & Swift"` while the same text works in a browser. What is wrong with how the URL is built?

```swift
func search(_ term: String, page: Int) async throws -> Data {
  let url = URL(string: "https://api.example.com/search?q=\(term)&page=\(page)")!
  let (data, _) = try await URLSession.shared.data(from: url)
  return data
}

// search("C++ & Swift", page: 1)
```

The term is interpolated into the URL string, so its `&` starts a new parameter and the server receives `q=C++ `; query values must go through `URLComponents` and `URLQueryItem`, which encode each value.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63307)

### 7. What are the `URLSessionConfiguration` types (`default`, `ephemeral` and `background`), and when do you use each?

`default` stores cache, cookies and credentials on disk; `ephemeral` keeps them only in memory and loses them with the session; `background` hands transfers to a system process so they continue while the app is suspended or terminated.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63319)

### 8. This loader is called for every image in a feed. What is wrong with it, and what does it cost?

```swift
final class ImageLoader: NSObject, URLSessionDataDelegate {
  func load(_ url: URL) async throws -> Data {
    let session = URLSession(configuration: .default,
                             delegate: self, delegateQueue: nil)
    let (data, _) = try await session.data(from: url)
    return data
  }
}
```

It creates a new `URLSession` per request: every call pays a fresh TCP and TLS handshake because the connection pool belongs to the session, and every session keeps its delegate alive until it is invalidated, so nothing is released.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63309)

### 9. How would you structure a networking layer on top of `URLSession`, and which errors should it distinguish?

Put one client between the app and `URLSession` that builds requests from typed endpoints, sends them, validates the status and decodes the body, and have it throw one error type whose cases match what the UI does next.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-3302)

### 10. When would you use `URLSession` directly, and when a library like Alamofire?

Modern `URLSession` with async/await and `Codable` covers most apps in a few hundred lines you own; Alamofire pays off when you want its ready-made interceptors, retry policies, validation, multipart and logging instead of writing and testing them yourself.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63320)

### 11. Decoding fails with "Expected date string to be ISO8601-formatted" although `created_at` looks like a normal ISO date. What does `JSONDecoder` object to?

`.iso8601` is strict: it needs a full date-time with a time zone, like `2026-03-01T12:00:00Z`, so a date-only value, a time without `Z` or an offset, or a space instead of `T` throws; older OS versions also rejected fractional seconds.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63308)

### 12. The backend adds a new post type and the whole feed goes blank instead of skipping one row. How do you make decoding survive it?

Decoding an array is all-or-nothing: `JSONDecoder` throws on the first element it cannot build and you lose the other forty-nine, so make unknown values decodable and isolate the failure to a single element.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63310)

### 13. How do you cancel network requests and handle timeouts in Swift?

Cancel by cancelling the `Task` that awaits the request, which cancels the underlying `URLSessionTask` and throws `CancellationError` or `URLError(.cancelled)`; bound waiting with `timeoutIntervalForRequest` and `timeoutIntervalForResource`, and never show a cancellation to the user as an error.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-3303)

### 14. Find the bug: when the backend returns an error page, the profile screen spins forever instead of showing an error.

```swift
func fetchProfile(id: Int) async throws -> Profile {
    try await withCheckedThrowingContinuation { continuation in
        LegacyAPI.shared.get("/profiles/\(id)") { data, error in
            if let error {
                continuation.resume(throwing: error)
                return
            }
            guard let data,
                  let profile = try? JSONDecoder().decode(Profile.self, from: data) else {
                return
            }
            continuation.resume(returning: profile)
        }
    }
}
```

The `guard … else { return }` exits without resuming the continuation, so when decoding fails (an HTML `502` page, a changed contract) or `data` is `nil`, the awaiting task stays suspended forever and the spinner never stops.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63321)

### 15. The connection drops mid-request. How should the networking layer react, and why not check reachability before every call?

A drop after the request has started fails it with `URLError.networkConnectionLost`: retry an idempotent request with backoff, and set `waitsForConnectivity` so requests that cannot start yet wait; do not gate calls on a reachability check.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63303)

### 16. Which failed requests do you retry, and how do you retry without stampeding the server?

Retry only failures a later attempt could plausibly fix, and space attempts with exponential backoff plus random jitter so a recovering server is not hit by every device at the same instant.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63302)

### 17. How do you cache HTTP responses on iOS, and what do `URLCache`, `Cache-Control` and `ETag` each do?

`URLCache` is the HTTP cache built into `URLSession`: `Cache-Control: max-age` lets it answer with no request at all, and an `ETag` lets it revalidate so the server replies `304` with no body.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63301)

### 18. How do you upload a file as `multipart/form-data` with `URLSession`?

URLSession has no multipart builder, so you assemble the body yourself: a unique boundary string, one part per field carrying its own Content-Disposition header, and a closing boundary that ends in two extra dashes.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-3305)

### 19. How do you test code that calls URLSession without touching the network?

Two techniques cover it: inject a protocol you can fake when you are testing your own client, and register a `URLProtocol` subclass when the real URL loading path is what you need to exercise.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63306)

### 20. When would you use a WebSocket instead of HTTP requests, and what does a client built on `URLSessionWebSocketTask` need?

A WebSocket keeps one two-way connection open so the server can push messages the moment they happen (live prices, chat, multiplayer); `URLSessionWebSocketTask` gives you send and receive, while the read loop, keepalive and reconnect are yours to write.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63305)

### 21. How does token authentication work in an app, and how do you refresh the access token when many requests fail with 401 at once?

The app sends a short-lived access token (often a JWT) with every request and keeps a long-lived refresh token to get a new one; when several requests hit 401 together, they must share one refresh through an actor and then each replays once.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-3304)

### 22. An export endpoint streams 400 MB of NDJSON and `data(for:)` gets the app killed for memory on older phones. What do you call instead?

`data(for:)` buffers the whole body in memory before it returns, while `bytes(for:)` hands you the response as an `AsyncSequence` you consume as it arrives, so peak memory is one record instead of the entire payload.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63311)

### 23. What is certificate pinning, how do you implement it on iOS, and what can go wrong?

Pinning makes the app accept a server only if its certificate chain contains a key you shipped in the app, so a man-in-the-middle holding a certificate from any other trusted CA (a corporate proxy, a compromised CA, a root the user was talked into installing) is rejected.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-3306)

### 24. A two-gigabyte video must finish downloading even if the user leaves the app. How do you do it?

A background session hands the transfer to a system daemon that keeps working while your app is suspended or terminated, and reports back by relaunching the app.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63304)

### 25. Auth, logging, retry and metrics all have to wrap every request without turning the client into a god object. How do you compose them?

Model each concern as a middleware that receives the request plus a `next` closure it must call, then fold the array into one send function: the position in the array is the nesting order, and the nesting order is the behaviour.

[Full answer →](https://irbisa.com/topic/ios-networking/#q-63313)
