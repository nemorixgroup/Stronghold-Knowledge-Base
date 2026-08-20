# Account Bootstrap Helpers

**Phase:** 2 - SHx Asset Operations & Onboarding  
**Status:** ✅ Done (`0.0.3-dev`)  
**Implementation:** [`lib/src/wallet/shx_wallet.dart`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/main/lib/src/wallet/shx_wallet.dart), [`lib/src/wallet/shx_account.dart`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/main/lib/src/wallet/shx_account.dart)

## What This Is

Before a developer can do anything SHx-specific (trustline, payment, governance), they need a Stellar account that actually exists on the network. This document records why that step was designed as a set of small, composable helpers instead of one all-in-one "create account" function.

## The Question

On Stellar, a freshly generated key pair is not an account until it receives its first XLM payment. That already breaks "create an account" into at least two steps (generate, fund), and a SHx-ready account needs a third (trustline). The open question was how much of that sequence the SDK should collapse into a single call.

## Options Considered

1. **One monolithic `onboardForShx()` call.** Rejected as the only option. It is the friendliest for the most common case, but it forces every developer into the same sequence, even ones who need to show a user their address before funding it, or who fund accounts through a flow the SDK cannot anticipate (an in-app purchase, a treasury batch job, a manual approval step).
2. **Small, composable steps, chosen at whichever granularity fits the app.** Chosen. Each step returns an [`ShxAccount`](../../phase-1/composition-not-fork/README.md) carrying an explicit [`ShxAccountStatus`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/main/lib/src/wallet/shx_account.dart) (`pending`, `funded`, `shxReady`), so a UI can always ask "where is this account right now" instead of re-deriving that state from raw balance queries.

## What Shipped in `0.0.3-dev`

- `ShxWallet.generate()`, `createPending()`, identity only, no network call  
- `ShxWallet.fundOnTestnet()`, Testnet funding via Friendbot  
- `ShxWallet.createAndFund()`, Mainnet funding via a caller-supplied source (see [Mainnet Funding Source](../mainnet-funding-source/README.md))

A higher-level `bootstrapForShx()` convenience wrapper (chaining create + fund + trustline in one call, for the common case) is still planned, once the trustline step is integrated into the `ShxAccount` lifecycle.

## Decision

Composable steps over a single opinionated flow. A monolithic helper can still be added later as a thin wrapper around these primitives without breaking anything, the reverse (breaking a monolithic call apart after developers already depend on it) would not be as clean.

## Related

- [Mainnet Funding Source](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-2/mainnet-funding-source/README.md)
