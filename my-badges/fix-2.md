<img src="https://my-badges.github.io/my-badges/fix-2.png" alt="I did 2 sequential fixes." title="I did 2 sequential fixes." width="128">
<strong>I did 2 sequential fixes.</strong>
<br><br>

Commits:

- <a href="https://github.com/Sajjon/NanoViewController/commit/d426f9cf9949cd293239dc206480627346d50e1b">d426f9c</a>: Fix swiftlint trailing-closure violations in SinkOnMainTests

CI's swiftlint --strict run flagged two `multiple_closures_with_trailing_closure`
violations in `SinkOnMainTests` that the local pre-commit hook would have
caught — the hook hadn't been installed in this working tree. Both call
sites now pass both closures explicitly.

Pre-commit + pre-push hooks reinstalled locally so future regressions
are caught before push.

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
- <a href="https://github.com/Sajjon/NanoViewController/commit/16fdebc745587ddad3ddc74c023445c9c0e86894">16fdebc</a>: Fix cancel() race + add concurrency-sensitive tests

Bug fix:

* `UIControlSubscription.cancel()` previously wrote `subscriber = nil`
  outside the main hop while `removeTarget` was scheduled inside it,
  giving a read/write race against `handleEvent` (which runs on main).
  Both writes now happen inside the same main-thread block.

* Switched to strong `[self]` captures for both `init`'s `addTarget`
  and `cancel`'s `removeTarget`. `[weak self]` would race with
  `AnyCancellable.deinit` releasing its only strong reference: the main
  hop could resolve `nil` before `removeTarget` ran, leaving a stale
  pointer in UIKit's target/action map.

* Replaced the in-class `@objc handleEvent()` with a non-generic
  `ControlEventTarget: NSObject` that holds the closure. `@objc` methods
  on Swift generic classes don't always survive `objc_msgSend` dispatch
  with their unmangled selector names; routing through a plain
  `NSObject` subclass avoids the issue.

New tests:

* `BindingsBuilderTests` (11 cases) — every result-builder method:
  `buildExpression(AnyCancellable)`, `buildExpression([AnyCancellable])`,
  `buildBlock`, `buildOptional`, `buildEither(first:/second:)`,
  `buildArray`, `buildLimitedAvailability`. Confirms legacy array-literal
  callers still work alongside the new statement form.

* `SinkOnMainTests` (3 cases) — synchronous custom-schedule delivery,
  default-schedule (DispatchQueue.main.async) async delivery, and
  cancellation. Exercises the `@MainActor`-typed inner block path.

* `UIControlPublisherTests` (4 cases) — subscribe+cancel smoke,
  repeated subscribe/cancel without leak (verifies `removeTarget`
  ran), off-main `cancel()` via `Task.detached` (the old
  `MainActor.assumeIsolated` path that would trap), and weak-reference
  semantics. Skips value-delivery testing because
  `UIControl.sendActions(for:)` doesn't fire registered targets in the
  iOS 26 simulator without real touch tracking.

* New `NanoViewControllerDIPrimitivesTests` target with `ClockTests`
  (4 cases) — `MainQueueClock`'s `Task`-based delay semantics:
  block fires after delay, cancelled task skips block, multiple
  scheduled tasks fire independently, cancelling one doesn't affect
  others.

Total: 75 tests pass (was 53), zero strict-concurrency warnings.

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>


Created by <a href="https://github.com/my-badges/my-badges">My Badges</a>