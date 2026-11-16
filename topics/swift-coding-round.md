# Live Coding Round in Swift interview questions

Big O, hashing, stacks and queues, two pointers, sliding window, binary search, linked lists, trees, LRU cache, heaps and DP in Swift. Middle · 18 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/swift-coding-round/)** · [All iOS topics](../README.md#all-topics)

### 1. What is Big-O notation, and how do you state the time and space complexity of your solution in a coding round?

Big O describes how an algorithm's running time or memory grows with the input size n, keeping only the fastest-growing term and dropping constants. In a coding round you state both, as in "O(n) time, O(n) extra space", because a faster solution often pays with memory.

[Full answer →](https://irbisa.com/topic/swift-coding-round/#q-74001)

### 2. How does a `Set` or a `Dictionary` turn two-sum, duplicate detection or an anagram check from O(n²) into O(n) in Swift?

Instead of comparing every element with every other one, remember what you have already seen in a hash-based `Set` or `Dictionary`. A lookup there is O(1) on average, so one pass over the input is enough, paid for with O(n) extra memory.

[Full answer →](https://irbisa.com/topic/swift-coding-round/#q-74002)

### 3. How do you implement a stack and a queue in Swift, and why is `Array.removeFirst()` a poor way to dequeue?

An `Array` already is a stack: `append` pushes and `popLast()` pops, both O(1). A queue also needs O(1) at the front, and `removeFirst()` shifts every remaining element down, so draining n items that way costs O(n²) in total.

[Full answer →](https://irbisa.com/topic/swift-coding-round/#q-74003)

### 4. Live task: check that a string of brackets like `"([]{})"` is balanced. Which structure do you use, and which inputs break a naive solution?

A stack: push every opening bracket, and on a closing one pop and check that it is the matching opener. The string is balanced only if no check failed and the stack is empty at the end. One pass: O(n) time, O(n) memory in the worst case.

[Full answer →](https://irbisa.com/topic/swift-coding-round/#q-74004)

### 5. Live task: merge two sorted arrays into one sorted array in O(n + m) without calling `sorted()`. What are the edge cases?

Walk both arrays with two indices, append the smaller of the two current elements and advance that index; when one array runs out, append the rest of the other. Every element is read once: O(n + m) time, plus O(n + m) for the result.

[Full answer →](https://irbisa.com/topic/swift-coding-round/#q-74005)

### 6. Live task: write your own `map` and `reduce` as extensions on `Sequence`. What do generics and `rethrows` add to the signature?

Extend `Sequence`, loop with `for-in` and build the result. Generic parameters (`T`, `Result`) let the output type differ from the element type, and a `throws` closure with a `rethrows` method means callers write `try` only when the closure they pass can throw.

[Full answer →](https://irbisa.com/topic/swift-coding-round/#q-74006)

### 7. How do `Dictionary(grouping:by:)`, `reduce(into:)` and `uniquingKeysWith:` solve "group anagrams" and "count words", and why is plain `reduce` a trap here?

They replace the hand-written "look up, insert if missing, append" loop: `Dictionary(grouping:by:)` buckets elements by a key you compute, `reduce(into:)` folds into one dictionary that you mutate in place, and `uniquingKeysWith:` merges the values of repeated keys.

[Full answer →](https://irbisa.com/topic/swift-coding-round/#q-74007)

### 8. Find the bug: this palindrome check returns the right answer, but takes seconds on a 100,000-character string.

```swift
func isPalindrome(_ s: String) -> Bool {
  var left = 0
  var right = s.count - 1
  while left < right {
    let l = s[s.index(s.startIndex, offsetBy: left)]
    let r = s[s.index(s.startIndex, offsetBy: right)]
    if l != r { return false }
    left += 1
    right -= 1
  }
  return true
}
```

`s.index(s.startIndex, offsetBy: k)` walks the string from the start on every call, because a `Character` can span several bytes, so the loop is O(n²): billions of steps for 100,000 characters, about 9 seconds in a Release build on a Mac. Move two `String.Index` values instead.

[Full answer →](https://irbisa.com/topic/swift-coding-round/#q-74008)

### 9. Which algorithm does Swift's `sort()` use, is it stable, and how do you sort users by score descending and then by name?

`sort()` is a modified timsort: O(n log n) in the worst case, near O(n) on mostly sorted input, and documented as stable, so equal elements keep their order. For several keys, write one comparator that compares the primary key and falls back to the next only on a tie.

[Full answer →](https://irbisa.com/topic/swift-coding-round/#q-74009)

### 10. Live task: merge overlapping time slots like `[1, 3], [2, 6], [8, 10]`. Why sort first, and should touching slots such as `[1, 3]` and `[3, 5]` merge?

Sort the slots by start, then sweep once: if a slot starts before the last merged slot ends, extend that one to the later of the two ends, otherwise start a new slot. Sorting costs O(n log n) and dominates the O(n) sweep. Whether touching slots merge depends on whether the ends are inclusive.

[Full answer →](https://irbisa.com/topic/swift-coding-round/#q-74010)

### 11. How do the two-pointer and sliding-window techniques turn an O(n²) scan into O(n)? Show it on the longest substring without repeating characters.

Both replace the nested loop with indices that only move forward. Two pointers close in on a sorted array from both ends; a sliding window keeps a contiguous range, growing its right edge and moving its left edge past whatever made the window invalid. Each index crosses the input once: O(n).

[Full answer →](https://irbisa.com/topic/swift-coding-round/#q-74011)

### 12. This binary search crashes on some inputs and never returns on others. What is wrong with it, and which loop invariant makes it correct?

```swift
func indexOf(_ target: Int, in a: [Int]) -> Int? {
  var lo = 0
  var hi = a.count
  while lo <= hi {
    let mid = (lo + hi) / 2
    if a[mid] == target { return mid }
    if a[mid] < target {
      lo = mid
    } else {
      hi = mid - 1
    }
  }
  return nil
}
```

It mixes two conventions: `hi = a.count` is a half-open bound, but `lo <= hi` treats it as closed, so an empty array reads `a[0]` and traps; and when `a[mid]` is too small, `lo = mid` keeps `mid` in range, so once `mid == lo` nothing changes and the loop spins forever.

[Full answer →](https://irbisa.com/topic/swift-coding-round/#q-74012)

### 13. Live task: reverse a singly linked list in place. Follow-up: can freeing a million-node list overflow the stack in Swift, and what about a deep tree?

Walk the list once, saving `node.next` before pointing it back at `prev`: O(n) time, O(1) memory. Freeing a long `Node` list is safe since Swift 5.7, which releases a chain of `next` links in a loop, but a deep tree or an `indirect enum` list still frees recursively and can crash.

[Full answer →](https://irbisa.com/topic/swift-coding-round/#q-74013)

### 14. How do you traverse a binary tree depth-first and breadth-first in Swift, and which one fits "max depth" and which "level order"?

Depth-first goes down one branch before backing up, through recursion or an explicit stack; breadth-first visits the tree level by level. Max depth is a natural DFS: a node's depth is one more than its deeper subtree. Level order is BFS by definition.

[Full answer →](https://irbisa.com/topic/swift-coding-round/#q-74014)

### 15. Live task: implement an LRU cache with O(1) `get` and `put`. Which two structures do you combine, and why is a `Dictionary` alone not enough?

A `Dictionary` from key to node finds an entry in O(1), and a doubly linked list keeps the nodes in order of use, so moving a node to the front and dropping the last one are O(1) pointer updates. A dictionary alone has no order, and an array of keys makes every touch O(n).

[Full answer →](https://irbisa.com/topic/swift-coding-round/#q-74015)

### 16. How do you recognise a dynamic programming problem, and how do you go from plain recursion to memoization to a bottom-up table in Swift?

It is DP when the answer is built from answers to smaller versions of the same problem, and a plain recursion would solve the same subproblem many times. Write the recursion, cache it by its state (memoization), then fill the same states in a loop from the smallest up (bottom-up).

[Full answer →](https://irbisa.com/topic/swift-coding-round/#q-74016)

### 17. Swift's standard library has no heap. How do you find the 100 most frequent words in a large text, and what does each approach cost?

Count the words into a dictionary in one O(n) pass, then pick the top k of the u distinct words: sort them all (O(u log u)), keep a min-heap of k entries (O(u log k)), or bucket them by count (O(n + u)). Swift has no heap built in: write a small one or use `Heap` from swift-collections.

[Full answer →](https://irbisa.com/topic/swift-coding-round/#q-74017)

### 18. Feature modules declare which modules they depend on. How do you compute a build order with a topological sort, and how does the same pass detect a cycle?

Treat the modules as a directed graph and run Kahn's algorithm: count each module's unbuilt dependencies, start with the modules at zero, and every time one is "built", decrement its dependents and add any that reach zero. Modules left over at the end sit on a cycle or depend on one.

[Full answer →](https://irbisa.com/topic/swift-coding-round/#q-74018)
