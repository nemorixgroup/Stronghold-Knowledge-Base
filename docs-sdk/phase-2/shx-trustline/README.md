# SHx Trustline

**Phase:** 2 - SHx Asset Operations & Onboarding  
**Status:** ✅ Done (`0.0.4-dev`)  
**Implementation:** [`lib/src/wallet/shx_wallet.dart`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/master/lib/src/wallet/shx_wallet.dart), `ShxWallet.establishShxTrustline`; [`test/src/asset/shx_asset_test.dart`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/master/test/src/asset/shx_asset_test.dart)

## What This Is

`ShxWallet.establishShxTrustline()` completes the `funded -> shxReady` transition in the `ShxAccount` lifecycle. This document covers two decisions that came out of building it: who authorizes the transaction, and why it cannot be end-to-end tested on Testnet.

## Decision 1: Self-Authorized, No Funding Source Parameter

Unlike [`createAndFund`](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-2/mainnet-funding-source/README.md), which requires a caller-supplied `fundingSourceKeyPair` because a brand-new account cannot pay for its own creation, `establishShxTrustline()` takes no such parameter. The account opening the trustline signs for itself, using `account.keyPair`, and pays the added reserve requirement out of its own existing balance. There is no third party to model here: a trustline is a promise an account makes about itself, not a transfer between two accounts.

## Decision 2: Testnet Cannot Fully Verify This Method

`establishShxTrustline()` opens a trustline toward the real SHx issuer (`StrongholdConstants.shxIssuerAccountId`), intentionally: a trustline toward any other issuer would not be SHx, it would be a lookalike asset with the same code and nothing else in common (see [Module 02 of the Knowledge Base](https://github.com/nemorixgroup/Stronghold-Knowledge-Base) for why the issuer, not just the asset code, is what makes SHx real).

The first integration test written for this method targeted Testnet, matching every other integration test in this SDK so far ([Account Bootstrap Helpers](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-2/account-bootstrap-helpers/README.md), [Mainnet Funding Source](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-2/mainnet-funding-source/README.md)). It failed with:

```
transaction=tx_failed, operations=[op_no_issuer]
```

Stellar's Testnet and Mainnet are entirely separate ledgers with independent account state. The SHx issuer account exists on Mainnet only; it was never created on Testnet, so any operation referencing it as an issuer fails there, regardless of how correct the operation itself is.

This is the first method in the SDK where that matters: `fundOnTestnet()` and `createAndFund()` are asset-agnostic, they only move native XLM, which exists identically on both networks. `establishShxTrustline()` is the first method intentionally hardcoded to a specific real-world asset, and that asset simply does not exist outside Mainnet.

### Options Considered

1. **Accept an injectable asset for testing.** Rejected. Letting the method trust an arbitrary asset for test purposes would undermine the one guarantee its name makes: that calling it results in a trustline toward the real SHx, not a lookalike.
2. **Unit-test only the operation construction (asset code, issuer, limit), skip true end-to-end verification for now.** Chosen for the current release. A full Mainnet integration test is planned once a dedicated, funded Mainnet test account is available; opening a trustline only costs the Stellar minimum reserve (a small, recoverable amount, not spent, since [`ShxTrustline.buildRemoveOperation()`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/master/lib/src/asset/shx_asset.dart) already exists to close it again), so this is a cost/timing decision, not a technical blocker.

## Decision

`test/src/asset/shx_asset_test.dart` verifies the operation is built correctly (correct asset code, correct issuer, correct default and custom limits) without submitting it anywhere. The Testnet integration test for `establishShxTrustline` is marked `skip` with the real reason stated inline, rather than deleted or left silently failing, so the gap stays visible in every test run until a Mainnet test account closes it.

## Related

- [Account Bootstrap Helpers](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-2/account-bootstrap-helpers/README.md)
- [Mainnet Funding Source](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-2/mainnet-funding-source/README.md)
