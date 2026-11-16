# iOS System Design interview questions

The design round: feed, chat, live tracking, uploads, pagination, caching, idempotency, server-driven UI, analytics, super-apps. Senior · 16 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/ios-system-design/)** · [All iOS topics](../README.md#all-topics)

### 1. You have 45 to 60 minutes to design an iOS feature on a whiteboard. How do you split the time, and what is the interviewer actually grading?

Spend the first ten minutes pinning down requirements, then sketch the client architecture and the API, then go deep on the one or two parts that are hard on a phone. The grade is for reasoning about trade-offs under iOS constraints, not for the number of boxes.

[Full answer →](https://irbisa.com/topic/ios-system-design/#q-73901)

### 2. REST, GraphQL or gRPC: which API style do you propose for the iOS client, and what does each one cost the app?

Default to REST with JSON: `URLSession`, `Codable`, HTTP caching and every debugging proxy work out of the box. GraphQL pays off when many screens need different slices of many services, gRPC when the company already lives on protobuf or needs streaming; both move work into the client.

[Full answer →](https://irbisa.com/topic/ios-system-design/#q-73902)

### 3. Why does offset pagination show duplicates and gaps in a feed that changes while the user scrolls, and what does a cursor fix?

An offset counts rows from the top, so every post added or deleted above the user's position shifts the window: an insert repeats the last item of the previous page, a delete skips one. A cursor names the last item the client has and asks for what comes after it, which nothing above can move.

[Full answer →](https://irbisa.com/topic/ios-system-design/#q-73903)

### 4. Every screen must be "fresh", so the app sends forty requests on cold start. Which data do you cache, for how long, and where is stale data acceptable?

Ask what a stale value costs for each kind of data, not whether it may be stale. Most of the forty are config, reference data and content that can be drawn from disk at once and refreshed behind it; only a few values, like a balance or a price at checkout, must be fetched before you trust them.

[Full answer →](https://irbisa.com/topic/ios-system-design/#q-73904)

### 5. A banking home screen shows six widgets from six services and stays blank until the slowest answers. What do you change on the client and the backend?

Let every widget load, fail and retry on its own, starting from its last cached value, so the screen is never hostage to its slowest dependency. On the backend, give the screen one aggregating endpoint with a deadline per section that returns partial results instead of waiting for everything.

[Full answer →](https://irbisa.com/topic/ios-system-design/#q-73905)

### 6. Find the bug: tap Like and then Unlike quickly on a weak network, and the heart stays filled while the server has the post unliked.

```swift
@MainActor final class PostCellModel {
  private(set) var post: Post
  private let api: any LikeAPI
  init(post: Post, api: any LikeAPI) { self.post = post; self.api = api }

  func likeTapped() {
    post.isLiked.toggle()                       // optimistic: the heart fills at once
    post.likeCount += post.isLiked ? 1 : -1
    Task {
      do { try await api.toggleLike(postID: post.id) }
      catch {
        post.isLiked.toggle()                   // roll back
        post.likeCount += post.isLiked ? 1 : -1
      }
    }
  }
}
```

Two relative `toggleLike` requests race, and the rollback flips the current state instead of restoring the server's: when the first request times out after the server applied it, its `catch` re-fills the heart that the second tap had just emptied.

[Full answer →](https://irbisa.com/topic/ios-system-design/#q-73906)

### 7. A payment request times out and the user taps Pay again. How do the client and the API make sure the card is charged only once?

With an idempotency key that names the payment, not the request: the app creates it once per order, saves it before sending and reuses it on every retry, and the server stores the first result under that key and returns it for any repeat instead of charging again.

[Full answer →](https://irbisa.com/topic/ios-system-design/#q-73907)

### 8. A flash sale should end at midnight, yet some users can still open it the next morning. Which clock does the app trust, and who enforces the end?

No clock on the device: the user can change the date in Settings, and "midnight" differs by time zone. The server owns the end as an absolute instant and refuses a purchase after it; the app only displays it, using the server's time plus a monotonic clock that ignores Settings.

[Full answer →](https://irbisa.com/topic/ios-system-design/#q-73908)

### 9. Design the feed of an Instagram-like app: data flow, pagination, prefetching, media, and getting the user back to the same post after a relaunch.

The feed is a local cache the UI observes: a repository pages from the API by cursor into a store, the list renders from the store, media loads ahead of the scroll at the size of the cell, and the position is saved as an anchor post id so a relaunch can open where the user left off.

[Full answer →](https://irbisa.com/topic/ios-system-design/#q-73909)

### 10. Design a stories viewer like Instagram's: what do you preload, how many video players do you keep alive, and what is the memory budget?

Preload one step ahead in each direction the user can go: the next segment of the current story and the first segment of the next person's. Keep two or three players for the current and next segment, reused rather than created per tap, and budget memory in decoded full-screen frames.

[Full answer →](https://irbisa.com/topic/ios-system-design/#q-73910)

### 11. Design posting a marketplace listing with ten photos: compression, an upload queue, retries, and a draft that survives the app being killed.

Save the draft and processed photos locally first, upload each photo as a file task in a background `URLSession`, and create the listing with an idempotent request once every photo has a server id.

[Full answer →](https://irbisa.com/topic/ios-system-design/#q-73911)

### 12. Design a messenger client like Telegram: transport, message states, local storage, ordering, and what happens offline or in the background.

The local database is the chat: the UI renders from it, an outgoing message is written there first with a client id, and a sync layer uses a socket while the app is open and catches up by sequence number after every gap. In the background APNs and a notification service extension take over.

[Full answer →](https://irbisa.com/topic/ios-system-design/#q-73912)

### 13. Design live courier tracking: how does the courier's location reach the customer's screen, and what changes once the app is in the background or the phone is locked?

The courier's app sends throttled location fixes to the server, which matches them to the road and pushes them to the customer over a socket while the map is open. Once the customer leaves the app or locks the phone, a Live Activity updated by push takes over: a suspended app receives nothing.

[Full answer →](https://irbisa.com/topic/ios-system-design/#q-73913)

### 14. How would you build the app's own analytics layer: batching events, keeping them offline, sampling, honouring consent and switching vendors without touching features?

Features call one typed `track` API that returns at once; events land in a disk-backed queue, leave in compressed batches with a unique id each, and fan out to vendor adapters behind a protocol. Consent and sampling are decided once, at the front of that pipeline.

[Full answer →](https://irbisa.com/topic/ios-system-design/#q-73914)

### 15. When is server-driven UI worth it, and how does an old app version cope with a component it has never seen?

It pays off on screens product rearranges weekly (home, promos, catalogue layouts), where App Review and slow updates are the bottleneck. Old builds cope because every request says which components the build can draw, and an unknown type decodes to a fallback instead of failing the screen.

[Full answer →](https://irbisa.com/topic/ios-system-design/#q-73915)

### 16. A dozen teams ship banking, taxi and food into one iOS super-app. How do you structure it so no team slows launch or another team's release?

As a thin host app on a shared platform layer, with a pair of modules per business: a small interface module other teams may import and an implementation only the host links. Nothing starts at launch, budgets are enforced in CI, and features ship dark behind flags on a shared release train.

[Full answer →](https://irbisa.com/topic/ios-system-design/#q-73916)
