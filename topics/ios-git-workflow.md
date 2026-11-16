# Git & Team Workflow interview questions

project.pbxproj and Package.resolved conflicts, shared schemes, App Store build tags, storyboards, worktrees, leaked App Store Connect keys. Junior · 18 questions.

**[Practice this topic on irbisa.com →](https://irbisa.com/topic/ios-git-workflow/)** · [All iOS topics](../README.md#all-topics)

### 1. Each build uploaded to App Store Connect gets a tag such as `v1.8.0-412`. What are a commit, a branch and a tag, and why tag the build rather than note the branch?

A commit is an immutable snapshot of the project that records its parent, a branch is a name that moves forward with each new commit, and a **tag** is a name fixed to one commit, so `v1.8.0-412` always shows exactly which code Apple reviewed as build 412.

[Full answer →](https://irbisa.com/topic/ios-git-workflow/#q-75901)

### 2. Xcode's Integrate menu offers Fetch Changes and Pull. What does each one do, and why set `pull.rebase` for your local `main`?

**Fetch Changes** runs `git fetch`: it downloads new commits and updates `origin/main` without touching your branch or files, while Pull also merges or rebases them into the branch you are on. With `pull.rebase` set, your local commits go on top instead of producing a merge commit.

[Full answer →](https://irbisa.com/topic/ios-git-workflow/#q-75902)

### 3. Should you merge `main` into your `feature/paywall` branch or rebase it onto `main` when both sides keep editing `project.pbxproj`?

A merge brings `main` in with one merge commit, while a **rebase** replays each of your commits on top of `main` with new hashes. With `project.pbxproj` changing on both sides, a merge resolves that file once and a rebase possibly once per commit.

[Full answer →](https://irbisa.com/topic/ios-git-workflow/#q-75903)

### 4. How do you resolve a merge conflict in a Swift file, and why are conflicts in a storyboard so much harder?

In a **Swift file** you read both sides between the markers, write code that keeps both intentions, delete the markers, then build and test. A storyboard is generated XML full of random object ids and frame values, so its conflicts rarely make sense to a person.

[Full answer →](https://irbisa.com/topic/ios-git-workflow/#q-75904)

### 5. You must fix a bug on `release/2.1` while your branch has uncommitted SwiftUI changes. When is `git stash` enough, and when is a second worktree better?

**`git stash`** shelves uncommitted changes so you can switch branches, and `git stash pop` restores them later; for a fix that takes hours, `git worktree add` checks `release/2.1` out into its own folder, so your branch and its Xcode state stay untouched.

[Full answer →](https://irbisa.com/topic/ios-git-workflow/#q-75905)

### 6. Which Xcode files belong in `.gitignore` and which must be committed, and why can't your teammate see the scheme you created?

Ignore per-user and generated files such as `xcuserdata/`, `DerivedData/` and `*.xcuserstate`, and commit the project, `Package.resolved` and **shared schemes**: a scheme with Shared unticked lives in `xcuserdata`, and ticking it moves the file to `xcshareddata/xcschemes`.

[Full answer →](https://irbisa.com/topic/ios-git-workflow/#q-75906)

### 7. Your branch and `main` both conflict in `project.pbxproj` and `Package.resolved`. How do you resolve each, and how do you stop the `.pbxproj` conflicts coming back?

In **`project.pbxproj`** keep both sides' entries in every section, because each added file appears in several places, then lint and build. `Package.resolved` is resolver output: take `main`'s copy and let Xcode resolve your additions again.

[Full answer →](https://irbisa.com/topic/ios-git-workflow/#q-75907)

### 8. Starting from the state below each time, what does each command change, and which ones can lose work?

```shell
# feature/settings, nothing pushed yet
#   b4 Add notification toggle   <- HEAD
# staged:   Info.plist (CFBundleVersion 411 -> 412)
# unstaged: Sources/Settings/SettingsView.swift

git restore Sources/Settings/SettingsView.swift   # A
git restore --staged Info.plist                   # B
git reset --soft HEAD~1                           # C
git reset --hard                                  # D
```

A throws away the unstaged edits in `SettingsView.swift`, B only unstages `Info.plist`, C undoes commit `b4` but keeps its changes staged, and **`git reset --hard`** (D) wipes every staged and unstaged edit. A and D can lose work for good.

[Full answer →](https://irbisa.com/topic/ios-git-workflow/#q-75908)

### 9. One commit mixes a new SwiftUI screen with a 400-line `project.pbxproj` reshuffle. How do you split it with an interactive rebase and push the result safely?

Mark the commit `edit` in `git rebase -i`; when Git stops there, `git reset HEAD~` undoes the commit but keeps its changes, so you commit the project file and the screen separately, run `git rebase --continue` and push with **`--force-with-lease`**.

[Full answer →](https://irbisa.com/topic/ios-git-workflow/#q-75909)

### 10. After an interactive rebase, the WidgetKit timeline commit is missing from `feature/widgets`. How do you find it with `git reflog` and bring it back?

A rebase writes new commits and moves the branch, but the old commits stay in the repository: **`git reflog`** still lists them with their hashes, so `git cherry-pick <sha>` brings the lost one back, or `git reset --hard ORIG_HEAD` undoes the whole rebase.

[Full answer →](https://irbisa.com/topic/ios-git-workflow/#q-75910)

### 11. The app now crashes on launch on iOS 17 only, and 120 commits landed since the last good TestFlight build. How do you find the culprit with `git bisect run`?

Mark `HEAD` bad and the tag of the last good build good, then let **`git bisect run`** execute a script that builds the app and runs one launch UI test on an iOS 17 simulator; about seven steps cover 120 commits.

[Full answer →](https://irbisa.com/topic/ios-git-workflow/#q-75911)

### 12. How do commit messages and Conventional Commits fit an iOS release, where `MARKETING_VERSION` and the build number change for every App Store upload?

Each commit gets a short imperative subject and a body that explains why; with **Conventional Commits** a release tool reads `feat`, `fix` and `!` to propose the next `MARKETING_VERSION`, while CI alone raises `CURRENT_PROJECT_VERSION`, the build number App Store Connect requires to grow.

[Full answer →](https://irbisa.com/topic/ios-git-workflow/#q-75912)

### 13. What makes an iOS pull request easy to review, and what do you check first when reviewing someone else's?

A good iOS pull request is small, says why it exists and how it was tested, and shows **screenshots** of each changed screen in light and dark mode and at a large Dynamic Type size; as a reviewer you check behaviour, threading and memory before style.

[Full answer →](https://irbisa.com/topic/ios-git-workflow/#q-75913)

### 14. App Review can take a day or more. How do GitFlow, GitHub flow and trunk-based development cope with that, and which would you choose for an iOS app?

Most iOS teams use **trunk-based development** with short release branches: small pull requests merge into `main`, a `release/1.9` branch is cut for submission, and fixes found during review land on `main` first and are cherry-picked into the release branch.

[Full answer →](https://irbisa.com/topic/ios-git-workflow/#q-75914)

### 15. Which checks belong in a pre-commit hook for an iOS project, and why must CI repeat them anyway?

A **pre-commit hook** should run only fast checks on the staged Swift files, such as SwiftFormat and SwiftLint, while builds and tests stay in CI: hooks live in `.git/hooks`, are not cloned with the repository, and `git commit --no-verify` skips them.

[Full answer →](https://irbisa.com/topic/ios-git-workflow/#q-75915)

### 16. An App Store Connect API key, `AuthKey_7XK2P9.p8`, was pushed along with the fastlane setup. What do you do, and why is removing it in the next commit not enough?

**Revoke the key** in App Store Connect under Users and Access, Integrations, and create a new one kept as a CI secret: every clone still has the old file in its history, and a delete commit only hides it from the latest version.

[Full answer →](https://irbisa.com/topic/ios-git-workflow/#q-75916)

### 17. Onboarding PR #88 was reverted on `main` after it crashed on first launch. The fixed branch then merged cleanly, but the new onboarding never appears. What went wrong?

```shell
git switch main
git merge --no-ff feature/onboarding-v2     # PR #88: six commits, new SwiftUI flow
git revert -m 1 HEAD                        # crash on first launch: back it out
git push

git switch feature/onboarding-v2
git commit -am "fix: no force unwrap of the stored user on first launch"
git switch main
git merge feature/onboarding-v2             # 1 file changed
```

The **revert commit** undid the onboarding code, but the six feature commits are still in `main`'s history, so the second merge adds only what is new on the branch: the one-file fix, applied to a `main` that no longer has the feature.

[Full answer →](https://irbisa.com/topic/ios-git-workflow/#q-75917)

### 18. Your app, its widget extension and a second app share networking and UI code. Local Swift packages in one repository, or separate repositories with version tags?

Start with **local Swift packages** in the app's repository: the apps and extensions change shared code and its callers in one pull request, and Xcode builds everything from source. Move a package into its own repository only when it gets separate owners or outside consumers.

[Full answer →](https://irbisa.com/topic/ios-git-workflow/#q-75918)
