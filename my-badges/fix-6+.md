<img src="https://my-badges.github.io/my-badges/fix-6+.png" alt="I did 8 sequential fixes." title="I did 8 sequential fixes." width="128">
<strong>I did 8 sequential fixes.</strong>
<br><br>

Commits:

- <a href="https://github.com/Sajjon/Zhip/commit/4125500b8f355d3045ee38219b36bb91c9d9d276">4125500</a>: fix(review): address Copilot PR comments

Three actionable Copilot review comments on PR #136:

1. Rename `protocol Identifiable` → `ReuseIdentifiable` in
   Sources/Views/TableView/ClassIdentifiable.swift to avoid shadowing
   Swift's stdlib `Identifiable` protocol. The two protocols are
   unrelated — Swift.Identifiable is a SwiftUI/diffing identity, this
   one is a UIKit cell-reuse identifier — but the name collision would
   make any future use of stdlib Identifiable ambiguous. Updated the
   one stale doc reference in SingleCellTypeTableView.swift to match.

2. Trim duplicate trailing empty entry in the Crashlytics
   PBXShellScriptBuildPhase.shellScript array
   (Zhip.xcodeproj/project.pbxproj:2643-2644).

3. Trim duplicate trailing empty entry in the SwiftLint
   PBXShellScriptBuildPhase.shellScript array (same file).

Skipping the fourth comment (Debug code-signing override clearing
PROVISIONING_PROFILE_SPECIFIER) per the user's reply: "this is a
one-off… for an otherwise obsolete repo".

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
- <a href="https://github.com/Sajjon/Zhip/commit/b85810b44b0a6f1735de5d9a4364241f008d4047">b85810b</a>: fix(safety): convert [unowned] → [weak] in 1_Main scenes

MainCoordinator + MainViewModel + SendCoordinator + Send sub-scenes
(PrepareTransaction VM, ScanQRCode View+VM, PollTransactionStatus VM)
+ ReceiveCoordinator + SettingsCoordinator + SettingsViewModel +
RemovePincodeViewModel: replace all remaining [unowned self] (and one
[unowned useCase]) with [weak] equivalents. Same crash-safety rationale
as the navigation-infra and onboarding fixes.

MainViewModel gains an explicit Foundation import so the new
`-> Date?` annotation on the balanceWasUpdatedAt map closure resolves.

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
- <a href="https://github.com/Sajjon/Zhip/commit/863f056021cd7391434c51c5619f434b24d3d223">863f056</a>: fix(safety): convert [unowned] → [weak] in 0_Onboarding scenes

Coordinators (Onboarding, ChooseWallet, CreateNewWallet, BackupWallet,
DecryptKeystore, RestoreWallet, SetPincode) and ViewModels
(TermsOfService, AskForCrashReportingPermissions, WarningCustomECC,
DecryptKeystoreToRevealKeyPair, RestoreWallet, ConfirmNewPincode) and
the RestoreWalletView segment-switch sink.

All conversions follow the pattern in the code-review fix: replace
[unowned self] with [weak self], unwrap with `self?.` for void calls or
`guard let self else { return }` for multi-step bodies, and return
Empty for flatMapLatest closures whose self is gone. ChooseWallet
coordinator's doc comment about the unowned rationale updated to
describe the [weak self] rationale.

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
- <a href="https://github.com/Sajjon/Zhip/commit/f1674caf2296a1ccc3a1a52572e7e601b8b20bb5">f1674ca</a>: fix(safety): convert [unowned] → [weak] in DefaultTransactionsUseCase

sendTransaction's flatMapLatest captured self unowned to call
zilliqaService.sendTransaction. The use case is held by Container, but
the publisher chain can be retained by a subscriber (the Send flow's
ViewModel) past the use case's lifetime in tests/teardown. Switch to
[weak self] returning Empty if self is gone — drops the in-flight send
silently rather than crashing.

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
- <a href="https://github.com/Sajjon/Zhip/commit/4bda3668a6e6168feec6977f6325f8aa61c590a7">4bda366</a>: fix(safety): convert [unowned] → [weak] in core navigation infra

