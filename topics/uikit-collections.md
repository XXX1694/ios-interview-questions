# Collection Views & Diffable Data interview questions

Compositional layout, diffable data sources, cell registration, content configuration, prefetching. Middle · 23 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/uikit-collections/)** · [All iOS topics](../README.md#all-topics)

### 1. When would you choose `UICollectionView` over `UITableView`?

Choose a collection view whenever the screen is anything other than one vertical column of full-width rows (grids, carousels, mixed sections) and, since iOS 14, often for plain lists too, because a list configuration gives table-style rows inside it.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69317)

### 2. Products that are not on sale start showing the SALE badge after some scrolling: what is wrong with this code?

```swift
override func collectionView(_ collectionView: UICollectionView,
                             cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
  let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "ProductCell",
                                                for: indexPath) as! ProductCell
  let product = products[indexPath.item]
  cell.titleLabel.text = product.title
  if product.isOnSale {
    cell.saleBadge.isHidden = false
    cell.priceLabel.textColor = .systemRed
  }
  return cell
}
```

The cell is reused, and the code sets the badge and the red price only in the `true` branch: a cell that last showed a sale item keeps both when it is dequeued for a normal product. The title is fine because it is set on every path.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69319)

### 3. Find the bug: swiping to delete a note crashes the app with `NSInternalInconsistencyException`.

```swift
override func tableView(_ tableView: UITableView,
                        commit editingStyle: UITableViewCell.EditingStyle,
                        forRowAt indexPath: IndexPath) {
  guard editingStyle == .delete else { return }
  tableView.deleteRows(at: [indexPath], with: .automatic)
  notes.remove(at: indexPath.row)
  storage.save(notes)
}
```

`deleteRows` runs before the model shrinks: the table asks the data source for the new row count during the update, still gets the old one, and throws `NSInternalInconsistencyException` with "Invalid update: invalid number of rows in section 0".

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69326)

### 4. Cells in a photo grid flash the previous item's image during fast scrolling: what causes it, and how do you fix it?

The dequeued cell is a recycled object that still shows the old image, and a request started for its previous item later finishes into it. Clear the image and cancel the load on reuse, and check which item the cell shows before setting a result.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69301)

### 5. Find the bug: after new songs arrive, tapping the star on a row sometimes favorites a different song.

```swift
override func tableView(_ tableView: UITableView,
                        cellForRowAt indexPath: IndexPath) -> UITableViewCell {
  let cell = tableView.dequeueReusableCell(withIdentifier: "Song", for: indexPath) as! SongCell
  cell.titleLabel.text = songs[indexPath.row].title
  cell.starButton.tag = indexPath.row
  cell.starButton.addTarget(self, action: #selector(starTapped(_:)), for: .touchUpInside)
  return cell
}

@objc private func starTapped(_ sender: UIButton) {
  songs[sender.tag].isFavorite.toggle()
}

func songArrived(_ song: Song) {
  songs.insert(song, at: 0)
  tableView.insertRows(at: [IndexPath(row: 0, section: 0)], with: .automatic)
}
```

The row index is copied into `starButton.tag` when the cell is configured, and `insertRows` does not reconfigure cells already on screen, so after a song arrives at the top, every visible star keeps its old index and toggles the song one row above.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69325)

### 6. How do you build a three-column grid with `UICollectionViewFlowLayout`, and when do you switch to compositional layout?

Flow layout places items in a line and wraps to the next one, so a grid is just an item width computed from the available width: subtract the section insets and the gaps, divide by the column count, and round down.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69320)

### 7. Why use a diffable data source instead of `reloadData` or `performBatchUpdates`?

Because you describe the whole state as a snapshot and UIKit works out the inserts, deletes and moves itself. `reloadData` has no animation and resets every visible cell; `performBatchUpdates` crashes as soon as your index-path arithmetic disagrees with the model.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69321)

### 8. Should a diffable snapshot hold your model structs or only their ids, and what breaks when it holds the structs?

Only stable ids. The diff matches items by their `Hashable` identity, so when the struct is the identifier, editing any field creates a new identity (the row animates out and back in), and two rows with equal values become duplicates that crash the snapshot.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69307)

### 9. Toggling a favorite star with `reloadItems` makes the artwork flicker and drops the keyboard from a text field in the same row: what should you have called?

`snapshot.reconfigureItems([id])`: it runs your cell registration again on the *same* cell that is on screen, while `reloadItems` throws that cell away and dequeues a fresh one.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69309)

### 10. What do `CellRegistration` and `contentConfiguration` replace in a modern collection view, and what do they give you?

They replace string reuse identifiers with `as!` casts and hand-set labels and image views, removing bugs that approach allowed: a registration ties a cell class to an item type at compile time, and a content configuration makes the cell's look a value you assign instead of properties you mutate.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69306)

### 11. This data source compiles without warnings: what happens when it runs, and how do you fix it?

