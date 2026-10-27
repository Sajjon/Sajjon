<img src="https://my-badges.github.io/my-badges/fix-4.png" alt="I did 4 sequential fixes." title="I did 4 sequential fixes." width="128">
<strong>I did 4 sequential fixes.</strong>
<br><br>

Commits:

- <a href="https://github.com/Sajjon/Zhip/commit/20b3e71b9da8a03d4b7ddcdd74cc431b54ac671a">20b3e71</a>: fix(security): pasteboard expiration on sensitive copies

Pasteboard.copy(_:) now takes an optional `expiringAfter:` interval. Sites
that copy security-sensitive material (private key, public key on the same
screen, both keystore-copy paths) pass `SensitivePasteboard.expirationSeconds`
(60s). The system pasteboard auto-clears the entry — preventing indefinite
residency for clipboard managers, Universal Clipboard sync to the user's
Mac, or other apps reading pasteboard later.

Non-sensitive copies (receive address, transaction id) keep the legacy
no-expiration behaviour via the protocol-extension overload.

MockPasteboard updated to record (string, expiringAfter) pairs so tests
can assert on the auto-clear policy. Existing copiedString assertions are
unchanged.

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
- <a href="https://github.com/Sajjon/Zhip/commit/dc82a064ceaeb4e05c6f1359b5261600726aa0f4">dc82a06</a>: fix(safety): phase 3 — graceful pop on missing wallet (no fatal traps)

Previously these three coordinators/view-models hit `incorrectImplementation`
(a `Never`-returning fatal) when the wallet was missing at scene-init time.
Reachable via races (wallet removed in Settings while a Send modal was open),
keychain access failures (biometric-protected access flag fires), or any
future feature that touches wallet on the same flow.

Wallet-app crashes inside Send / Backup / Decrypt-Keystore are the worst-tier
UX, so all three now bail out gracefully:

- SignTransactionViewModel adds a new `.walletUnavailable` navigation step;
  on missing wallet, schedules the pulse on next viewDidLoad and returns an
  inert Output. SendCoordinator handles `.walletUnavailable` by finishing
  the whole Send flow.

- BackupWalletCoordinator and DecryptKeystoreCoordinator: replace the
  `walletStorageUseCase.wallet.map { guard … else { incorrectImplementation(…) } }`
  pattern with a `.compactMap { $0 }`. Downstream waits for a wallet that
  never arrives and the user can dismiss the modal cleanly.

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
- <a href="https://github.com/Sajjon/Zhip/commit/8bfce179d843f38277e4101fddc24eced6d20148">8bfce17</a>: fix(security/safety): phase 2 — reinstall wipe + unlock single-fire

- KeyValueStore<KeychainKey>.wallet: pure read, no side effects. The
  destructive reinstall logic moves to wipeStaleKeychainOnReinstallIfNeeded()
  in Bootstrap, called once at launch, routed through the injected
  Preferences/SecurePersistence (no more Preferences.default hardcoded
  reference). Tests can now control this path via Container registrations.

- UnlockAppWithPincodeViewModel: .prefix(1) on the biometric viewDidAppear
  trigger (so the prompt doesn't reappear on rotation/app-switcher return)
  and .first() on the merged unlock signal (so a biometric-success during
  mid-pincode entry can't fire userDid(.unlockApp) twice).

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
- <a href="https://github.com/Sajjon/Zhip/commit/5b7c685872b2505c86050f92a4a97d1c40da0002">5b7c685</a>: fix(security/safety): phase 1 — quick wins from code review

- MainCoordinator + SendCoordinator: [unowned self] → [weak self] in
  publisher chains whose source (deeplinkedTransaction) outlives the
  coordinator. Previously: hard crash if a deep link emitted after the
  coordinator was torn down (e.g. wallet removal during pending deep link).

- ReviewTransactionBeforeSigningViewModel.totalCostInZil: drop the
  `try!` and return Optional; the view shows "—" instead of crashing
  if Amount(qa:) ever rejects. The crashing path was on the funds-display
  step of Send — the worst-tier UX in a wallet app.

- SendCoordinator.openInBrowserDetailsForTransaction: route through the
  injected UrlOpener instead of UIApplication.shared.open. Restores DI
  consistency with the rest of the codebase.

- PrepareTransactionViewModel: fix the misleading comment claiming the
  recipient address is "not auto-checksummed". The payment construction
  silently checksums via toChecksummedLegacyAddress() — comment now
  describes actual behaviour and notes how to surface it as a real UX.

- KeyChainSwift+KeyValueStoring: document the load-time ordering invariant
  (each KeychainKey is read as the same type it was written; KeychainSwift
  stores Bool/String as raw bytes so getData would otherwise alias them).

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>


Created by <a href="https://github.com/my-badges/my-badges">My Badges</a>