AppCoordinator + Coordinating+Scene{Present,Replace} +
Coordinating+Child+Start + DeepLinkHandler: replace [unowned] capture
lists with [weak] and adjust call sites for the optional unwrap. Same
rationale as the earlier MainCoordinator/SendCoordinator fix — these
publishers (deeplinks, navigator pulses) can outlive the
coordinator/scene during teardown, so [unowned] is a latent crash
trigger. [weak] is the safer default.

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
- <a href="https://github.com/Sajjon/Zhip/commit/30c1f78e69a44f45cd8876f11cc6013c67ca75e4">30c1f78</a>: fix(wallet-create): persist immediately, gate on backup-confirmed flag

Previously: a wallet generated in CreateNewWallet was held in memory only
until the user confirmed "I have backed up" in BackupWalletCoordinator.
An app kill anywhere in between would lose the freshly-derived private key
forever (the key is randomly generated, so password alone can't recover it).

Now:
  - CreateNewWalletCoordinator persists the wallet to Keychain immediately
    after derivation, in the .createWallet branch of toCreateWallet.
  - A new preferences key `hasConfirmedNewWalletBackup` is set to false
    at persist time and flipped true when BackupWalletCoordinator returns
    `.backUp`. Future work can gate Send behind this flag and surface a
    "back up your wallet" banner until it's true.
  - ChooseWalletCoordinator no longer double-saves on the create branch
    (the create coordinator already did it). The restore branch keeps the
    save, since that flow doesn't write the wallet itself. Renamed the
    helper from `userFinishedChoosing(wallet:)` to
    `finishChoosing(wallet:persistFirst:)` so the call sites are explicit
    about which branch owns persistence.

Tests:
  - ChooseWalletCoordinatorTests: drop the now-incorrect save assertion on
    the create branch (responsibility moved); restore-branch save assertion
    stays.
  - CreateNewWalletCoordinatorTests: register an in-memory `Preferences`
    so the new flag write doesn't leak into real UserDefaults during the
    test, plus add two tests covering the new persist-on-create + flag-flip
    behaviour.

Also clean up TestStoreFactory.makeSecurePersistence: the prior
`Preferences.default.save(value: true, for: .hasRunAppBefore)` workaround
is no longer needed (the destructive reinstall side effect was moved out
of the wallet getter into Bootstrap).

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
- <a href="https://github.com/Sajjon/Zhip/commit/69ee99d04f1747bfab76ee6ea8cded8a1197109a">69ee99d</a>: fix(send): pre-verify password against keystore before enabling Sign

SignTransactionViewModel previously enabled the Sign button as soon as the
password met the structural minimum-length policy. The actual decryption
happened during sendTransaction, which then bounced back through
errorTracker and flipped the field red — wasting a network round-trip on a
wrong password and surfacing a confusing post-broadcast-attempt error.

Now the Sign button gates on BOTH structural validity AND a debounced
local keystore-decrypt check via verifyEncryptionPasswordUseCase. Wrong
password → button stays disabled, no network round-trip wasted, error
state is immediate.

Debounced 250ms + removeDuplicates so we don't fire fresh KDF runs on
every keystroke. flatMapLatest cancels any in-flight check when a newer
password lands.

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
- <a href="https://github.com/Sajjon/Zhip/commit/ad2d58176c19c3805b8f5a916ff67721c4624a35">ad2d581</a>: fix(send): gate Review button on real network nonce

Drop `prepend(0)` on the nonce publisher in PrepareTransactionViewModel.

Previously: Review button enabled on a stale nonce=0 before the network
balance/nonce fetch had completed. User submits → network rejects with
"nonce too low" → confusing error mid-Send.

Now: payment stays nil (Review disabled) until the first network response
lands. Disabled state also doubles as a "no network" gate which is the
right UX anyway — sending with no balance/nonce is never sensible.

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>


Created by <a href="https://github.com/my-badges/my-badges">My Badges</a>