```swift
dataSource = UICollectionViewDiffableDataSource<Section, Contact.ID>(collectionView: collectionView) {
  collectionView, indexPath, id in
  let registration = UICollectionView.CellRegistration<UICollectionViewListCell, Contact.ID> {
    cell, _, id in
    var content = cell.defaultContentConfiguration()
    content.text = ContactStore.shared.name(for: id)
    cell.contentConfiguration = content
  }
  return collectionView.dequeueConfiguredReusableCell(using: registration, for: indexPath, item: id)
}
```

On iOS 15 and later it throws on the first dequeue: UIKit detects a registration created inside the cell provider. Each call would create a new registration with its own reuse identifier, so no cell could ever be reused.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69322)

### 12. In a compositional layout, what do item, group and section each contribute, and what does `.fractionalWidth(1.0)` actually measure against?

An item is one cell's slot, a group lines items up along an axis, a section repeats its group and adds insets and scrolling, and the layout holds the sections. A fractional size is always a fraction of the element's container: an item's group, a group's section.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69302)

### 13. A feed of self-sizing cells jumps as you scroll and the scroll indicator keeps changing size: what is causing the jump?

Until a cell is measured, the layout counts its *estimated* height in the content size. Each real measurement replaces an estimate, the total changes, and the content under the finger shifts; the further the estimate is from reality, the bigger the jump.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69311)

### 14. Design wants an inset-grouped table look with separators and swipe-to-delete, but the screen is already a collection view: what do you reach for?

`UICollectionLayoutListConfiguration`: it turns a compositional layout, or one section of it, into table-style rows with separators, headers, swipe actions and accessories.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69305)

### 15. The user flings past a hundred rows and image requests keep landing for cells that scrolled off long ago: what did the prefetch code skip?

`collectionView(_:cancelPrefetchingForItemsAt:)`: without it every fling fills the download and decode queues with work for items nobody will look at, and the visible rows wait behind it.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69312)

### 16. This infinite-scroll code sometimes loads the same page twice and then crashes with a duplicate-identifier error: what is wrong, and how should it be built?

```swift
func collectionView(_ collectionView: UICollectionView,
                    willDisplay cell: UICollectionViewCell,
                    forItemAt indexPath: IndexPath) {
  guard indexPath.item == posts.count - 1 else { return }
  Task {
    let page = try await api.fetchPosts(page: currentPage + 1)
    currentPage += 1
    posts += page
    var snapshot = dataSource.snapshot()
    snapshot.appendItems(page.map(\.id))
    await dataSource.apply(snapshot)
  }
}
```

Nothing stops a second request while the first is in flight. `willDisplay` fires for the last cell again (a bounce, a re-layout), both tasks read the same `currentPage`, and the second append adds identifiers already in the snapshot: "supplied item identifiers are not unique".

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69323)

### 17. How do you put a horizontally scrolling collection view inside a table view cell, and what goes wrong on reuse?

Give each cell its own collection view and data source, pass the row's items in on configure, and save each row's horizontal offset by row id; otherwise reuse shows one row scrolled where another row was left, and a row you scrolled resets when it comes back.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69324)

### 18. A drag reorder looks correct until the next model refresh puts the row back where it started: what did the reordering handler miss?

UIKit moved the cell and the data source updated its own snapshot, but nothing wrote the new order back to your model, so the next snapshot built from the model puts the row back.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69313)

### 19. You need a rounded tinted card behind an entire section and a small badge hanging off a cell's corner: which layout objects give you those?

A decoration item for the section background and an anchored supplementary item for the badge. The layout alone creates and places decorations; supplementaries are dequeued through your data source like cells.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69304)

### 20. A horizontally paging carousel has to scale its center card up as the user swipes: how do you build that inside a compositional layout?

Set the section's `orthogonalScrollingBehavior` to `.groupPagingCentered` and scale the cards in `visibleItemsInvalidationHandler`, which runs on every scroll frame of that section with the visible items and the current offset.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69303)

### 21. You wrote a custom `UICollectionViewLayout` for a staggered grid and scrolling crawls: which method is doing too much?

`layoutAttributesForElements(in:)`: UIKit calls it again and again while the user scrolls, so a version that computes frames or walks every item makes each scroll step cost O(items).

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69314)

### 22. A live feed of ten thousand rows freezes for a moment on every socket update: where is `apply` spending the time, and what do you change?

Inside `apply`: it hashes and compares every identifier to find the changes, then animates each change, on the main thread. With heavy struct identifiers and one `apply` per socket message, that full diff repeats many times a second.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69315)

### 23. You inherit a 600-line `UITableViewDelegate` and are told to move it onto a compositional list: what moves first, and how do you prove nothing regressed?

Extract the row model first (whatever `cellForRowAt` switches on becomes the item identifier of a snapshot) and cover that pure state-to-rows function with tests before any UIKit type changes.

[Full answer →](https://irbisa.com/topic/uikit-collections/#q-69316)
